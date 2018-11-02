
# 安装python工作环境以及flaskr的配置环境

`sudo apt install python3-dev python3-pip build-essential`

## 创建虚拟工作区
没有需求可以不用virtualenv，testvenv为虚拟工作环境目录

`sudo apt install pip virtualenv
virtualenv -p /usr/bin/python3 testvenv`

启动虚拟python方式如下：`source venv/bin/activate` 或者 `. venv/bin/activate`，以下操作无说明情况下无需进入虚拟工作环境

## flask
虚拟工作环境下，使用pip安装flask

`pip install flask`

## uwsgi
uwsgi使用apt源安装，否则出现python库的静态链接文件找不到的问题，使用python3必须使用uwsgi-plugin-python3库

`sudo apt install uwsgi uwsgi-plugin-python3`

官方推荐使用uwsgitop查看uwsgi工作情况，用pip安装即可`pip install uwsgitop`

## nginx
`sudo apt install nginx`

# 配置服务器
## flask
建立runserver.py文件，里面必须含有flask的app，如下，此时可以使用`python runserver.py`试试能不能跑
```
# runserver.py
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080) #0.0.0.0表示在网络暴露，127.0.0.1表示本机访问
```
## uwsgi
建立一个uwsgi的配置文件 app_uwsgi.ini文件，如下，此处;为注释符号
```
[uwsgi]

socket = :4999
stats = :5001
; 设置主目录
set-placeholder = basedir=/home/dora/Desktop/flaskr_test 
; basedir和后面不可以有空格

gid = 1000
uid = 1000
; 可以使用id命令在shell中查看gid和uid
chdir = %(basedir)
plugins = python3
pathon-path = %(basedir)/testvenv/bin
virtualenv = %(basedir)/testvenv
wsgi-file = runserver.py
callable = app
processes = 5
threads = 2

harakiri = 30 
; 时长过30s后就断开连接
```
手动启动如下

`uwsgi --ini --plugins=python app_uwsgi.ini `

4999端口为给nginx连接的socket，5001端口为汇报工作情况的，可以使用`uwsgitop :5001`查看具体信息

为了方便，必须需要多个应用共存，需要使用emperor来操作这些应用，而且希望emperor后台启动
建立文件/etc/systemd/system/emperor.uwsgi.service文件如下
```
[Unit]
Description=uWSGI Emperor
After=syslog.target

[Service]
ExecStart=/usr/bin/uwsgi --ini /etc/uwsgi/emperor.ini

RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```
这里指向了/etc/uwsgi/emperor.ini文件如下
```[uwsgi]

emperor = /etc/uwsgi/apps-available
```

这里是为了方便给每个应用设置相同的参数，比如线程数之类的。apps-available为系统自带目录，告诉emperor在哪里找到应用的配置文件

这些配置文件适合使用软连接从应用目录链接过去

`sudo ln -s /home/dora/Desktop/flaskr/flaskr_uwsgi.ini /etc/uwsgi/apps-available/`

其他的应用也是如是操作，建立uwsgi.ini文件，然后软连接到apps-available文件夹即可，用不一样的名称即可。使用如下命令启动、重启、关闭、查看状态
```
systemctl start emperor.uwsgi.service
systemctl restart emperor.uwsgi.service
systemctl stop emperor.uwsgi.service
systemctl status emperor.uwsgi.service
```

使用`systemctl enable emperor.uwsgi.service`使服务开机启动

修改配置文件即重启uwsgi

## nignx
备份以及删除默认的nginx文件

`sudo mv /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/default.bk`

在应用目录下建立一个nginx的配置文件flask_test_nginx.conf
```
server {
    listen  5000;
    server_name localhost;
    charset utf-8;
    client_max_body_size    75M;

    location / { try_files $uri @yourapp; }
    location @yourapp {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:4999;
    }
}
```
可以看到4999的socket，也是使用软链接到/etc/nginx/conf.d/文件夹

`sudo ln -s /home/dora/Desktop/flaskr/flaskr_nginx.conf /etc/nginx/conf.d/`

nginx启动，重启，关闭如下
```
sudo nginx
sudo nginx -s reload
sudo nginx -s stop
```
## uwsgi的log输出在syslog上，niginx的log在/var/log/nginx/目录中
