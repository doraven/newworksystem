# newworksystem
It's just a personal instruction after Ubuntu install.

## 常见软件包安装
- 安装vim,vim-gnome支持系统剪切板"+p, 解压软件

`sudo apt install vim vim-gnome unrar rar tree`

- 安装texlive

`sudo apt install texlive texlive-lang-chinese texmaker`

安装宋体、楷体、黑体，文件头需添加如下参数
# 	\documentclass[fontset=windows][ctexbook]
`mkdir ~/.fonts && mkdir ~/.fonts/WinFonts
cp [fonts] ~/.fonts/WinFonts/ && cd ~/.fonts/WinFonts
sudo chmod 744 *
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -f -v`

# 安装chrome
[Download www.google.cn/intl/zh-CN/chrome/]
sudo touch /etc/default/google-chrome
dpkg -i chrome.deb

# 安装sougou输入法
[Download sougou linux]
sudo dpkg -i sogou.deb
sudo apt install -f
sudo dpkg -i sogou.deb

# 设置目录名称为英文
[先更改各目录名为Desktop Downloads 
	Templates Documents Pictures PublicShares Music Videos]
~/.config/user-dir.dirs

# 安装inkscape以及插件textext
sudo apt install inkscape
[download github.com/textext/textext]
tar xf TexText-Linux----.tgz && cp TexText-linux----/extention/* ~/.config/inkscape/extentions/
sudo apt install python-gtk2 python-gtksourceviews pdf2svg imagemagick libcanberra-gtk-module

# 安装python工作环境###########以及flaskr的配置环境
sudo apt install pip virtualenv
virtualenv -p /usr/bin/python3 venv # 创建虚拟工作区
source venv/bin/activate ///or/// . venv/bin/activate # 启动虚拟python
pip install flask niginx uwsgi uwsgi-plugin-python uwsgi-plugin-python3 # 安装所需要的库
sudo mv /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/default.bk # 备份nginx的默认文件
# 建立一个nginx的配置文件nginx.conf
sudo ln -s /***/nginx.conf /etc/nginx/conf.d/
sudo nginx -s reload
# 建立一个uwsgi的配置文件app_uwsgi.ini
# 手动启动 uwsgi --ini --plugins=python ***.ini
# 自动启动如下
ln -s /home/**/app_uwsgi.ini /etc/uwsgi/apps-available/
# 建立配置文件 /etc/uwsgi/uwsgi.ini 和 /etc/systemd/system/emperor.uwsgi.service
systemctl start/stop/restart/status emperor.uwsgi.service

######### 常用配置 ##############
# 这里只记录非 \home 目录下的设置，\home 下的设置记录设置文件
~/.bash_aliases
