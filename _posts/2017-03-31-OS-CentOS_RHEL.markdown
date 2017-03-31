---
layout: post
title:  "CentOS & RHEL 최소 설정"
date:  2017-03-31
categories:
- OS
tags:
- CentOS
- RHEL
published: true
---
CentOS & RHEL 최소 설정
---

기본
===

# PRE. 파티션 분리 및 기본 디렉토리 생성(/server, /web-source 또는 /app-source, /logs, -DB-/data)
# 계정생성 및 리눅스 환경 확인
- 한글확인 : locale (ko_KR.UTF-8)
- KST 확인 (ls /usr/share/zoneinfo/Asia/Seoul --> ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime)
- root 및 신규계정 패스워드 변경
- 번들로 설치된 httpd, mysql, svn 제거 (yum remove)
- Package 설치
{% highlight linux-config %}
yum install ntp gcc gc gcc-c++ make apr-util openssl openssl-devel zlib zlib-devel unzip perl
{% endhighlight %}
- sudo 권한이 필요한 관리 계정인 경우:  wheel 그룹에 계정 그룹(/etc/group)을 추가 후 다음을 실행 또는 직접 계정 추가 -- 개발서버만 할 것
{% highlight linux-config %}
$ visudo
## Allows people in group wheel to run all commands (주석제거)
%wheel  ALL=(ALL)       ALL
{% endhighlight %}
- jdk 설치: /usr/local/java
{% highlight linux-config %}
alternative 설정 (Jenkins 서버에서 재기동을 위해 기본 Java 실행 위치를 지정해주어야 한다)
$ alternatives --install /usr/bin/java java /usr/local/java/jdk1.7.0_67/bin/java 100
$ alternatives --config java (등록한 Java가 1번 으로 선택되어있어야 한다)
{% endhighlight %}
- iptables disable : vhost web 서버일 경우는 on

# hostname 재설정 (hostname, /etc/hosts, /etc/sysconfig/network) : 딱히 정해진 Rule은 없지만 대부분
   "회사(Shortcut) + H(HTTPD)/W(WAS)/D(DB, Cache) + 시스템 + [글로벌일 경우 Region] + D(개발)/P(운영) + 일련번호"
- API Service WAS-개발: HSWAPISVCD01
- API Service DB-운영 : HSDAPISVCP01

# logroatated 설정 (선택사항)
   /etc/logrotate.conf
{% highlight linux-config %}
   # see "man logrotate" for details
   # rotate log files weekly
   daily
   # keep 14 days worth of backlogs
   rotate 14
{% endhighlight %}

# NTP 설정 (/etc/ntp.conf)
{% highlight linux-config %}
# 자체 NTP 서버가 없을 경우
server 0.rhel.pool.ntp.org
server 1.rhel.pool.ntp.org
server 2.rhel.pool.ntp.org
{% endhighlight %}

설정 확인됬으면
{% highlight linux-config %}
$ ntpdate 0.rhel.pool.ntp.org
$ service ntpd start
$ chkconfig ntpd on
# 확인
$ ntpq -p
{% endhighlight %}

# FTP 설정(no guest) 및 불필요 계정 삭제
{% highlight linux-config %}
# CentOS/RHEL 종류, 버전에 따라 차이 있음
userdel adm
userdel lp
userdel ftp
userdel sync
userdel shutdown
userdel halt
userdel news
userdel uucp
userdel operator
userdel games
userdel gopher
userdel apache
userdel oprofile
userdel nfsnobody

groupdel adm
groupdel lp
groupdel news
groupdel uucp
groupdel games
groupdel dip
groupdel pppusers
groupdel slipusers
{% endhighlight %}

# Limit 'su' command (/etc/pam.d/su) 주석제거
{% highlight linux-config %}
#auth           required        pam_wheel.so use_uid
{% endhighlight %}
{% highlight linux-config %}
$ chmod 4750 /bin/su
{% endhighlight %}

(선택사항) root 계정 login 막을려면
/etc/ssh/sshd_config 파일의 'PermitRootLogin no' 수정 후,
{% highlight linux-config %}
service sshd restart
{% endhighlight %}

# 계정별 기본 설정
-   .bashrc
{% highlight linux-config %}
###########################################################
# Bash Setting
###########################################################
set -o vi
alias ll='ls -alF'  <-- 계정 비중을 생각해서 alias ll='ls -lF' 도 가능
alias vi=vim
(root 계정 전용)
DEFAULT="\[\033[m\]"
BG_BLUE="\[\033[44m\]"
BG_RED="\[\033[41m\]"
PS1=$BG_RED"[\u@\h \W]"$DEFAULT" \\$ "
export PS1
{% endhighlight %}

# profile 기본 설정 (/etc/profile)
{% highlight linux-config %}
###########################################################
# Profile Setting
###########################################################

# Add timestamp to .bash_history
export HISTTIMEFORMAT="%F %T "

# Add session timeout 1800
TMOUT=1800
HISTSIZE=10000
{% endhighlight %}

(JDK 설치 서버)
{% highlight linux-config %}
## JDK
export JAVA_HOME=/usr/local/java/jdk1.7.0_80
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
{% endhighlight %}

(MariaDB 설치 서버)
{% highlight linux-config %}
## MariaDB
export MARIADB_HOME=/engn001/maria/mariadb-10.0.21
export PATH=$PATH:$MARIADB_HOME/bin
{% endhighlight %}

# ulimit 환경 수정 (/etc/security/limits.conf)
작업전에 아래의 설정 존재여부 확인 후, 없으면 추가
/etc/pam.d/login
{% highlight linux-config %}
session required pam_limits.so
{% endhighlight %}
/etc/pam.d/sshd
{% highlight linux-config %}
session required pam_limits.so
{% endhighlight %}

/etc/security/limits.conf
{% highlight linux-config %}
*               soft    nofile          65536
*               hard    nofile          65536
*               soft    stack           20480
*               hard    stack           20480
### 여기서 일반
*               soft    nproc           65536
*               hard    nproc           65536
### Apache
# max user processes for apache (pending signals에 맞추면 좋다)
*               soft    nproc           158757
*               hard    nproc           158757
{% endhighlight %}

# System 커널 수정 (/etc/sysctl.conf  --> sysctl -p)
{% highlight linux-config %}
--- 일반 미들웨서 서버
##### max file count #####
fs.file-max = 65535

--- DB서버
##### max file count #####
fs.file-max = 6815744

### Disable IPv6 ###
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

--- Front End 계열 서버 설정
### MAX Connection ###
net.core.somaxconn = 4096 ~ 65535

--- 일반 미들웨서 서버
### MAX Connection ###
net.core.somaxconn = 2048

--- DB서버
### MARIA setting ### : 안해도 됨
net.core.rmem_default = 8388608
net.core.wmem_default = 8388608
net.core.rmem_max = 8388608
net.core.wmem_max = 8388608
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_retries2 = 5
net.ipv4.tcp_rmem = 8388608 8388608 8388608
net.ipv4.tcp_wmem = 8388608 8388608 8388608
net.ipv4.tcp_rfc1337 = 1
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_fin_timeout = 30
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_max_tw_buckets = 2000000

### REDIS Support ###
vm.overcommit_memory = 2
vm.overcommit_ratio = 99
vm.swappiness = 0
{% endhighlight %}

# 기타
/etc/rc.d
{% highlight linux-config %}
### for REDIS ###
echo never > /sys/kernel/mm/transparent_hugepage/enabled
{% endhighlight %}