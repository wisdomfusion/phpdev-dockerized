FROM registry.cn-hangzhou.aliyuncs.com/pixseed/centos7_base
MAINTAINER WisdomFusion <WisdomFusion@gmail.com>

# ENVs
ENV INSTALL_DIR /usr/local/webserver
EVN DBROOTPWD=123456

# centos repo
RUN mv /etc/yum.repos.d/CentOS-Base.repo{,.backup} \
  && wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo \
  && yum -y install epel-release

# update system
RUN yum clean all \
  && yum makecache \
  && yum -y update

# install some libs and tools
RUN yum -y install gcc gcc-c++ autoconf automake cmake zlib zlib-devel compat-libstdc++-33 \
  glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel \
  libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel \
  libxml2 libxml2-devel libXpm libXpm-devel libmcrypt-devel krb5 krb5-devel libidn libidn-devel \
  libicu libicu-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients \
  openldap-servers pam-devel \
  man wget unzip zip nmap net-tools iptraf sysstat iotop ntp vim-enhanced bash-completion

# Vim settings
RUN alias vi='vim' \
  && echo "alias vi='vim'" >> ~/.bashrc \
  && echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc \
  && echo "set fileformats=unix,dos" >> ~/.vimrc

# copy packages to /usr/src/, and extract them
COPY packages/* /usr/src/ \
  && tar zxf *.tar.gz \
  && tar zxf *.tgz

# install libs before php and mysql

WORKDIR /usr/src/libmcrypt-2.5.8/
RUN ./configure \
    && make \
    && make install \
    && /sbin/ldconfig
WORKDIR libltdl/
RUN ./configure --enable-ltdl-install \
  && make \
  && make install

WORKDIR /usr/src/mhash-0.9.9.9/
RUN ./configure \
    && make \
    && make install

RUN ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la \
  && ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so \
  && ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4 \
  && ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8 \
  && ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a \
  && ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la \
  && ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so \
  && ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2 \
  && ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1

WORKDIR /usr/src/mcrypt-2.6.8/
RUN /sbin/ldconfig \
  && ./configure \
  && make \
  && make install

RUN echo "/usr/local/lib" >> /etc/ld.so.conf \
  && /sbin/ldconfig

# install mysql

RUN /usr/sbin/groupadd mysql \
  && /usr/sbin/useradd -g mysql -s /sbin/nologin mysql

WORKDIR /usr/src/mysql-5.7.17/
RUN cmake \
    -DCMAKE_INSTALL_PREFIX=/usr/local/webserver/mysql \
    -DMYSQL_DATADIR=/u01/mysql \
    -DEXTRA_CHARSETS=all \
    -DDEFAULT_CHARSET=utf8 \
    -DDEFAULT_COLLATION=utf8_general_ci \
    -DMYSQL_TCP_PORT=3306 \
    -DMYSQL_USER=mysql \
    -DSYSCONFDIR=/etc \
    -DWITH_MYISAM_STORAGE_ENGINE=1 \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_PARTITION_STORAGE_ENGINE=1 \
    -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
    -DWITH_MEMORY_STORAGE_ENGINE=1 \
    -DWITH_BOOST=boost/boost_1_59_0 \
    -DENABLE_DOWNLOADS=1 \
  && make -j `nproc` \
  && make install

RUN cp /usr/local/webserver/mysql/support-files/mysql.server /etc/init.d/mysqld \
  && chmod +x /etc/init.d/mysqld \
  && chkconfig --add mysqld \
  && chkconfig mysqld on \
  && cat > /etc/my.cnf <<\EOF
[client]
port = 3306
socket = /tmp/mysql.sock
#default-character-set = utf8mb4

[mysqld]
port = 3306
socket = /tmp/mysql.sock
basedir = /usr/local/webserver/mysql
datadir = /u01/mysql
pid-file = /u01/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1
#init-connect = 'SET NAMES utf8mb4'
#character-set-server = utf8mb4
#skip-name-resolve
#skip-networking

back_log = 300
max_connections = 1000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M
thread_cache_size = 8
query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M
ft_min_word_len = 4
log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 30
log_error = /u01/mysql/mysql-error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file = /u01/mysql/mysql-slow.log
performance_schema = 0
explicit_defaults_for_timestamp
#lower_case_table_names = 1
skip-external-locking

default_storage_engine = InnoDB
#default-storage-engine = MyISAM
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120

bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1
interactive_timeout = 28800
wait_timeout = 28800

[mysqldump]
quick
max_allowed_packet = 16M

[Myisamchk]
Key_Buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
EOF

RUN mkdir -p /u01/mysql \
  && chown mysql:mysql /u01/mysql/ \
  && /usr/local/webserver/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/webserver/mysql --datadir=/u01/mysql \
  && echo "/usr/local/webserver/mysql/lib/" > /etc/ld.so.conf.d/mysql.conf \
  && /sbin/ldconfig \

RUN /etc/init.d/mysqld start \
  && /usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$DBROOTPWD\" with grant option;" \
  && /usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$DBROOTPWD\" with grant option;" \
  && /etc/init.d/mysqld stop

# install php

RUN /usr/sbin/groupadd www \
  && /usr/sbin/useradd -g www -s /sbin/nologin www \
  && mkdir -p /u01/www \
  && chmod +w /u01/www \
  && chown -R www:www /u01/www

WORKDIR /usr/src/php-7.1.0/
RUN ./configure --prefix=/usr/local/webserver/php7 \
    --with-config-file-path=/usr/local/webserver/php7/etc \
    --with-libdir=lib64 \
    --enable-fpm \
    --with-fpm-user=www \
    --with-fpm-group=www \
    --enable-mysqlnd \
    --with-mysqli=mysqlnd \
    --enable-pdo \
    --with-pdo-mysql=mysqlnd \
    --enable-opcache \
    --enable-pcntl \
    --enable-mbstring \
    --enable-soap \
    --enable-zip \
    --enable-calendar \
    --enable-bcmath \
    --enable-exif \
    --enable-ftp \
    --enable-intl \
    --enable-xml \
    --enable-sockets \
    --with-xmlrpc \
    --with-openssl \
    --with-zlib \
    --enable-mbregex \
    --with-curl \
    --with-gd \
    --with-zlib-dir=/usr/lib \
    --with-png-dir=/usr/lib \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2/ \
    --with-gettext \
    --with-mhash \
    --with-mcrypt \
    --with-ldap \
    --disable-ipv6 \
  && make -j `nproc` \
  && make install

RUN cp php.ini-production /usr/local/webserver/php7/etc/php.ini \
  && cp /usr/local/webserver/php7/etc/php-fpm.conf.default /usr/local/webserver/php7/etc/php-fpm.conf \
  && cp /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default /usr/local/webserver/php7/etc/php-fpm.d/www.conf

RUN sed -i '/^;pid/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^;error_log/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^;emergency_restart_threshold/c\emergency_restart_threshold = 60' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^;emergency_restart_interval/c\emergency_restart_interval = 60' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^;process/c\process_control_timeout = 5s' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^;daemonize/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^;rlimit_files/c\rlimit_files = 65535' /usr/local/webserver/php7/etc/php-fpm.conf \
  && sed -i '/^pm =/c\pm = static' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^pm.max_children/s/[0-9]\+$/256/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^pm.start_servers/s/[0-9]\+$/20/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^pm.min_spare_servers/s/[0-9]\+$/5/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^pm.max_spare_servers/s/[0-9]\+$/35/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^;pm.process_idle_timeout/s/^;//' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^;pm.max_requests/c\pm.max_requests = 51200' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default \
  && sed -i '/^;request_terminate_timeout/s/^;//' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default

RUN cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm \
  && chmod +x /etc/init.d/php-fpm \
  && chkconfig --add php-fpm \
  && chkconfig php-fpm on

# install php extensions

WORKDIR /usr/src/ImageMagick-7.0.4-0/
RUN ./configure \
  && make \
  && make install \
  && /sbin/ldconfig

WORKDIR /usr/src/imagick-3.4.3RC1/
RUN /sbin/ldconfig \
  && /usr/local/webserver/php7/bin/phpize \
  && ./configure --with-php-config=/usr/local/webserver/php7/bin/php-config \
  && make \
  && make install

WORKDIR /usr/src/redis-3.1.0/
RUN /usr/local/webserver/php7/bin/phpize \
  && ./configure --with-php-config=/usr/local/webserver/php7/bin/php-config \
  && make \
  && make install

# php.ini

RUN sed -i '/; extension_dir = "ext"/a\extension_dir = "/usr/local/webserver/php7/lib/php/extensions/no-debug-non-zts-20160303/"\nextension=imagick.so\nextension=redis.so' $INSTALL_DIR/php7/etc/php.ini \
  && sed -i '/\[opcache\]/a\\nzend_extension=opcache.so\nopcache.enable=1\nopcache.memory_consumption=256\nopcache.interned_strings_buffer=8\nopcache.max_accelerated_files=4000\nopcache.revalidate_freq=60\nopcache.fast_shutdown=1\n' $INSTALL_DIR/php7/etc/php.ini

# install nginx

WORKDIR /usr/src/pcre-8.39/
RUN ./configure \
  && make \
  && make install

WORKDIR /usr/src/nginx-1.10.2/
RUN /sbin/ldconfig \
  && ./configure \
  --prefix=/usr/local/webserver/nginx \
  --user=www \
  --group=www \
  --with-http_v2_module \
  --with-http_sub_module \
  --with-http_ssl_module \
  --with-http_gunzip_module \
  --with-http_gzip_static_module \
  --with-http_realip_module \
  --with-http_flv_module \
  --with-http_stub_status_module \
  --with-http_addition_module \
  && make \
  && make install

RUN cat > /etc/init.d/nginx <<\EOF
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# nginx:       /usr/local/webserver/nginx/sbin/nginx
# config:      /usr/local/webserver/nginx/conf/nginx.conf
# pidfile:     /usr/local/webserver/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/webserver/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/webserver/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
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
EOF

RUN chmod +x /etc/init.d/nginx
RUN chkconfig --add nginx
RUN chkconfig nginx on

RUN mkdir -p /u01/logfiles/nginx
RUN chmod +w /u01/logfiles/nginx
RUN chown -R www:www /u01/logfiles/nginx

RUN cp /usr/local/webserver/nginx/conf/nginx.conf{,.original}
RUN cat > /usr/local/webserver/nginx/conf/nginx.conf <<\EOF
user  www www;
worker_rlimit_nofile 65535;

error_log  /u01/logfiles/nginx/nginx_error.log  warn;

events
{
  use epoll;
  worker_connections 65535;
}

http
{
  include       mime.types;
  default_type  application/octet-stream;

  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;

#  limit_conn_zone $binary_remote_addr zone=connlimit:9m;

  server_tokens off;
  sendfile on;
  tcp_nopush on;

  keepalive_timeout 10;

  tcp_nodelay on;

  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 8 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;

  gzip on;
  gzip_min_length  1k;
  gzip_buffers     4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types       text/plain application/x-javascript text/css application/xml;
  gzip_vary on;

  log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
              '$status $body_bytes_sent "$http_referer" '
              '"$http_user_agent" $http_x_forwarded_for';

  server
  {
    listen       80;
    index index.php index.html;
    root  /u01/www/;

    #limit_conn  connlimit 15;
    #limit_rate  256k;

    location ~ .*\.php$
    {
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
      expires      30d;
      access_log   off;
    }
    location ~ .*\.(js|css)$
    {
      expires      1h;
      access_log   off;
    }

    access_log  /u01/logfiles/nginx/access.log  access;
  }

}
EOF

# nginx log rotation

RUN cat > /usr/local/webserver/nginx/cut_nginx_log.sh <<\EOF
#!/bin/bash
# cut_nginx_log.sh
# This script run at 00:00

# The Nginx logs path
log_path="/u01/logfiles/nginx"
new_log_path=${log_path}/$(date -d 'yesterday' '+%Y%m%d')

mkdir -p ${new_log_path}
mv ${log_path}/*.log ${new_log_path}

kill -USR1 `cat /usr/local/webserver/nginx/logs/nginx.pid`
EOF

RUN echo '0 0 * * * /bin/bash /usr/local/webserver/nginx/cut_nginx_log.sh' >> /var/spool/cron/root

# ports

EXPOSE 80 443 3306 8379 9000 9200 9300

