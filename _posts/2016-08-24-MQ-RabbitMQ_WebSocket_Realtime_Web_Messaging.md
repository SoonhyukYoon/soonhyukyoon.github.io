---
layout: post
title:  "RabbitMQ와 WebSocket 기반 모바일 실시간 웹 메시징"
date:  2016-08-24
categories:
- MQ
tags:
- RabbitMQ
published: true
---
*작성자가 직접 수행한 의미있는 아키텍처 설계 및 구축 작업을 간략히 소개함*

### 실시간 웹 메시징

* ‘실시간 웹 메시징’은 모바일/PC 브라우저 환경에서 웹 접속 사용자가 수행하는 동작 또는 메시지 입력 같은 조작을 데이터로 변환하여 다른 사용자가 화면에 접속한 상태에서 실시간으로 브라우저 상에서 전달 받는 기능을 의미한다.

* 본 시스템을 사용하는 참여자들로 구성된 사용자 그룹을 임의로 구성하고 같은 목적을 가지고 특정 기능을 동시에 사용한다.

* 위의 기능요건을 시스템으로 표현 시 ‘실시간 웹 메시징’이라고 정의하며 사용 과정을 업무 기능으로 표현 시 ‘상호작용’ 이라고 명명한다.

* 구축의 요건은 소규모 동시 사용자 (약 5백명) 이지만, 업무 특성(교육)을 고려 HA 구성이 필수적이다.

* 기술 요구는 HTML5 기반 모바일 웹에 수행할 수 있는 실시간 웹 메시징이다.

### 실시간 웹 메시징 기술 상세

* 로그인 또는 별도 인증된 사용자만 실시간 웹 메시징을 수행할 수 있어야 하며 모바일 웹 시스템에 필요한 보안 요소를 지원해야 한다.

* 메시지를 송/수신 하는 범위는 ‘상호작용’ 기능을 활용하여 논리적으로 만들어진 참여자들로 구성된 사용자 그룹이다. 즉, 동일 ‘상호작용 그룹’에 속한 참여자(사용자)들끼리만 메시지를 송/수신 할 수 있다.

*	상호작용 그룹이 같을 경우 물리적 제약 없이 상호 메시지 전달이 가능해야 한다.

*	서버는 실시간으로 메시지를 전달하기 위해 Push 방식으로 하나의 Target Client에서 요청한 이벤트를 다른 Target Client에 전달한다.

*	Target Client의 기능 중단 없이 접속하는 사용자에게 기능을 제공하는 응용 프로그램 또는 서버간 Failover를 유도할 수 있어야 한다.

### Message Broker 필요성

* Target Client간 또는 Target Client와 서버간 실시간 웹 메시징 기능 수행을 구축일정 단축, 성능 및 분산/이중화 메시징 아키텍처 구성 측면을 고려하여. 별도 ‘Message Broker’ 같은 별도 미들웨어를 도입하여 메시지 송/수신 채널 역할을 수행한다.

* Message Broker: 다중 사용자-서버간 메시지 요청/응답을 중계/저장/전달처리 하는 미들웨어 또는 Server Side Application

* HTML5 Websocket 기능을 서버에서 효과적으로 제공하기 위해 반드시 필요한 Layer로 결정함.

### Message Broker 선택

*무지막지한 대형 시스템이 아닌이상, 간단하면서도 원하는걸 쉽게쉽게 만족하기에는 RabbitMQ 만한게 없다. 이 빠돌이!*

* Message Broker 도입 시 검토할 수 있는 기능 및 요건은 아래와 같다.

   - 반드시 웹 브라우저를 대상으로 하는 클라이언트 프로토콜(Websocket) 지원
   
   - 자체 인증 또는 외부 시스템 연계를 통한 확장 인증 방법 지원
   
   - 클러스터를 통한 이중화 구성 가능
   
   - 관리자 도구를 통한 전체적은 모니터링이 용이
   
   - 사용자 수신 전 까지 메시지를 유실하지 않고 저장하는 기능
   
   - 오픈 소스 임에도 불구하고 오랜 기간 전 세계적으로 검증되었거나 오픈 소스 및 상용화 버전을 모두 제공하면서 해당 분야에서 검증된 제품이어야 함

<img src="/message_broker_compare.png" />

   - RabbitMQ 결정 이유: 검토한 결과 위의 요건을 만족하고, Java Integration 개발 지원(Spring Framework 지원)이 강력하고, 사용하기 편리하고 (관리 Web 페이지 제공), 주요 운영체제에서 모두 실행 가능하며, 오픈 소스 제공 뿐만 아니라 상용 및 사후 지원 제품(vFabric RabbitMQ)을 제공한다. (고객의 OSS 거부감 배려)

### WebSocket 선택

<img src="/websocket.png" />

* 모바일/PC 브라우저 상에서 실시간 웹 메시징 기능은 HTML5의 Websocket 또는 SockJS 등 관련 기술 적용이 필요하다.

* Websocket 기술을 사용할 경우 PC 웹에서 제공하는 기능은 브라우저 제약을 감수 해야 한다.

