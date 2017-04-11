---
layout: post
title:  "RabbitMQ 설치 및 구성"
date:  2016-08-22
categories:
- MQ
tags:
- RabbitMQ
published: true
---
### RabbitMQ란?

* RabbitMQ는 간단하게 말하면 AMQP (Advanced Message Queueing Protocol) 기반의 오픈 소스 메시지 브로커 소프트웨어(Message broker)이다.
* Message Broker는 프로세스 또는 서버간 데이터(메시지) 전달을 약속된 프로토콜에 따라 효율적으로 송/수신 하기 위해 저장 및 전달을 수행하는 연결자(서버)를 말한다.
* 저장은 전송 과정에 있어 메시지 유실을 방지하고 한정된 전송 대역을 유지하는 방법의 제공이고 전달은 동기/비동기, Text/Stream, Failover, Retry 등 정확하고 효과적인 메시지 전달 방법에 대한 기술 제공을 의미한다.
* 위와 같이 서로 다른 시스템간 비용/기술/시간적인 측면에서 최대한 효율적인 방법으로 메시지를 교환하기 위한 MQ 프로토콜 중 RabbitMQ가 주력으로 사용하는 프로토콜은 AMQP(Advanced Message Queing Protocol)이다.
* Message Broker가 가지고 있는 비동기/분산/다중 프로토콜 메시징 기능은 모바일 서버 생태계의 데이터 Push/UX 상호작용/데이터 수집 채널 등 다양한 요건을 만족할 수 있다.

### RabbitMQ 특/장점

* 표준 AMQP (Advanced Message Queueing Protocol) 메세지 브로커 소프트웨어(message broker software)
* 메시지 중계(Topic) 및 Queueing(Queue) 모두 지원: 즉, Pub/Sub과 Produce/Consume 모두 쉽게 설계 가능
* Java Integration 개발 지원(Spring Integration AMQP, Spring Rabbit)
* 관리 Web 페이지를 제공하여 내부 자원(Queue, Topic) 설정 및 모니터링 가능
* 주요 운영체제에서 모두 실행 가능
* 오픈 소스 제공 뿐만 아니라 상용 및 사후 지원 제품(Pivotal RabbitMQ)을 제공
* HA(Highly Available) 구성이 편하다.

### 설치 사전 과정

*아래의 설치과정은 Redhat 계열 리눅스 대상 설치를 전제함*

1. Erlang(OTP) 설치

* OS 기본 설정은 이곳에 작성된 'CentOS & RHEL 최소 설정' 글을 참고하고 여기 추가적인 컴파일 라이브러리 설치

   ```shell
   yum install glibc glibc-devel make ncurses-devel autoconf libssl-dev
   ```

* 작성자는 yum으로 erlang 설치를 좋아하지 않으므로 http://www.erlang.org/ 에서 OTP(Java로 따지면 JDK)를 다운받는다.

   - 단, http://www.rabbitmq.com/which-erlang.html 페이지를 꼭 참고해서 적당한 버전을 다운로드 해야한다.
   - 자신없으면 가장 최근 안정화(Stable) 버전

* 다운로드 및 컴파일

  ```shell
  [root] mkdir -p /usr/local/erlang_inst && cd /usr/local/erlang_inst
  [root] wget http://erlang.org/download/otp_src_18.3.tar.gz
  [root] tar -zxvf otp_src_18.3.tar.gz
  [root] cd otp_src_18.3
  [root] ./configure
  [root] make && make install
  # 컴파일 로그 잘 확인한 다음 설치 확인
  [root] erl
  ```

2. /etc/profile 설정

   ```shell
   [root] vi /etc/profile
   ## erlang 추가
   export ERL_HOME=/usr/local/lib/erlang
   export PATH=$PATH:$ERL_HOME/bin
   (저장 후 적용)
   [root] source /etc/profile
   ```

### RabbitMQ 설치

1. 다운로드 및 압축해제

   ```shell
   [rabbitmq] mkdir -p /engn001/rabbitmq/ && cd /engn001/rabbitmq
   [rabbitmq] wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-generic-unix-3.6.5.tar.xz
   [rabbitmq] tar -xvf rabbitmq-server-generic-unix-3.6.5.tar.xz
   [rabbitmq] chown rabbitmq.rabbitmq rabbitmq_server-3.6.5
   [rabbitmq] cd rabbitmq_server-3.6.5
   ```

2. 환경설정

*간단한 환경설정 예시*

* ~/sbin/rabbitmq-defaults

```shell
vi /engn001/rabbitmq/rabbitmq_server-3.6.5/sbin/rabbitmq-defaults

# 아래의 부분을 편집하고 저장
### next line potentially updated in package install steps

## RABBIT_HOME 재정의
RABBITMQ_HOME=/engn001/rabbitmq/rabbitmq_server-3.6.5

SYS_PREFIX=${RABBITMQ_HOME}

### next line will be updated when generating a standalone release
ERL_DIR=

CLEAN_BOOT_FILE=start_clean
SASL_BOOT_FILE=start_sasl

if [ -f "${RABBITMQ_HOME}/erlang.mk" ]; then
    # RabbitMQ is executed from its source directory. The plugins
    # directory and ERL_LIBS are tuned based on this.
    RABBITMQ_DEV_ENV=1
fi

## Set default values

BOOT_MODULE="rabbit"

CONFIG_FILE=${SYS_PREFIX}/etc/rabbitmq/rabbitmq

## 상용 레벨에서는 로그와 DB 경로는 별도의 파티션으로 관리되는 것이 좋다##
LOG_BASE=${SYS_PREFIX}/var/log/rabbitmq
MNESIA_BASE=${SYS_PREFIX}/var/lib/rabbitmq/mnesia

ENABLED_PLUGINS_FILE=${SYS_PREFIX}/etc/rabbitmq/enabled_plugins

PLUGINS_DIR="${RABBITMQ_HOME}/plugins"

CONF_ENV_FILE=${SYS_PREFIX}/etc/rabbitmq/rabbitmq-env.conf

## Thread Pool 설정 (기본값)
IO_THREAD_POOL_SIZE=64

## Node 이름 (반드시 '@' 뒤에는 Hostname)
NODENAME=node1@MY_HOSTNAME
```

