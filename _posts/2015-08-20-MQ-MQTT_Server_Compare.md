---
layout: post
title:  "IoT 서비스를 위한 MQTT Message Broker"
date:  2015-08-20
categories:
- MQ
tags:
- MQTT
published: true
---
### MQTT란?

* MQ TELEMETRY TRANSPORT

* 텔레메트리 장치, 모바일 기기에 최적화된 라이트 메시징 프로토콜

* 제한된 통신 환경을 고려하여 디자인되었으며 Embedded Device to Server간 최적의 프로토콜로 주목받고 있다.

* 프로토콜이 차지하는 모든 면의 리소스 점유를 최소화한다.

   - 결국 베터리 문제이며 데이터 전송에 있어 사용되는 자원 소모를 최소화 하되 있을 건 다 있는 메시징 프로토콜

   - 느리고 품질이 낮은 네트워크의 장애와 단절에 대비하고 IoT 디바이스는 애플리케이션 동작에 자원 활용이 극히 제한적임을 고려한다.
   
   - 다수의 클라이언트 연결은 Publish/Subscribe 기반 메시징 제공
   
   - 신뢰성 있는 메시징을 위한 QoS(Quality of Service) 옵션을 제공한다.

### 서버는 어떻게 구성하나?

* 본인이 직접 아키텍처 설계를 수행한 모 통신사 OneM2M IoT 플랫폼은 아래와 같은 고려가 필요했다.

   -	QoS 2 까지 요청/응답 지원
   
   -	외부 HTTP(REST) 연동을 통해 동적으로 사용자 인증
   
   -	클러스터 구성 또는 불가할 경우 Bridge 지원
   
   -	SSL 통신 지원을 통한 보안 준수
   
   -	REST 방식의 관리자 API가 있을 경우면 좋지만 없을 경우 System Topic($SYS)을 통해 서버의 상태 및 클라이언트 접속에 대한 현황을 확인할 수 있어야 한다.

### MQTT Broker를 골라보자

<img src="/mqtt_broker.png" />