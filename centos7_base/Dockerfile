FROM registry.cn-hangzhou.aliyuncs.com/pixseed/centos7_base
MAINTAINER WisdomFusion <WisdomFusion@gmail.com>

RUN yum -y install wget

RUN mv /etc/yum.repos.d/CentOS-Base.repo{,.backup}

RUN wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo 

RUN yum clean all && yum makecache && yum -y update