* ~/etc/rabbitmq/enabled_plugins

```shell
vi /engn001/rabbitmq/rabbitmq_server-3.6.5/etc/rabbitmq/enabled_plugins

# 아래와 같이 편집하고 저장
[rabbitmq_management,rabbitmq_management_agent,rabbitmq_management_visualiser].
```

* ~/etc/rabbitmq/rabbitmq.config

```shell
cd ./etc/rabbitmq/
wget --no-check-certificate https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/stable/docs/rabbitmq.config.example
cp rabbitmq.config.example rabbitmq.config
vi rabbitmq.config

# 아래와 같은 부분을 편집하고 저장 (편집시 ',' 문자에 주의한다. 마지막 속성 끝에 ','이 붙지 않도록 주의)

   %% To listen on a specific interface, provide a tuple of {IpAddress, Port}.
   %% For example, to listen only on localhost for both IPv4 and IPv6:
   %%
   %% {tcp_listeners, [{"127.0.0.1", 5672},
   %%                  {"::1",       5672}]},
   %% Listen
   {tcp_listeners, [{"0.0.0.0", 5672}]},

(중간 생략)

   %% Log levels (currently just used for connection logging).
   %% One of 'debug', 'info', 'warning', 'error' or 'none', in decreasing
   %% order of verbosity. Defaults to 'info'.
   %%
   %% {log_levels, [{connection, info}, {channel, info}]},
   %% 로그 레벨 설정
   {log_levels, [{connection, warning}, {channel, warning}]},

(중간 생략)

   %% Set the max permissible number of channels per connection.
   %% 0 means "no limit".
   %%
   %% {channel_max, 128},
   %% 최대 Connection Channel 값
   {channel_max, 128},

(중간 생략)

   %% Customising Socket Options.
   %%
   %% See (http://www.erlang.org/doc/man/inet.html#setopts-2) for
   %% further documentation.
   %%
   %% {tcp_listen_options, [{backlog,       128},
   %%                       {nodelay,       true},
   %%                       {exit_on_close, false}]},
   %% TCP Socket 기본 설정
   {tcp_listen_options, [{backlog,       1024},
                         {nodelay,       true}]},

(중간생략)

   %% Alternatively, we can set a limit (in bytes) of RAM used by the node.
   %%
   %% {vm_memory_high_watermark, {absolute, 1073741824}},
   %%
   %% Or you can set absolute value using memory units.
   %%
   %% {vm_memory_high_watermark, {absolute, "1024M"}},
   %%
   %% Supported units suffixes:
   %%
   %% k, kiB: kibibytes (2^10 bytes)
   %% M, MiB: mebibytes (2^20)
   %% G, GiB: gibibytes (2^30)
   %% kB: kilobytes (10^3)
   %% MB: megabytes (10^6)
   %% GB: gigabytes (10^9)
   %% Erlang VM 메모리 사용 한계 설정 (설치 서버 상황에 따라 다르게 설정 필요)
   {vm_memory_high_watermark, {absolute, "1024M"}},

(중간생략)

   %% Set disk free limit (in bytes). Once free disk space reaches this
   %% lower bound, a disk alarm will be set - see the documentation
   %% listed above for more details.
   %%
   %% {disk_free_limit, 50000000},
   %%
   %% Or you can set it using memory units (same as in vm_memory_high_watermark)
   %% {disk_free_limit, "50MB"},
   %% {disk_free_limit, "50000kB"},
   %% {disk_free_limit, "2GB"},
   %% Disk 작성되는 내장 DB 사이즈
   {disk_free_limit, "2GB"}
```

* iptables

```shell
vi /etc/sysconfig/iptables
(iptables 를 사용하고 있다면 아래의 설정 추가)

# RabbitMQ
-A INPUT -m state --state NEW -m tcp -p tcp --dport 5672 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 15672 -j ACCEPT

[root] service iptables restart
```

3. 실행

```shell
cd /engn001/rabbitmq/rabbitmq_server-3.6.5/sbin
(Daemon 실행)
./rabbitmq-server -detached
```

4. 상태 확인

```shell
(/engn001/rabbitmq/rabbitmq_server-3.6.5/sbin)
./rabbitmqctl status
```

* 로그
   - $RABBIT_HOME/var/log/rabbitmq/node1@MY_HOSTNAME.log

5. Management Console

*RabbitMQ의 Queue 구성 및 모니터링의 편의를 위한 구성*

* Management Console 계정 추가

```shell
(/engn001/rabbitmq/rabbitmq_server-3.6.5/sbin)
./rabbitmqctl add_user admin admin
./rabbitmqctl set_user_tags admin administrator
```

* 브라우저에서 서버IP:15672 로 접속한 다음 로그인 해본다.

<img src="/rabbitmq_console_home.png" />