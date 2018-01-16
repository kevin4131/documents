#  lnmp搭建&&PHP安装扩展

## 目录
[TOC]
## nginx 编译安装

### 下载安装包

- 地址

  <http://nginx.org/en/download.html>

- 下载

  `mkdir /download`

  `cd /download`

  `wget http://nginx.org/download/nginx-1.13.4.tar.gz`


###  解压安装包

`tar -vzxf nginx-1.13.4.tar.gz`

###  安装支持库 

````commonlisp
yum install -y gcc gdb strace gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs patch e2fsprogs-devel krb5-devel libidn libidn-devel openldap-devel nss_ldap openldap-clients openldap-servers libevent-devel libevent uuid-devel uuid mysql-devel
````
### 创建nginx组&&用户

```nginx
# 创建组
groupadd nginx   
addgroup nginx

# 创建用户
useradd -g nginx nginx 
adduser -g nginx nginx 
```



### 配置nginx

```nginx
# 切换解压目录
cd nginx-1.13.4 

# 配置
./configure --prefix=/usr/local/nginx  --sbin-path=/usr/local/nginx/sbin/nginx --user=nginx --group=nginx 
```

### 编译nginx

```nginx 
# 编译命令
make

# 检测编译
make test
```

### 安装nginx

- 安装命令

  `make install`


### 配置nginx.conf

- 切换至conf目录

  `cd /usr/local/nginx/conf`

- 备份nginx.conf

  `cp nginx.conf nginx.conf.bak`

- 创建子conf文件夹

  `mkdir conf.w`

- 编辑nginx.conf

  `vim nginx.conf`

  ```nginx
    #user nobody                => user nginx; 
    #error_log  logs/error.log; => error_log  logs/error.log;
    #pid        logs/nginx.pid; => pid        logs/nginx.pid;
    #gzip  on;                  => gzip  on;
  ```


- 复制代码

  ```nginx
        server {
            listen       80;
            server_name  localhost;

            #charset koi8-r;

            #access_log  logs/host.access.log  main;

            location / {
                root   html;
                index  index.html index.htm;
            }

            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }

            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
                    #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            #location ~ \.php$ {
            #    root           html;
            #    fastcgi_pass   127.0.0.1:9000;
            #    fastcgi_index  index.php;
            #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            #    include        fastcgi_params;
            #}

            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny  all;
            #}
        }
  ```



- 删除上面复制的代码并修改为如下

  ```nginx
  include /usr/local/nginx/conf.w/*;
  ```

- 保存退出nginx.conf

  `exit`  `:wq`

- 创建子配置文件

  `vim ./conf.w/test.conf`

  `i`   `ctrl ins` 

   `exit` `:wq`

- 检测配置文件是否正确

  `nginx -t`

  ```nginx
  # 正确结果
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful 
  ```



- 注意

  ```tex
    注意行结尾分号;
  ```




### 设置配置路径 

- 命令

  `nginx -c /usr/local/nginx/conf/nginx.conf`

  ​

### 启动nginx

- 命令

  `nginx -s start`

  ​

### 关闭nginx

- 命令

  `nginx -s stop`

  ​

### 重载nginx

- 命令

  `nginx -s reload`


### 端口

- 开启端口

  `iptables -A INPUT -p tcp --dport xxxport -j ACCEPT`

- 关闭端口

  `iptables -A OUTPUT -p tcp --dport xxxport -j DROP`

### 查看是否启动

- 命令

  `ps -ef|grep nginx`

  ```commonlisp
  root     17957     1  0 Aug17 ?        00:00:00 nginx: master process nginx -c /usr/local/nginx/conf/nginx.conf
  nginx    18010 17957  0 Aug17 ?        00:00:00 nginx: worker process
  nginx    21969 21968  0 Aug17 ?        00:00:00 php-fpm: pool www
  nginx    21970 21968  0 Aug17 ?        00:00:00 php-fpm: pool www
  root     26175 17094  0 11:27 pts/0    00:00:00 grep --color=auto nginx
  ```

------

### 开机自启动

- 命令

