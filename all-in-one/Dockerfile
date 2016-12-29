FROM centos7_base:base
#FROM registry.cn-hangzhou.aliyuncs.com/pixseed/centos7_base
MAINTAINER WisdomFusion <WisdomFusion@gmail.com>

# ENVs
ENV INSTALL_DIR /usr/local/webserver
ENV DBROOTPWD=123456

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
COPY packages/* /usr/src/
WORKDIR /usr/src/
RUN cat *.tar.gz *.tgz | tar -zxf - -i

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
  && chkconfig mysqld on

COPY conf/my.cnf /etc/

RUN mkdir -p /u01/mysql \
  && chown mysql:mysql /u01/mysql/ \
  && /usr/local/webserver/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/webserver/mysql --datadir=/u01/mysql \
  && echo "/usr/local/webserver/mysql/lib/" > /etc/ld.so.conf.d/mysql.conf \
  && /sbin/ldconfig

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

COPY conf/nginx /etc/init.d/
RUN chmod +x /etc/init.d/nginx \
  && chkconfig --add nginx \
  && chkconfig nginx on \
  && mkdir -p /u01/logfiles/nginx \
  && chmod +w /u01/logfiles/nginx \
  && chown -R www:www /u01/logfiles/nginx \
  && cp /usr/local/webserver/nginx/conf/nginx.conf{,.original}

COPY conf/nginx.conf /usr/local/webserver/nginx/conf/

EXPOSE 80 443 3306 8379 9000 9200 9300

ENTRYPOINT ["/etc/init.d/nginx"]
ENTRYPOINT ["/etc/init.d/php-fpm", "start"]
ENTRYPOINT ["/etc/init.d/mysqld", "start"]