* 사용자 환경(모바일 or PC의 경우 Chrome Browser 유도)을 고려하여 ‘Websocket’ 기술을 핵심으로 사용하기로 했다.

### Stomp

*Streaming Text Oriented Message Protocol, Simple Text Oriented Message Protocol; 약자*

* 스트리밍 및 단순 텍스트 지향 메시지 프로토콜 (텍스트/Binary 기반)

* HTTP로 부터 영감을 받은 디자인

* 메시지 지향 미들웨어에서 사용하기 위해 설계됨

* 다양한 언어, 플랫폼에서 클라이언트, 브로커 사용가능 (Client: Javascript, Server: SpringFramework)

* Websocket 조합과 함께 널리 사용되고 있다.

### 이렇게 구성해보았다

* 아키텍처 컨셉1

<img src="/rwms1.png" />

* 아키텍처 컨셉2: 요청 부터 응답까지

<img src="/rwms2.png" />

   - 왜 이렇게 했는가: 사용자가 참여하는 메시지는 이력유지가 되어야 하고 메시지를 수신해야 하는 사용자 그룹은 반드시 전달 받아야 한다.
   
   - 사용자 참여(요청) 메시지는 AJAX 요청이지만 서버에서는 동기식으로 처리하여 메시지를 서버에서 반드시 확보한다. (흘리지 않는다)
   
   - 흘리지 않기 위해 요청하는 메시지 이력은 별도 데이터 저장소에 관리한 다음 Message Broker에 전달한다.
   
   - 사용자는 Websocket 연결을 유지하기 위한 메커니즘을 제공받고 Push 형태로 사용자가 받아야 하는 메시지를 받는다.
   
   - 즉 서버가 Produce 역할을 수행하고 메시지 중요성(반드시 전달, 인스턴트)에 따라 클라이언트는 Consume 또는 Subscribe 수행.
   
   - 과정 요약
   
      1) RabbitMQ의 메세지는 ‘Producer’(WAS)로부터 발행이 되어 ‘Exchanges’(우체통이나 우체국에 해당함)에게 보내지게 된다.
   
      2) Exchanges는 메시지를 ‘Bindings’이라 불리는 룰에 의해 전달 되어야 하는 Queue에 분배한다.
      
      3) RabbitMQ가 Queue를 구독하는 ‘Consumers’(사용자-모바일/PC 웹 브라우저/웹앱)에게 메시지를 전달한다.

* 아키텍처 컨셉3: RabbitMQ 활용

<img src="/rwms3.png" />

   - RabbitMQ의 WebSocket-Stomp Adapter Plugin을 활용하여 사용자 Front End 연결 및 서버간 메시지 중계
   
   - 메시지 확인 중요도에 따라 관리자 등급은 Consume 방식의 Private Queue 할당
   
   - 사용자 그룹 참여자의 경우 Subscribe 방식의 동적 Queue(Topic 메시지 처리 특징)가 할당되며 메시지 수신 무결성 확보를 위해
   
      * Health Check (Stomp)
      
      * 일정 시간 이후 메시지 수신이 없나면 비동기로 마지막 시점 이후 메시지 데이터 Sync를 서버에서 받는다.

* 아키텍처 컨셉4: Topic Exchange

   - Topic Exchange 는 메시지를 이미 Exchange에 등록된 큐 중에서 ‘Routing Key’나 ‘Routing Key Pattern’이 매칭되는 경우 전달한다. 보통 Multicast 라우팅에 많이 쓰인다.
   
   - 실시간 웹 메시징은 1:N, N:1 (관리자가 상호작용 그룹 참여자에게 메시지 전달 또는, 여러 참여자가 해당 상호작용 그룹의 관리자에게 메시지를 전송) 방식의 메시징 설계가 필요하므로 RabbitMQ에서 제공하는 Exchange 종류 중 ‘Topic Exchange’를 활용(기본으로 제공하는 ‘amq.topic’ Exchange 활용)한다.
   
   - Routing key(Pattern)는 다음과 같이 dot<.>로 구분된 단어 리스트 형식을 가져야 한다.
   
   ```
   “단어.단어”, “단어.단어.단어”, “*.단어.*”, “단어.#” 등등..
   - Wildcard -
   1) *(star): 임의의 한 단어를 대신합니다.
   2) #(hash): 0 개 이상의 단어를 대신합니다.
   ```

   - Topic exchange는 다른 Exchange 타입과 같은 동작을 할 수 있는데, 만약 Routing Key=”#” 를 지정한다면 모든 message를 받기 때문에 ‘Fanout Exchange’와 같이 동작하며, routing key=”단어” 와 같이 Wildcard(‘#’, ‘*’)를 쓰지 않는다면 ‘Direct Exchange’와 같이 동작하게 된다.

   - Websocket 클라이언트 접속 후, Subscribe를 수행하면 RabbitMQ는 자동으로 ‘stomp-subscription-‘로 시작하는 Queue를 생성한 다음 클라이언트가 Subscribe하려는 Topic을 확인해서 해당 Queue와 자동으로 Binging 처리한다. 이렇게 자동으로 생성된 Queue는 클라이언트가 Subscribe를 종료(Connection 종료)하면 자동으로 삭제된다. 이러한 연동은 여려명의 참여자 클라이언트에서 사용한다.

   - 상호작용 참여 방 생성시 개발자는 제공되는 공통 모듈을 이용하여 미리 Queue(Name: “acdm.mo.interact.private.상호작용일련번호”, Binding: “acdm.mo.interact.*.상호작용 일련번호”)를 생성하고 사용자(관리자/참여자)는 해당 Queue에 접속하기 위해 특정 Queue에 Subscribe를 수행한다. 따라서 관리자 1인에게 전달되는 메시지를 처리(질문)하거나 상호작용 참여 그룹간 주고 받는 메시지를 모두 처리할 수 있다.
   
   - 이렇게 생성된 Queue는 장시간 미사용 시 Queue Expire 정책(RabbitMQ 설정: 1일)을 적용하여 접속이 끊겼더라도 삭제되지 않는다.
