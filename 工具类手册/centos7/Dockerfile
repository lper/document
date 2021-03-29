FROM xxxx:nginx:v7-centos

MAINTAINER lper <lper@foxmail.com>
RUN mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
COPY CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
RUN mv /etc/yum.repos.d/systemd.repo /etc/yum.repos.d/systemd.repo.bak
COPY systemd.repo /etc/yum.repos.d/systemd.repo
RUN rpm --rebuilddb && yum -y update systemd  && yum -y clean all


RUN set -xe \
    && buildDeps="gcc gcc-c++ make cmake wget git" \
    && rpm --rebuilddb && yum install -y $buildDeps \
    && mkdir -p /data/server && mkdir -p /data/deps \
    && cd /data/deps \
    && wget https://luarocks.org/releases/luarocks-3.5.0.tar.gz \
    && tar zxpf luarocks-3.5.0.tar.gz \
    && cd luarocks-3.5.0 \
    &&./configure \
    && make \ 
    && make install \
    && make clean \
    && luarocks install openssl
ENV TERM xterm
CMD ["/bin/sh", "/var/temp/start.sh"]