```nginx
# 创建自启动脚本
vim /etc/init.d/nginx
# 添加服务项
chkconfig --add nginx
# 开启自启动
chkconfig nginx on
# 检测
chkconfig --list | grep nginx
```

```tex
# 345 为on即可
nginx          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
# 如果3 4 5 off
chkconfig --level 345 nginx on
```

- /etc/init.d/nginx `脚本

```nginx
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

# 修改成自己编译安装的nginx路径
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

# 修改成自己nginx配置文件路径
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```



  ## PHP7 编译安装

  ### 下载安装包

- 官网地址

  <http://php.net/downloads.php>

- 下载命令

  `cd /download`

  `wget http://hk1.php.net/get/php-7.1.8.tar.gz/from/this/mirror`

  ​

###  解压安装包

- 解压命令
  `tar -vzxf mirror`

  ​

###  安装依赖包 

- 安装命令

  `````commonlisp
  yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
  `````
### 配置PHP7

- 切换解压目录

  `cd mirror` 

- 配置

  ```commonlisp
  ./configure \
  --prefix=/usr/local/php \
  --with-config-file-path=/etc \
  --enable-fpm \
  --with-fpm-user=nginx  \
  --with-fpm-group=nginx \
  --enable-inline-optimization \
  --disable-debug \
  --disable-rpath \
  --enable-shared  \
  --enable-soap \
  --with-libxml-dir \
  --with-xmlrpc \
  --with-openssl \
  --with-mcrypt \
  --with-mhash \
  --with-pcre-regex \
  --with-sqlite3 \
  --with-zlib \
  --enable-bcmath \
  --with-iconv \
  --with-bz2 \
  --enable-calendar \
  --with-curl \
  --with-cdb \
  --enable-dom \
  --enable-exif \
  --enable-fileinfo \
  --enable-filter \
  --with-pcre-dir \
  --enable-ftp \
  --with-gd \
  --with-openssl-dir \
  --with-jpeg-dir \
  --with-png-dir \
  --with-zlib-dir  \
  --with-freetype-dir \
  --enable-gd-native-ttf \
  --enable-gd-jis-conv \
  --with-gettext \
  --with-gmp \
  --with-mhash \
  --enable-json \
  --enable-mbstring \
  --enable-mbregex \
  --enable-mbregex-backtrack \
  --with-libmbfl \
  --with-onig \
  --enable-pdo \
  --with-mysqli=mysqlnd \
  --with-pdo-mysql=mysqlnd \
  --with-zlib-dir \
  --with-pdo-sqlite \
  --with-readline \
  --enable-session \
  --enable-shmop \
  --enable-simplexml \
  --enable-sockets  \
  --enable-sysvmsg \
  --enable-sysvsem \
  --enable-sysvshm \
  --enable-wddx \
  --with-libxml-dir \
  --with-xsl \
  --enable-zip \
  --enable-mysqlnd-compression-support \
  --with-pear \
  --enable-opcache
  ```

  ​

### 编译PHP7

- 编译命令

  `make`

- 检测编译

  `make test`

  ​

### 安装PHP7

- 安装命令

  `make install`


### 配置环境变量

- vim 编辑profile

   `vim /etc/profile`

- 修改末尾

  ```tex
   #添加
   PATH=$PATH:/usr/local/php/bin
   export PATH
  ```

- 立即生效

  `source /etc/profile`


### 配置php-fpm

```commonlisp
# 创建php.ini
cp php.ini-production /etc/php.ini
# php-fpm 配置
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
# 创建应用配置
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
# 自启动项
cp ./sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
# 添加可执行权限
chmod +x /etc/init.d/php-fpm
```

### 开机自启动

```commonlisp
# 添加php-fpm
chkconfig --add php-fpm
# 开启自启动
chkconfig php-fpm on
# 查看是否开启
chkconfig --list | grep php-fpm
```

```tex
php-fpm        	0:off	1:off	2:on	3:on	4:on	5:on	6:off
# 如果3 4 5 off
chkconfig --level 345 php-fpm on
```



### 启动fpm

