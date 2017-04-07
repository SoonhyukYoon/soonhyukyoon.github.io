---
layout: post
title:  "Apache Httpd 2.4 간단 구축"
date:  2016-05-12
categories:
- Web Server
tags:
- Apache
- HTTPD
published: true
---
### 설치 작업 전 확인사항

1. 시스템코드 확인 : 시스템 설치를 위한 구성정보 요청서에 명시된 5자리 이하의 입력코드

2. 엔진 설치여부 및 엔진관리 계정(apaadm) 생성 확인

3. 엔진 미설치시 openssl / GCC compiler 설치여부 확인

   - openssl과 openssl-devel (openssl lib로 openssl관련 헤더 파일정보가 있어 compile시 필요)

      예제>
      ```shell
      [root@XXXXXDEV01 extra]# rpm -qa | grep openssl
      openssl-0.9.8e-12.el5_4.1
      openssl-devel-0.9.8e-12.el5_4.1
      ```

   - GCC는 apache 바이너리 설치파일 compile시 필수적임

      예제>

      ```shell
      -bash-3.2$ rpm -qa | grep gcc
      compat-gcc-34-3.4.6-4
      compat-libgcc-296-2.96-138
      gcc-gfortran-4.1.2-48.el5
      gcc-gnat-4.1.2-48.el5
      gcc-java-4.1.2-48.el5
      libgcc-4.1.2-48.el5
      gcc-c++-4.1.2-48.el5
      gcc-objc-4.1.2-48.el5
      gcc-4.1.2-48.el5
      libgcc-4.1.2-48.el5
      ...등등...
      ```

      *없으면 설치: yum install gcc gc gcc-c++ make apr-util openssl openssl-devel zlib zlib-devel unzip perl*

4. 시스템코드+adm(application 관리) 계정 생성 확인

5. 사용 포트 선정 (가장 최근 설치된 도메인의 Port Set 확인 및 netstat -an | grep 포트)

### 설치 작업

1. apr, apr-util,pcre 다운로드 및 설치

   * 압축 풀어서 apache 컴파일일 경로에 옮긴다

   * APR: 아파치와 한꺼번에 컴파일
   ```shell
   mv ./apr-1.5.1 ./httpd-2.4.9/srclib/apr
   mv ./apr-util-1.5.3 ./httpd-2.4.9/srclib/apr-util
   ```

   * CRONOLOG (rotatelogs를 사용하지 않기로 결정할 경우)
   ```shell
   [root] cd cronolog
   [root] ./configure --prefix=/usr/local/cronolog 
   [root] make && make install
   ```

   * PCRE
   ```shell
   [root] cd pcre-8.35
   [root] ./configure --enable-unicode-properties=yes
   [root] make && make install
   ```

2. apache 컴파일 옵션 및 엔진 설치

   * apache 2.4.xx 설치

   ```shell
   [root] cd /engn001/apaadm/apache2
   [root] ./configure --prefix=/engn001/apaadm/apache24 --enable-modules=all --enable-mods-shared=most --enable-mpms-shared=all --enable-rewrite --enable-proxy --enable-so --enable-proxy-http --enable-proxy-connect --enable-cache --enable-mem-cache --enable-disk-cache --enable-deflate --enable-ssl --with-ssl=/usr/include/openssl --with-included-apr --with-included-apr-util --enable-nonportable-atomics=yes
   [root] make && make install
   ```

3. AJP compile 및 설치

   - tomcat-connectors 1.2.28 설치

   ```shell
   [root] cd tomcat-connectors-1.2.28-src
   [root] cd native
   [root] ./configure --with-apxs=/engn001/apaadm/apache24/bin/apxs; make ; make install
   ```

