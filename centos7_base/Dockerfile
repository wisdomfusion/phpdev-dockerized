FROM registry.cn-hangzhou.aliyuncs.com/pixseed/centos7_base
MAINTAINER WisdomFusion <WisdomFusion@gmail.com>

RUN yum -y install wget

RUN mv /etc/yum.repos.d/CentOS-Base.repo{,.backup}

RUN wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo 

RUN yum clean all && yum makecache && yum -y update

RUN yum -y install gcc gcc-c++ autoconf automake cmake zlib zlib-devel compat-libstdc++-33 glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel unzip zip nmap ncurses ncurses-devel sysstat ntp curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel libxml2 libxml2-devel libXpm libXpm-devel libmcrypt libmcrypt-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers pam-devel libicu libicu-devel

RUN yum -y install man wget iptraf iotop htop vim-enhanced lrzsz bash-completion

RUN alias vi='vim'
RUN echo "alias vi='vim'" >> ~/.bashrc
RUN echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
RUN echo "set fileformats=unix,dos" >> ~/.vimrc