`/etc/init.d/php-fpm start`

`service php-fpm start`



###  关闭fpm

`/etc/init.d/php-fpm stop`

`service php-fpm stop`



### 重启fpm

`/etc/init.d/php-fpm restart`

`service php-fpm restart`



### 查看是否启动

- 命令

  `ps -ef|grep php-fpm`

  ​

### 修改nginx配置支持PHP

- 编辑子配置test.conf

  `vim /usr/local/nginx/conf/conf.w/test.conf`

  ```nginx
  #server_name下添加如下两行命令
    root /wwwroot/test;
    index index.php index.html index.htm;

    #修改代码
    location / {
    root   html;                     => /wwwroot/test;
    index  index.html index.htm;     => index.php index.html index.htm;
  }
  ```
  ```nginx
  #找到此行
  #location ~ \.php$ {                                               
  #    root           html;                                         
  #    fastcgi_pass   127.0.0.1:9000;
  #    fastcgi_index  index.php;
  #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
  #    include        fastcgi_params;
  #}

  #修改成如下
  location ~ \.php$ {
    root           /wwwroot/test;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
  }
  ```

### 设置nginx配置路径

- 命令

  `nginx -c /usr/local/nginx/conf/nginx.conf`

  ​

### 启动nginx

- 命令

  `nginx -s start`

  ​

### 关闭nginx

- 命令

  `nginx -s stop`

  ​

### 重载nginx

- 命令

  `nginx -s reload`\

  ​

### 创建index.php

`echo '<?php phpinfo();?>' >> /wwwroot/test/index.php `



### 访问localhost

`php -i`

### 参考地址

<http://www.jianshu.com/p/246ffcd5e77d>

------

## mysql二进制包安装

### 下载二进制包

- 官网地址

  <https://dev.mysql.com/downloads/mysql/>

- 下载命令

  `wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz`

###  解压二进制包

`tar -vzxf mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz`

### 创建mysql组和用户

`addgroup mysql`     `adduser -r -g mysql mysql`

### 移动解压包

`mv -i mysql-5.7.19-linux-glibc2.12-x86_64 /usr/local/mysql` 

### 修改mysql解压包用户和组

`chown -R mysql:mysql /usr/local/mysql`

### 修改my.cnf

- 是否有my_default.cnf

```mysql
#查看support-files文件夹的内容，发现并没有my_default.cnf默认的配置文件
#如果没有默认的配置文件，需要手动创建一个my_default.cnf配置文件
#my_default.cnf示例

# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
# *** DO NOT EDIT THIS FILE. It's a template which will be copied to the
# *** default location during install, and will be replaced if you
# *** upgrade to a newer version of MySQL.

[mysqld]
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 

# 一般配置选项
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
socket = /usr/local/mysql/mysqld/mysqld.sock
character-set-server=utf8


#下面是可选项，要不要都行，如果出现启动错误，则全部注释掉，保留最基本的配置选项，然后尝试添加某些配置项后启动，检测配置项是否有误
back_log = 300
max_connections = 3000
max_connect_errors = 50
table_open_cache = 4096
max_allowed_packet = 32M
#binlog_cache_size = 4M

max_heap_table_size = 128M
read_rnd_buffer_size = 16M
sort_buffer_size = 16M
join_buffer_size = 16M
thread_cache_size = 16
query_cache_size = 128M
query_cache_limit = 4M
ft_min_word_len = 8

thread_stack = 512K
transaction_isolation = REPEATABLE-READ
tmp_table_size = 128M
#log-bin=mysql-bin
long_query_time = 6


server_id=1

innodb_buffer_pool_size = 1G
innodb_thread_concurrency = 16
innodb_log_buffer_size = 16M


innodb_log_file_size = 512M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120
innodb_file_per_table = on

[mysqldump]
quick

max_allowed_packet = 32M

[mysql]
no-auto-rehash
default-character-set=utf8
safe-updates

[myisamchk]
key_buffer = 16M
sort_buffer_size = 16M
read_buffer = 8M
write_buffer = 8M

[mysqlhotcopy]
interactive-timeout

[mysqld_safe]
open-files-limit = 8192

[client]
/bin/bash: Q: command not found
```

