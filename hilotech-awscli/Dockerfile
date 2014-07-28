FROM centos:centos6
MAINTAINER hilotech.jp
##
#   @container aws-cli on centos:centos6
#   @author SHIOBARA -Bar- Hiroaki
#   @update 2014/07/28
##

ENV _USERNAME iam
ENV _PASSWORD password
ENV _SMTP_RELAY smtp.relay.host

# networking
RUN echo "NETWORKING=yes" > /etc/sysconfig/network


# timezone
RUN \
    echo 'ZONE="Asia/Tokyo"' > /etc/sysconfig/clock && \
    rm -f /etc/localtime && \
    ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime


# packages
RUN \
    yum -y install perl yum-utils telnet wget which sudo passwd && \
    perl -07 -pi \
        -e 's/(centosplus.*?enabled)=0/$1=1/igs;' \
        /etc/yum.repos.d/CentOS-Base.repo && \
    echo 'multilib_policy=best' >> /etc/yum.conf && \
    yum -y remove sendmail && \
    yum -y install \
        postfix mailx cronie sysstat \
        openssl openssh-client openssh-server \
        vim
#RUN yum -y groupinstall "Development tools"
#RUN \
#    yum -y install \
#        cmake
RUN \
    rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.i686.rpm && \
    perl -07 -pi \
        -e 's/(\[rpmforge\].*?enabled *)= *1/$1=0/is;' \
        /etc/yum.repos.d/rpmforge.repo && \
    yum -y install --enablerepo=rpmforge-extras \
        subversion git mercurial
RUN \
    yum -y update


# some magic spell
RUN echo "relayhost = [$_SMTP_RELAY]" >> /etc/postfix/main.cf


# AWS CLI
RUN wget  -O - https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py \
        | python && \
    easy_install pip
RUN \
    pip install awscli
RUN if [ ! -d /usr/src/netsrc ]; then mkdir -p /usr/src/netsrc; fi
RUN \
    yum install -y \
        gcc gcc-c++ make ruby \
        libgcrypt-devel libcurl-devel expat-devel binutils-devel \
        qt-devel bzip2-devel libarchive-devel \
        json-c-devel \
        boost \
        cmake
RUN \
    cd /usr/src/netsrc && \
    git clone git://github.com/lloyd/yajl && \
    cd yajl && \
    ./configure && \
    make install && \
    cd /usr/src/netsrc && \
    rm -rf yajl
RUN \
    wget http://stedolan.github.io/jq/download/linux64/jq \
        -O /usr/local/bin/jq && \
    chmod 0755 /usr/local/bin/jq


# make user
RUN \
    useradd \
        --create-home \
        --user-group \
        --groups wheel \
        $_USERNAME
RUN echo 'password' | passwd --stdin $_USERNAME
RUN \
    mkdir /home/$_USERNAME/.aws && \
    chmod 0700 /home/$_USERNAME/.aws && \
    chown ${_USERNAME}.${_USERNAME} /home/$_USERNAME/.aws

RUN echo -e '\
LANG="C" \
' > /etc/sysconfig/i18n

RUN \
    /bin/echo -e \
        "set showcmd\nset visualbell\nset nobackup" >> /etc/virc && \
    /bin/echo -e \
        "set showcmd\nset visualbell\nset nobackup" >> /etc/vimrc && \
    /bin/echo -e \
        "%wheel ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/wheel && \
        chmod 0600 /etc/sudoers.d/wheel && \
    /bin/echo -e '\
alias ls="/bin/ls -aFC"\n\
alias dir="/bin/ls -aFCl"\n\
alias md="mkdir"\n\
alias rd="rmdir"\n\
' > /etc/bash.aliases && \
    /bin/sed -i -e \
        "s/\(if.*PS1.*then\)/\1\n  stty stop undef\n  . \/etc\/bash.aliases\n  complete -C aws_completer aws\n/" \
        /etc/bashrc && \
    /bin/echo -e \
        "\n\$if Bash\n\x22\\\en\x22: history-search-forward\n\x22\\\ep\x22: history-search-backward\n\$endif\nset bell-style visible" \
        >> /etc/inputrc && \
    /bin/echo -e \
        "\nexport HISTCONTROL=erasedups\nexport IGNOREEOF=10" \
        >> /etc/profile

RUN sed -ri 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config


# invoke services
RUN \
    chkconfig sshd on \
    chkconfig crond on \
    chkconfig postfix on

RUN echo -e '\
#!/bin/sh\n\
service sshd start\n\
service crond start\n\
service postfix start\n\
\n\
while true\n\
do\n\
    sleep 10\n\
done\n\
\n\
' > /etc/services.sh
RUN chmod +x /etc/services.sh


ENTRYPOINT /etc/services.sh