4. Apache 홈 디렉토리 구조 잡기

   - 엔진 홈 (예> /engn001/[서버기동계정:예) apaadm]/apache24/)
      - servers/ : 웹서버 인스턴스를 설치하기 위한 디렉토리로 임의로 생성함
      - bin/ : apache 인스턴스 기동 및 정지 등 실행에 관련된 쉘이 위치함
      - modules/ : *.so 등의 apache 에서 사용 가능한 모듈들의 파일이 위치
      - logs/ : apache에서 제공하는 기본 로그 디렉토리로 임의로 pid 등의 중요 로그를 위치시키는데 사용함
      - conf/ : apache에서 기본적으로 제공되는 template conf 파일이 위치하며 인스턴스별로 해당 디렉토리를 복사해서 사용함

5. Apache 인스턴스 집합 디렉토리 생성

   - 엔진 홈 디렉토리(예> /engn001/apaadm/apache24/) 아래 servers/ 하위에 인스턴스 디렉토리들이 위치함

   예제>
   ```shell
   [apaadm]$ cd /engn001/apaadm/apache24
   [apaadm]$ mkdir servers
   [apaadm]$ chmod 755 ./servers
   ```

6. 인스턴스 생성

   - 인스턴스 디렉토리 명은 xxxxx[_NN](xxxxx[_NN] : 5자리 이하의 시스템코드, 이중화 구성시 '_NN' 붙여 구분함)으로 디렉토리 생성함
   
   예제>
   ```shell
   [apaadm]$ cd /engn001/apaadm/apache24/servers
   [apaadm]$ mkdir xxxxx[_NN]
   [apaadm]$ chmod 755 ./xxxxx[_NN]
   ```

   ex> /engn001/apaadm/apache24/servers/test_10, /engn001/apaadm/apache24/servers/test_20

7. 인스턴스 conf 디렉토리 생성

   - 인스턴스 디렉토리 아래 apache에서 기본으로 제공되는 /engn001/apaadm/apache24/conf 디렉토리를 복사해서 수정하여 사용함

   예제>
   ```shell
   [apaadm]$ cd /engn001/apaadm/apache24/servers/xxxxx[_NN]
   [apaadm]$ cp -r /engn001/apaadm/apache24/conf /engn001/apaadm/apache24/servers/xxxxx[_NN]
   ```