- 复制my_default.cnf

  `cp /usr/local/mysql/support-files/my_default.cnf  /usr/local/mysql/my.cnf`

### 创建socket存储文件夹

`mkdir -p /usr/local/mysql/mysqld`

`chown -R mysql:mysql /usr/local/mysql/mysqld`

###  注册和初始化MySQL服务

- 切换目录

  `cd /usr/local/mysql`

- 注册&&初始化

  `./bin/mysqld --initialize-insecure --user=mysql  --group=mysql  --basedir=/usr/local/mysql/  --datadir=/usr/local/mysql/data/`

- 注意

  ```mysql
  #初始化后密码为空
   --initialize-insecure 

  #初始化后有密码
   --initialize
  #添加此行代码
  #跳跃权限表的限制，不用验证密码，直接登录
  #修改密码后注释掉
   skip-grant-tables
  ```

### 复制文件

- my.cnf

  `cp ./my.cnf /etc/my.cnf`

- mysql.server

  `cp ./support-file/mysql.server /etc/init.d/mysqld`

### 修改data文件夹权限

`chmod 777 ./data`

### 配置环境变量

- vim 编辑profile

   `vim /etc/profile`

- 修改末尾

  ```tex
  PATH=$PATH:/usr/local/php/bin:/usr/local/mysql/bin
  ```

- 立即生效

  `source /etc/profile`

### 配置sock软连接

- 安全模式

`./bin/mysqld_safe --user=mysql`

- 创建软连接（<span style="color:red;">此步在sock目录在`/usr/local/nginx/mysqld`不需要</span>）

`ln -s /var/run/mysqld/mysqld.sock  /tmp/mysql.sock`

### 启动mysql

`service mysqld start`    `/etc/init.d/mysqld start`

### 关闭mysql

`service mysqld stop`  `/etc/init.d/mysqld stop`

### 修改mysql密码

- 登陆

  ` mysql -uroot -p`

- 设置密码

  `mysql> set password for root@localhost=password('your password');`

- 重新加载权限表，更新权限

  `mysql> flush privileges;`

### 修改mysql外网访问

- 登陆

  `mysql -uroot -p`

- 修改权限

  `mysql> use mysql;`

  `mysql> update user set Host = '%' where User = 'root';`

- 重新加载权限表，更新权限

  `mysql> flush privileges;`

### 重启mysql

`service mysqld restart`     `/etc/init.d/mysqld restart`

### command检测外网能否访问

```commonlisp
C:\Users\kevin>mysql -uyourUser  -hyourHost  -p
Enter password: yourPassword
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1566
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### 开机自启动

```mysql
# 添加可执行条件
chmod +x /etc/init.d/mysql
# 添加服务项
chkconfig --add mysqld
# 开启自启动
chkconfig mysqld on
# 检测
chkconfig --list | grep mysqld
```

```mysql
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
# 如果3 4 5 off
chkconfig --level 345 mysqld on
```

------



## 安装PHP扩展

### 下载扩展包

- 官网地址

  <http://pecl.php.net/>

- 下载命令(xdebug为例)

  ` cd /download`

  ` wget http://pecl.php.net/get/xdebug-2.5.5.tgz`

  ​

### 解压安装包

` tar -vzxf xdebug-2.5.5.tgz`

###  建立php外挂模块

`cd xdebug-2.5.5`

`/usr/local/php/bin/phpize`  或者

`phpize`

```commonlisp
[root@VM_1_239_centos xdebug-2.5.5]# phpize 
Configuring for:
PHP Api Version:         20160303
Zend Module Api No:      20160303
Zend Extension Api No:   320160303
```

### 配置

`./configure --enable-xdebug --with-php-config=/usr/local/php/bin/php-config`

### 编译

`make`

### 测试编译

`make test`

### 安装

`make install`

### 修改php.ini