따라서 학습자는 상호작용 참여 방 생성 후 관리자의 접속 없이 바로 Action을 수행하더라도 관리자는 상호작용 화면에 돌입하기 전까지 현재까지 수신 받은 메시지를 한꺼번에 받을 수 있다. 이러한 메시징 아키텍처 Concept은 관리자가 전체 참여 인원과 화면을 공유하여 제공하기 네트워크 문제에도 사용자들 간 Action 메시지를 유실하지 않고 처리하기 위한 목적으로 설계되었다.

   - 이러한 메시징 아키텍처 구성은 STOMP 프로토콜 자체가 Topic또는 Queue에 대한 Subscribe를 선택하여 접속하는 것을 지원하기 때문에 가능하다. 메시지 전문에서 JSON Body를 제외한 상위 영역을 헤더라고 정의하는데, ‘/topic/~’, ‘/queue/~’로 시작하는 주소를 STOMP에서는 ‘destination’이라고 정의하는 헤더에 설정해야 한다.

* 응용 레벨

<img src="/rwms4.png" />

   - Spring Integration AMQP를 활용하고, Publish Channel을 설정하여 서버-RabbitMQ간 요청 채널 설정

<img src="/rwms2.png" />

   - 모바일 웹 채널 TLS/SSL 접속이 필요하다.
   
   - Client가 RabbitMQ에 직접 접속하기에는 네트워크 구성 제약(DMZ에 RabbitMQ 구성 불가 정책) 고려
   
   - RabbitMQ HA 구성시 Client 접근에 대한 Load Balancing 필요
   
   - 위와 같은 이유로 Apache HTTPD 서버의 'mod_proxy_wstunnel'를 활용한다.
   
   - Apache 서버를 이용해 Websocket 통신을 Proxy 연동 할 경우, 기존 웹 서버 공인인증서를 활용하여 WSS(TLS Websocket – HTTPS에 해당) 통신을 수행하기 용이하다.
   
   - Apache 서버는 일반 HTTP 요청은 WAS에 전달하며(MOD_JK), Websocket 요청일 경우 RabbitMQ의 Websocket Channel에 Bypass 한다.
   
   - WAS에 배포되는 응용 프로그램은 ‘Spring Integration AMQP’ 모듈을 통해 RabbitMQ에 메시지 Push를 요청한다. Spring Framework에 AMQP를 위한 연동 기능이 지원되기 때문에 Websocket과 무관하게 서버간 통신은 AMQP로 수행한다. ‘Topic Exchange’ 요청을 통해 다수 사용자(Fanout Exchange과 유사하게 동작) 또는 관리자가 메시지를 수신할 Queue에 집적 메시지를 전달(Direct Exchange와 유사하게 동작)할 수 있다.

### HA

<img src="/rabbitmq_ha.jpg" />

* 무중단 서비스 목적을 달성하기 위해 기본적으로 이중화 구성으로 서버를 구축하며 RabbitMQ서버의 경우 클러스터 구성을 통해 Failover를 가능하게 한다.
   - Web → RabbitMQ: Active/Active, Apache Websocket Proxy를 통한 Load Balancing (mod_proxy_wstunnel + mod_proxy_balancer 확장)
   - WAS → RabbitMQ: Active/Active, Spring Integration AMQP를 통한 Load Balancing
   - RabbitMQ: Cluster 구성, 참여자-Shared Queue(서버간 Queue 내용 공유), 관리자-HA Queue(서버간 동기화: Replication)을 통해 서로 다른 RabbitMQ서버에 접속한 사용자간 메시징 처리를 가능하게 하고 관리자의 경우 RabbitMQ서버 중 한대가 중단되더라도 Failover 가능(다른 서버에 복제된 Queue에 그대로 서비스 가능)

### 삽질의 순간

* mod_proxy_wstunnel 의 기능은 생각보다 별로 없다.

* RabbitMQ에 대한 접속 보안을 위해 'RabbitMQ HTTP Authentication Plugin'를 선택했지만 2016년 8월 당시 버전으로는 GET 방식의 연계만 가능했다...