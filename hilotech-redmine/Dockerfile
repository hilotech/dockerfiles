FROM centos:centos6
MAINTAINER hilotech.jp
##
#   @container Redmine /w Apache on centos:centos6
#   @author SHIOBARA -Bar- Hiroaki
#   @update 2014/07/24
##

ENV _MY_FQDN        your.redmine.site.domain.com
ENV _DATABASE_HOST  your.redmine.mysql.com
ENV _DATABASE_NAME  your-redmine-mysql-database-name
ENV _DATABASE_USER  your-redmine-mysql-user-name
ENV _DATABASE_PASS  your-redmine-mysql-user-password
ENV _SMTP_RELAY     smtp-server-for-relaying


# networking
RUN echo "NETWORKING=yes" > /etc/sysconfig/network
EXPOSE 80


# timezone
RUN \
    echo 'ZONE="Asia/Tokyo"' > /etc/sysconfig/clock && \
    rm -f /etc/localtime && \
    ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime


# packages
RUN \
    yum -y install perl yum-utils telnet wget which && \
    perl -07 -pi \
        -e 's/(centosplus.*?enabled)=0/$1=1/igs;' \
        /etc/yum.repos.d/CentOS-Base.repo && \
    echo 'multilib_policy=best' >> /etc/yum.conf && \
    yum -y groupinstall Base  && \
    yum -y remove sendmail && \
    yum -y install \
        postfix mailx cronie sysstat \
        gd gd-devel freetype freetype-devel \
        mysql mysql-devel mysql-libs \
        libyaml-devel zlib-devel tk-devel readline-devel \
        perl-DBI perl-DBD-MySQL openssl-devel ncurses-devel \
        libffi-devel gdbm-devel db4-devel curl-devel \
        ImageMagick-devel ImageMagick \
        libxml2-devel libxslt-devel \
        graphviz graphviz-gd ipa-gothic-fonts.noarch && \
    yum -y update


# some magic spell
RUN echo "relayhost = [$_SMTP_RELAY]" >> /etc/postfix/main.cf


# Ruby installation
RUN \
    yum -y install \
        gcc make openssl-devel git
RUN if [ ! -d /usr/src/netsrc ]; then mkdir -p /usr/src/netsrc; fi
RUN \
    cd /usr/src/netsrc && \
    wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.2.tar.gz && \
    tar xvzf ruby-2.1.2.tar.gz && \
    cd ruby-2.1.2 && \
    ./configure \
        --prefix=/usr && \
    make && make install && \
    echo 'gem: --no-ri --no-rdoc' >> ~/.gemrc && \
    gem install haml && \
    rm -rf /usr/src/netsrc/ruby-2.1.2
    rm -f ruby-2.1.2.tar.gz


# Apache /w Passenger installation
RUN \
    yum -y install \
        gcc-c++ httpd httpd-devel apr apr-devel apr-util apr-util-devel
RUN if [ ! -d /etc/httpd/vhosts ]; then mkdir -p /etc/httpd/vhosts; fi
RUN \
    gem install passenger && \
    passenger-install-apache2-module --auto --language ruby && \
    passenger-install-apache2-module --snippet > /etc/httpd/conf.d/passenger.conf && \
    sed -i \
        -e 's/^#NameVirtualHost.*$/NameVirtualHost *:80\nInclude vhosts\/*.conf/i;' \
        /etc/httpd/conf/httpd.conf && \
    echo -e "\
<VirtualHost *:80>\n\
    ServerName ${_MY_FQDN}\n\
    DocumentRoot /opt/redmine/public\n\
    <Directory /opt/redmine/public>\n\
         AllowOverride all\n\
         Options -MultiViews\n\
    </Directory>\n\
</VirtualHost>\n\
" > /etc/httpd/vhosts/${_MY_FQDN}.conf


