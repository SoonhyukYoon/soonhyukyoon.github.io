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

* /etc/profile 설정

   ```shell
   [root] vi /etc/profile
   ## erlang 추가
   export ERL_HOME=/usr/local/lib/erlang
   export PATH=$PATH:$ERL_HOME/bin
   (저장 후 적용)
   [root] source /etc/profile
   ```

2. RabbitMQ 설치

* 다운로드

   ```shell
   [root] mkdir -p /engn001/rabbitmq/ && cd /engn001/rabbitmq
   [root] wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-generic-unix-3.6.5.tar.xz
   [root] tar -xvf rabbitmq-server-generic-unix-3.6.5.tar.xz
   [root] cd rabbitmq_server-3.6.5
   ```
   