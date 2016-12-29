FROM registry.cn-hangzhou.aliyuncs.com/pixseed/centos7_base
MAINTAINER WisdomFusion <WisdomFusion@gmail.com>

# ENVs
ENV INSTALL_DIR /usr/local/webserver

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