# Redmine base installation
RUN if [ ! -d /usr/src/netsrc ]; then mkdir -p /usr/src/netsrc; fi
RUN if [ ! -d /opt ]; then mkdir -p /opt; fi
RUN \
    cd /usr/src/netsrc && \
    wget http://www.redmine.org/releases/redmine-2.5.2.tar.gz && \
    tar xvzf redmine-2.5.2.tar.gz && \
    mv redmine-2.5.2 /opt/redmine && \
    cd /opt/redmine && \
    gem install nokogiri -- --use-system-libraries && \
    gem install bundler --no-rdoc --no-ri && \
    cp /opt/redmine/config/database.yml.example /opt/redmine/config/database.yml && \
    perl -07 -pi \
        -e "s/(production:.*?database:\s+)(.*?)\n/\${1}$_DATABASE_NAME\n/igs;" \
        -e "s/(production:.*?host:\s+)(.*?)\n/\${1}$_DATABASE_HOST\n/igs;" \
        -e "s/(production:.*?username:\s+)(.*?)\n/\${1}$_DATABASE_USER\n/igs;" \
        -e "s/(production:.*?password:\s+)(.*?)\n/\${1}$_DATABASE_PASS\n/igs;" \
        /opt/redmine/config/database.yml && \
    cp /opt/redmine/config/configuration.yml.example /opt/redmine/config/configuration.yml && \
    sed -i \
        -e 's/^production:/#production:/i;' \
        -e 's/^\(  *attachments_storage_path:\)/\1 \/opt\/redmine\/files/i;' \
        -e 's/^\(  *rmagick_font_path:\)/\1 \/usr\/share\/fonts\/ipa-gothic\/ipag.ttf/i;' \
        /opt/redmine/config/configuration.yml && \
    echo -e "\
\n\
production:\n\
  email_delivery:\n\
    delivery_method: :smtp\n\
    smtp_settings:\n\
      address: "$_SMTP_RELAY"\n\
      port: 25\n\
\n\
" >> /opt/redmine/config/configuration.yml && \
    echo "gem 'mail-iso-2022-jp'" > Gemfile.local && \
    bundle install && \
    gem install activerecord-mysql-adapter && \
    bundle exec rake generate_secret_token && \
    RAILS_ENV=production bundle exec rake db:migrate


# Redmine themes and plugins
RUN \
    rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.i686.rpm && \
    perl -07 -pi \
        -e 's/(\[rpmforge\].*?enabled *)= *1/$1=0/is;' \
        /etc/yum.repos.d/rpmforge.repo && \
    yum -y install --enablerepo=rpmforge-extras subversion && \
    yum -y install mercurial
RUN \
    cd /opt/redmine/public/themes && \
        git clone https://github.com/farend/redmine_theme_farend_basic.git && \
        git clone https://github.com/farend/redmine_theme_farend_fancy.git && \
        git clone https://github.com/makotokw/redmine-theme-gitmike.git
RUN \
    cd /opt/redmine/plugins && \
        hg clone https://bitbucket.org/haru_iida/redmine_code_review && \
        hg clone https://bitbucket.org/kusu/redmine_work_time && \
        git clone https://github.com/suer/redmine_absolute_dates.git && \
        git clone https://github.com/daipresents/redmine_parking_lot_chart.git && \
        hg clone https://bitbucket.org/akiko_pusu/redmine_issue_templates && \
        git clone https://github.com/ameya86/redmine_maintenance.git && \
        git clone https://github.com/danmunn/redmine_dmsf.git && \
        git clone https://github.com/akiko-pusu/redmine_banner.git && \
        svn co http://subversion.andriylesyuk.com/sidebar-content/ sidebar && \
        svn co https://github.com/teleological/redmine_stealth && \
        hg clone https://bitbucket.org/haru_iida/redmine_logs && \
        git clone git://github.com/splendeo/redmine-environment-css.git && \
        git clone git://github.com/commandprompt/redmine_pastebin.git && \
        hg clone https://bitbucket.org/haru_iida/redmine_theme_changer && \
        git clone https://github.com/samchen2009/ekanban.git && \
        git clone https://github.com/AZielinski/Redmine-Todo-lists.git && \
    cd /opt/redmine && \
    bundle install --without xapian && \
    bundle exec rake redmine:plugins:migrate RAILS_ENV=production


# permissions and services 
RUN chown -R apache.apache /opt/redmine
RUN \
    chkconfig httpd on \
    chkconfig crond on \
    chkconfig postfix on


# invoke services
RUN echo -e '\
#!/bin/sh\n\
service httpd start\n\
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
