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
## 잡다한 잊어먹기 쉬운 Command 정리

### CPU정보

```shell
watch cat /proc/cpuinfo
```

### 메모리 정보

```shell
watch cat /proc/meminfo
free -m
```

### NTP 수정방법

```shell
[root] rpm -qa | grep ntp --> 없으면  yum -y install ntp
[root] vi /etc/ntp.conf  --> server 추가
[root] ntpdate <NTP_SERVER_HOST> --> 1차 동기화
[root] service ntpd start
[root] chkconfig ntpd on
[root] ps -ef | grep ntp   OR   ntpq -n -p
```

### hostname 변경

```shell
[root] hostname *HOSTNAME*
[root] vi /etc/hosts -(추가)-> 127.0.0.1 *HOSTNAME*
[root] vi /etc/sysconfig/network -(편집)-> HOSTNAME=*HOSTNAME*
```

### Daemon 서비스 확인

```shell
/usr/sbin/ntsysv
```

### TAR 사용법

```shell
[압축] tar -zcvf Test.tar Test
[해제] tar -zxvf Test.tar
```

### 기동시간 확인

```shell
uptime
```

### 리눅스 캐쉬 메모리 정리

<b>운영중인 시스템에 drop_caches 조정은 위험하고, 지양할 것.</b>

Clearning the Linux Memory cache can be a quick way to regain system resources. Writing to the drop_cache process will cause the kernel to drop clean caches, dentries and inodes from memory, causing that memory to become free.

To free pagecache:# echo 1 > /proc/sys/vm/drop_caches
To free dentries and inodes:# echo 2 > /proc/sys/vm/drop_caches
To free pagecache, dentries and inodes:# echo 3 > /proc/sys/vm/drop_caches
As this is a non-destructive operation, and dirty objects are not freeable, the user should run "sync" first in order to make sure all cached objects are freed.

```shell
sync;echo 3 > /proc/sys/vm/drop_caches
```

### 리눅스 파일내용 일괄 수정

```shell
find ./ -name "*.php" -exec perl -pi -e 's/OLD_TEXT/NEW_TEXT/g' {} \;
```

### JAVA Alternatives

* Jenkins 등 CI 도구에서 대상 서버로 소스 배포 이후 Ant를 활용한 Remote WAS 중단/기동 상황시 유용하다.
* 여러 버전의 Java가 설치되었을 경우 대표 java 명령어 설정.

수정 : alternatives --install /usr/bin/java java /usr/java/jdk1.6.0_29/bin/java 100
확인 : alternatives --config java

### 좀비 프로세스 죽이기

```shell
ps -ef | grep defunct | awk '{print $3}' | xargs kill -9
```

### 파일에서 내용 찾기

```shell
find 경로 -name "파일명" | xargs grep "찾을패턴"
```

### Network Outbound 확인하고 싶을때

```shell
netstat -nputw
```

### 계정 패스워드 상세 정보

```shell
chage -l <계정>
```