- 检测ini加载路径

  `php -i | grep php.ini`

  ```commonlisp
  [root@VM_1_239_centos php]# php -i|grep php.ini
  Configuration File (php.ini) Path => /etc
  Loaded Configuration File => /etc/php.ini
  ```

- 修改php.ini

  `vim /etc/php.ini`           

   `i`

  ```ini
  #添加如下代码
  [Xdebug]
  zend_extension             = xdebug.so  
  xdebug.profiler_enable     = On
  xdebug.auto_trace          = On
  xdebug.collect_params      = On
  xdebug.collect_return      = On
  xdebug.trace_output_dir    = "/usr/local/php/xdebug_trace"  
  xdebug.profiler_output_dir = "/usr/local/php/xdebug_profiler"
  ```

  `exit`           

  `:wq` 

### 重启php-fpm

`/etc/init.d/php-fpm restart`

`service php-fpm restart`

###  检测是否安装成功

`php -i |grep xdebug`

```commonlisp
[root@VM_1_239_centos php]# php -i |grep xdebug
xdebug
xdebug support => enabled
xdebug.auto_trace => On => On
xdebug.cli_color => 0 => 0
xdebug.collect_assignments => Off => Off
xdebug.collect_includes => On => On
xdebug.collect_params => 1 => 1
xdebug.collect_return => On => On
xdebug.collect_vars => Off => Off
xdebug.coverage_enable => On => On
xdebug.default_enable => On => On
xdebug.dump.COOKIE => no value => no value
xdebug.dump.ENV => no value => no value
xdebug.dump.FILES => no value => no value
xdebug.dump.GET => no value => no value
xdebug.dump.POST => no value => no value
xdebug.dump.REQUEST => no value => no value
xdebug.dump.SERVER => no value => no value
xdebug.dump.SESSION => no value => no value
xdebug.dump_globals => On => On
xdebug.dump_once => On => On
xdebug.dump_undefined => Off => Off
xdebug.extended_info => On => On
xdebug.file_link_format => no value => no value
xdebug.force_display_errors => Off => Off
xdebug.force_error_reporting => 0 => 0
xdebug.halt_level => 0 => 0
xdebug.idekey => no value => no value
xdebug.max_nesting_level => 256 => 256
xdebug.max_stack_frames => -1 => -1
xdebug.overload_var_dump => 2 => 2
xdebug.profiler_aggregate => Off => Off
xdebug.profiler_append => Off => Off
xdebug.profiler_enable => On => On
xdebug.profiler_enable_trigger => Off => Off
xdebug.profiler_enable_trigger_value => no value => no value
xdebug.profiler_output_dir => /usr/local/php/xdebug_profiler => /usr/local/php/xdebug_profiler
xdebug.profiler_output_name => cachegrind.out.%p => cachegrind.out.%p
xdebug.remote_addr_header => no value => no value
xdebug.remote_autostart => Off => Off
xdebug.remote_connect_back => Off => Off
xdebug.remote_cookie_expire_time => 3600 => 3600
xdebug.remote_enable => Off => Off
xdebug.remote_handler => dbgp => dbgp
xdebug.remote_host => localhost => localhost
xdebug.remote_log => no value => no value
xdebug.remote_mode => req => req
xdebug.remote_port => 9000 => 9000
xdebug.scream => Off => Off
xdebug.show_error_trace => Off => Off
xdebug.show_exception_trace => Off => Off
xdebug.show_local_vars => Off => Off
xdebug.show_mem_delta => Off => Off
xdebug.trace_enable_trigger => Off => Off
xdebug.trace_enable_trigger_value => no value => no value
xdebug.trace_format => 0 => 0
xdebug.trace_options => 0 => 0
xdebug.trace_output_dir => /usr/local/php/xdebug_trace => /usr/local/php/xdebug_trace
xdebug.trace_output_name => trace.%c => trace.%c
xdebug.var_display_max_children => 128 => 128
xdebug.var_display_max_data => 512 => 512
xdebug.var_display_max_depth => 3 => 3
OLDPWD => /data/xdebug-2.5.5
$_SERVER['OLDPWD'] => /data/xdebug-2.5.5
```

------

