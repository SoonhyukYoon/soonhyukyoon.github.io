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

### 구축 장점

* Java Integration 개발 지원(Spring Integration AMQP)이 강력하고
* 사용하기 편리하며(Console Admin)
* 주요 운영체제에서 모두 실행 가능하며
* 오픈 소스 제공 뿐만 아니라 상용-사후 지원 제품(Pivotal RabbitMQ) 모두 가능