8. 로그 디렉토리 생성

   - 로그를 위한 파일시스템 (/logs001 등) 이 존재하는 경우 : 로그 파일시스템 하위에 Apache관리계정명/apache24 디렉토리를 생성하고 해당 인스턴스를 위한 로그 디렉토리를 생성. 파일시스템 하위 로그 디렉토리를 바라보는 논리적인 링크인 logs를 각 인스턴스 디렉토리 아래에 생성

   예제>
   ```shell
   [apaadm]$ cd /logs001
   [apaadm]$ mkdir ?p apaadm/apache24/xxxxx[_NN]
   mkdir: cannot create directory `xxxxx': Permission denied
   [apaadm]$ ls -al
   drwxr-xr-x 13 root      root       4096 May  7 10:45 .
   drwxr-xr-x 38 root      root       4096 May  4 17:50 ..
   [apaadm]$ su (root 혹은 그에 준하는 권한 가진 계정으로 switch user)
   [apaadm]$ mkdir -p apaadm/apache24/xxxxx[_NN]
   [apaadm]$ chown -R apaadm:apaadma ./apaadm
   [apaadm]$ chmod 755 ./apaadm
   [apaadm]$ exit
   [apaadm]$ cd /engn001/apaadm/apache24/servers/xxxxx[_NN]
   [apaadm]$ ls
   conf  start.sh  stop.sh
   [apaadm]$ ln -s /logs001/apaadm/apache24/xxxxx[_NN] ./logs
   [apaadm]$ ls -rlt
   drwxr-xr-x  4 apaadm apaadm 4096 5?? 7 10:48 conf
   lrwxrwxrwx  1 apaadm apaadm 19  5?? 7 10:48 logs -> /logs001/apaadm/apache24/xxxxx[_NN]
   ```

9. 기동 쉘 작성

   - 해당 인스턴스 디렉토리 아래에 기동 쉘 작성

   예제>
   ```shell
   [apaadm]$ vi start.sh
   
   (편집)
   /engn001/apaadm/apache24/bin/apachectl -f /engn001/apaadm/apache24/servers/xxxxx[_NN]/conf/httpd.conf -k start
   
   [apaadm]$ chmod 744 start.sh
   ```

   - 해당 인스턴스 디렉토리 아래에 정지 쉘 작성

   예제>
   ```shell
   [apaadm]$ vi stop.sh
   
   (편집)
   /engn001/apaadm/apache24/bin/apachectl -f /engn001/apaadm/apache24/servers/xxxxx[_NN]/conf/httpd.conf -k stop
   
   [apaadm]$ chmod 744 stop.sh
   ```

10. 설정 변경

   1) http.conf 수정

   * 서비스 포트 지정

   ```shell
   #Listen 12.34.56.78:80
   Listen 6010
   #다수 서비스포트 추가시 Listen 포트 계속 추가함
   #Listen 6020
   ```

   * 유저/그룹 지정

   ```shell
   # User/Group: The name (or #number) of the user/group to run httpd as.
   # It is usually good practice to create a dedicated user and group for
   # running httpd, as with most system services.
   # 로그인 불가능한 계정으로 설정해야 함 : /bin/false
   User nobody
   Group nobody
   ```

   * 서버명 지정
   
   ```shell
   ServerName localhost
   ```

   * DocumentRoot 지정 및 directory index 비활성화 조치

   ```shell
   #DocumentRoot "/engn001/apaadm/apache24/htdocs"
   DocumentRoot "/sorc001/cppeadm/applications/htdocs"

   <Directory />
    Options FollowSymLinks
    AllowOverride None
    <LimitExcept GET POST HEAD>
    	Order deny,allow
    	Deny from all
    </LimitExcept>
   </Directory>

   #<Directory "/engn001/apaadm/apache24/htdocs">
   <Directory "/sorc001/qmntadm/applications/htdocs">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.2/mod/core.html#options
    # for more information.
    #
    Options -Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #
    AllowOverride None

    #
    # Controls who can get stuff from this server.
    #
    Order allow,deny
    Allow from all
   </Directory>
   ```

   * welcome 페이지 설정
   
   ```shell
   <IfModule dir_module>
    DirectoryIndex index.html index.jsp
   </IfModule>
   ```

   * 로그 수정
   
   ```shell
   #ErrorLog "logs/error_log"
   ErrorLog "|/engn001/apaadm/apache24/bin/rotatelogs /engn001/apaadm/apache24/servers/xxxxx[_NN]/logs/error_log.%Y.%m.%d 86400"

   #LogLevel warn
   LogLevel error

   #  LogFormat "%h %l %u %t \"%r\" %>s %b" common
   LogFormat "%h %l %u %t \"%r\" %>s %b %D" common

   #CustomLog "logs/access_log" common
   CustomLog "|/engn001/apaadm/apache24/bin/rotatelogs /engn001/apaadm/apache24/servers/xxxxx[_NN]/logs/access_log.%Y.%m.%d 86400" common
   ```

   * 주석풀고 경로 수정
   
   ```shell
   # Server-pool management (MPM specific)
   Include servers/xxxxx[_NN]/conf/extra/httpd-mpm.conf
   
   # Various default settings
   Include servers/xxxxx[_NN]/conf/extra/httpd-default.conf
   ```

   * 하단에 mod_jk 설정
   
   ```shell
   Include servers/xxxxx[_NN]/conf/mod_jk.conf
   ```

   * 상태 모니터링 페이지 설정
   
   ```shell
   <Location /server-status>
    SetHandler server-status
    Order deny,allow
    Deny from all
    Allow from 156.xx.xxx
   </Location>

   <Location /server-info>
    SetHandler server-info
    Order deny,allow
    Deny from all
    Allow from 156.xx.xxx
   </Location>
   ```

   * TRACE 메소드 무효화 (보안 이슈해결)
   
   ```shell
   TraceEnable Off
   ```

   2) httpd-default.conf 중 해당 설정을 아래와 같이 변경
   
   ```shell
   Timeout 300
   KeepAlive Off
             Off (불특정 다수 사용시)
             On (용량 큰 데이터 많이 오갈 때, 설정시 keepalive timeout 3초로)
   ServerTokens Prod
   ServerSignature Off
   HostnameLookups Off
   ```

   3) httpd-mpm.conf 중 해당 설정을 아래와 같이 변경
   
   ```shell
   PidFile "logs/xxxxx[_NN]_httpd.pid"
   LockFile "logs/xxxxx[_NN]_accept.lock"

   StartServers        4
   MaxClients         256
   MinSpareThreads    128
   MaxSpareThreads    256
   ThreadsPerChild     64
   MaxRequestsPerChild  0
   ```

   4) mod_jk.conf 생성
   
   ```shell
   [apaadm]$ cd /engn001/apaadm/apache24/servers/xxxxx[_NN]/conf
   [apaadm]$ vi mod_jk.conf

   # Load mod_jk module
   LoadModule jk_module modules/mod_jk.so

   # Specify the filename of the mod_jk lib
   <IfModule mod_jk.c>
   # Where to find workers.properties
   JkWorkersFile servers/xxxxx[_NN]/conf/workers.properties

   # Where to put jk logs
   JkLogFile "|/engn001/apaadm/apache24/bin/rotatelogs /engn001/apaadm/apache22/servers/xxxxx[_NN]/logs/modjk_log/mod_jk_%Y%m%d.log 86400"
   JkLogLevel error
   JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
   JkRequestLogFormat "%w %V %T"

   # JkOptions indicates to send SSK KEY SIZE
   # Note: Changed from +ForwardURICompat.
   # See http://tomcat.apache.org/security-jk.html
   JkOptions +ForwardkeySize +ForwardURICompatUnparsed -ForwardDirectories

   # Set the jk runtime status file
   JkShmFile servers/xxxxx[_NN]/logs/jk.shm

   # You can use external file for mount points
   # It will be checked for updates each 60 secondes
   # The format of the file is : /url=worker
   # /examples/*=loadbalancer
   # 정적 파일 경로는 Mod_JK 연동 안함
   SetEnvIf Request_URI "/resources/*" no-jk
   # 기타 연동안하는 URL 정의
   SetEnvIf Request_URI "favicon.ico" no-jk
   SetEnvIf Request_URI "/robots.txt" no-jk
   # RESTful 서비스 고려
   JkMount /* node_xxxx_01
   </ifModule>
   ```

   5) workers.properties 생성
   
   ```shell
   worker.list=node_xxxx_01,jkstatus

   # Templates을 선언해두어야 상속해서 확장하기 편하다
   worker.template.type=ajp13
   worker.template.maintain=60
   worker.template.lbfactor=1
   worker.template.ping_mode=A
   worker.template.ping_timeout=2000
   worker.template.prepost_timeout=2000
   worker.template.socket_timeout=60
   worker.template.socket_connect_timeout=2000
   worker.template.socket_keepalive=true
   worker.template.connection_pool_timeout=60
   worker.template.connect_timeout=10000
   worker.template.connection_pool_size=50
   worker.template.recovery_options=7

   # Set properties for node_xxxx_01(ajp13)
   worker.node_xxxx_01.reference=worker.template
   worker.node_xxxx_01.host=XXX.XXX.XXX.XXX
   worker.node_xxxx_01.port=8009

   # example) This is development server : Non Clustering
   #worker.xxxx_lb.type=lb
   #worker.xxxx_lb.balance_workers=node_xxxx_01
   #worker.xxxx_lb.method=Session
   #worker.xxxx_lb.sticky_session=True

   worker.jkstatus.type=status
   ```