burnell-web <br>
===========
burnell-web [www.burnelltek.com](www.burnelltek.com) 是一个个人博客网站系统，后端使用Python编写 <br>

# 开发环境 <br>
Python3.5.2 <br>
Python依赖库：<br>
1. aiohttp，异步web框架库 <br>
2. jinja2， 前端模板引擎库 <br>
3. aiomysql，mysql异步驱动库 <br>
4. piplow，图片库用于生成验证码 <br>

# 项目结构 <br>
>backup                备份目录 <br>
>conf                  配置文件目录 <br>
>dist                  打包目录 <br>
>www                   web目录, 存放.py文件 <br>
>>static               存放静态文件 <br>
>>templates            存放模板文件 <br>

# 部署示例(阿里云ECS Ubuntu14.04) <br>
1. 安装SSH服务 <br>
sudo apt-get install openssh-server <br>

2. 安装Python3.5 <br>
sudo add-apt-repository ppa:fkrull/deadsnakes <br>
sudo apt-get update <br>
sudo apt-get install python3.5 <br>
如果无法识别add-apt-repository命令, 则需要进行如下操作: <br>
sudo apt-get install python-software-properties <br>
sudo apt-get install software-properties-common <br>

3. 取消原本的 Python 3.4 ，并将 Python3 链接到最新的 3.5 上：<br>
sudo mv /usr/bin/python3 /usr/bin/python3-old <br>
sudo ln -s /usr/bin/python3.5 /usr/bin/python3 <br>

4. 安装Python3包管理工具: python3-pip <br>
sudo apt-get install python3-pip <br>

5. 安装需要的Python库 <br>
sudo pip3 install jinja2 aiomysql aiohttp <br>
如果没有切换Python3链接的动作, 则pip3会把Python库安装在Python3.4上 <br>
安装pillow: <br>
sudo apt-get install python3.5-dev <br>
sudo apt-get install libjpeg8-dev zlib1g-dev libfreetype6-dev <br>
sudo pip3 install pillow <br>

6. 切换回来链接文件：<br>
sudo rm /usr/bin/python3 <br>
sudo mv /usr/bin/python3-old /usr/bin/python3 <br>

7. 安装MySQL数据库服务 <br>
sudo apt-get install mysql-server <br>

8. 安装 Nginx：高性能Web服务器+负责反向代理: <br>
sudo apt-get install nginx <br>
  
9. 安装 Supervisor：监控服务进程的工具 <br>
sudo apt-get install supervisor <br>
启动 Supervisor: sudo supervisord <br>

10. 设置mysql可以远程链接 <br>
vim /etc/MySQL/my.cnf找到bind-address = 127.0.0.1 <br>
注释掉这行，如：#bind-address = 127.0.0.1 <br>
重启 MySQL：sudo /etc/init.d/mysql restart <br>
root登录mysql: mysql -u root -p <br>
授权远程链接: grant all privileges on *.* to root@"%" identified by "password" with grant option; <br>
刷新权限信息: flush privileges; <br>

11. 配置Supervisor <br>
编写一个Supervisor的配置文件burnellweb.conf，存放到/etc/supervisor/conf.d/目录下<br>
然后重启Supervisor后，就可以随时启动和停止Supervisor管理的服务了<br>
重启：sudo supervisorctl reload <br>
启动我们的服务：sudo supervisorctl start burnellweb <br>
查看状态：sudo supervisorctl status<br>
    ```
    [program:burnellweb]
    command     = python3.5 /srv/burnell_web/www/app.py
    directory   = /srv/burnell_web/www
    user        = www-data
    startsecs   = 3
    redirect_stderr         = true
    stdout_logfile_maxbytes = 50MB
    stdout_logfile_backups  = 10
    stdout_logfile          = /srv/burnell_web/log/app.log
    ```


12. 配置Nginx <br>
Supervisor只负责运行app.py，我们还需要配置Nginx，把配置文件burnellweb放到/etc/nginx/sites-available/目录下<br>
让Nginx重新加载配置文件：sudo /etc/init.d/nginx reload <br>
    ```
    server {
        listen      80;
        root       /srv/burnell_web/www;
        access_log /srv/burnell_web/log/access_log;
        error_log  /srv/burnell_web/log/error_log;
        # server_name burnelltek.com;
        client_max_body_size 1m;
        gzip            on;
        gzip_min_length 1024;
        gzip_buffers    4 8k;
        gzip_types      text/css application/x-javascript application/json;
        sendfile on;
        location /favicon.ico {
            root /srv/burnell_web/www/static/img;
        }
        location ~ ^\/static\/.*$ {
            root /srv/burnell_web/www;
        }
        location / {
            proxy_pass       http://127.0.0.1:9000;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
    ```


# Release Note: <br>
## 2016/11/27 V1.1.1 Release <br>
1. 限制每10秒只能发送一条评论 <br>
2. 注册账号页面需要输入验证码 <br>







