# somethingIneedtoKnow

在我的腾讯云上，第一步 添加用户,更新仓库
~~~
# 给已有的用户添加组
# gpasswd -a user group
groupadd dev
useradd -g dev xw
passwd xw:

cat /etc/group 查看组及组员
yum -y upgrade & update
# 添加用户sudo 权限

chmod -v u+w /etc/sudoers #先开放 当前用户写的权限

%groupname  ALL=(ALL)   NOPASSWD: ALL  给组内成员开放sudo权限
user    ALL+(ALL)   ALL 给用户开放sudo权限 并不需密码


#开放端口
firewall-cmd --add-port=8001/tcp
~~~
第二步，安装git、docker
~~~
yum -y install gcc g++ git docker-client  readline # readline 能解决python命令行退后乱码问题，需要重新源码安装python
git config --global user.name = 'xuwei'
git config --global user.email = 'xwzh987@163.com'
ssh-keygen -t rsa
~~~

第三步 安装python环境及相关组件 # python3.7.4不知道啥原因 总是出ssl的错误
~~~
openssl 要单独下载源码安装，反正就是要安装最新版本 不能从yum上安装
yum install libffi-devel -y # 解决python源码安装中的 no module named _ctypes问题 , 解决pip install ssl报错问题
./configure --prefix=/  --enable-optimazations  --with-openssl# 解决 --enable-shared CFLAGS=-fPIC # 把python安装成动态库
# 转变成动态库之后，需要指定动态库地址 cp libpython3.7m.so.1.0  /usr/lib64 否则pip用不了
~~~

MySQL 安装
~~~
# 已准备了安装包
~~~

第四步  Django+apache +mod-wsgi 高并发解决方案
~~~
yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel # 写错了 这是nginx需要的
yum -y group install "Development tools"
yum -y install httpd # 简单粗暴

vim /etc/httpd/conf/httpd.conf  # 修改配置

systemctl start httpd.service # 启动http

教程地址: https://blog.csdn.net/qq_33591055/article/details/80646482

1、上传项目至目标服务器并安装项目所需的模块
包整项目能正常通过runserver启动并且检查没有问题


----------


2、安装uwsgi 并使用uWSGI启动这个服务

执行这条命令的时候：一定要在这个项目目录中~ 
uwsgi --http 192.168.31.123:80 --file teacher/wsgi.py --static-map=/static=static


----------


3、使用配置文件启动uWSGI[ini]
    [uwsgi]
    # 项目目录
    chdir=/opt/project_teacher/teacher/
    # 启动uwsgi的用户名和用户组
    uid=root
    gid=root
    # 指定项目的application
    module=teacher.wsgi:application
    # 指定sock的文件路径
    socket=/opt/project_teacher/script/uwsgi.sock
    # 启用主进程
    master=true
    # 进程个数
    workers=5
    pidfile=/opt/project_teacher/script/uwsgi.pid
    # 自动移除unix Socket和pid文件当服务停止的时候
    vacuum=true
    # 序列化接受的内容，如果可能的话
    thunder-lock=true
    # 启用线程
    enable-threads=true
    # 设置自中断时间
    harakiri=30
    # 设置缓冲
    post-buffering=4096
    # 设置日志目录
    daemonize=/opt/project_teacher/script/uwsgi.log    


----------


4、安装Nginx
vim /etc/yum.repos.d/nginx.repo
    [nginx]
    name=nginx repo
    # 下面这行centos根据你自己的操作系统修改比如：OS/rehel
    # 6是你Linux系统的版本，可以通过URL查看路径是否正确
    baseurl=http://nginx.org/packages/centos/6/$basearch/
    gpgcheck=0
    enabled=1
# 安装nginx
yum -y install nginx
# 添加配置文件
vim teacher.conf # 这个名字随便起，最好是和项目一个名字
    server {
                    listen 80;
                    server_name 10.129.205.183 ;
                    access_log  /var/log/nginx/access.log  main;
                    charset  utf-8;
                    gzip on;
                    gzip_types text/plain application/x-javascript text/css text/javascript application/x-httpd-php application/json text/json image/jpeg image/gif image/png application/octet-stream;

                    error_page  404           /404.html;
                    error_page   500 502 503 504  /50x.html;
                    # 指定项目路径uwsgi
                    location / {
                        include uwsgi_params;
                        uwsgi_connect_timeout 30;
                        uwsgi_pass unix:/opt/project_teacher/script/uwsgi.sock;
                    }
                    # 指定静态文件路径
                    location /static/ {
                        alias  /opt/project_teacher/teacher/static/;
                        index  index.html index.htm;
                    }

    }


# django坑点1
xwdjango/xwdjango/__init__.py  + import pymysql  pymysql.install_as_MySQLdb()
# 坑点2  如果上述出错mysqlclient
pip3 install mysqlcient  xwdjango/xwdjango/__init__.py  import MySQLdb
~~~