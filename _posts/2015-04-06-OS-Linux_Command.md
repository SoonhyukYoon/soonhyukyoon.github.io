---
layout: post
title:  "Linux 환경 서버 구축시 자주쓰는 명령어"
date:  2015-04-06
categories:
- OS
tags:
- CentOS
- RHEL
published: true
---
Linux 환경 서버 구축시 자주쓰는 명령어
---

==== CPU정보
watch cat /proc/cpuinfo

==== 메모리 정보
watch cat /proc/meminfo
free -m

==== NTP 수정방법
0. # rpm -qa | grep ntp --> 없으면  yum -y install ntp
1. # vi /etc/ntp.conf  --> server 추가
2. # ntpdate <NTP_SERVER_HOST> --> 1차 동기화
3. # service ntpd start
4. # chkconfig ntpd on
5. # ps -ef | grep ntp   OR   ntpq -n -p

==== hostname 변경
1. # hostname <HOSTNAME>
2. # vi /etc/hosts --> 127.0.0.1  <HOSTNAME>
3. vi /etc/sysconfig/network --> HOSTNAME=<HOSTNAME>

==== Daemon 서비스 확인
/usr/sbin/ntsysv

==== TAR 사용법
압축 : prompt > tar -zcvf Test.tar Test
해제 : prompt > tar -zxvf Test.tar

==== 기동시간 확인
uptime

==== 리눅스 메모리 정리 방법
sync;echo 3 > /proc/sys/vm/drop_caches

==== 리눅스 파일내용 일괄 수정
find ./ -name "*.php" -exec perl -pi -e 's/OLD_TEXT/NEW_TEXT/g' {} \;

==== JAVA Alternatives
수정 : alternatives --install /usr/bin/java java /usr/java/jdk1.6.0_29/bin/java 100
확인 : alternatives --config java

==== 좀비 프로세스 죽이기
ps -ef | grep defunct | awk '{print $3}' | xargs kill -9

==== 파일에서 내용 찾기
find 경로 -name "파일명" | xargs grep "찾을패턴"

==== Network Outbound 확인하고 싶을때
netstat -nputw

==== 계정 패스워드 상세 정보
chage -l <계정>