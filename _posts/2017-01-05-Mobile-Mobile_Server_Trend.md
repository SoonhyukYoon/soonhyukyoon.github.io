---
layout: post
title:  "내가 생각하는 모바일 Backend 트렌드"
date:  2017-01-05
categories:
- MQ
tags:
- RabbitMQ
published: true
---
<img src="/mobile_server_trend.png" />

### Mobile Message Transfer

* Mobile Device & Service Mesh (가트너, ‘2016년 10대 전략 기술’)의 서버 기술 핵심은 다양한 디바이스간 메시징 처리 기술

   - http://www.gartner.com/smarterwithgartner/top-ten-technology-trends-signal-the-digital-mesh/

* Message Broker의 비동기/분산/다중 프로토콜 메시징 기능은 모바일 서버 생태계의 데이터 Push/UX 상호작용/데이터 수집 채널 등 다양한 요건을 만족할 수 있으므로 주요 OSS 영역이라 할 수 있다.

* 사업 규모 : Worldwide $10.3 billion in 2013 → $32.7 billion in 2020 (WinterGreen Research)

   - https://www.reportbuyer.com/product/963518/middleware-messaging-market-shares-strategies-and-forecasts-worldwide-2014-2020.html

### Cloud Delivery & OSS Prototyping

* OSS 기반의 모바일 생태계의 기술 선도 관건은 빠른 검증 및 활용 능력이다.

* 클라우드 환경을 통한 OSS Prototyping은 빠르고 실제 외부 환경의 모바일 서비스 검증에 효과적이다.

* Micro Service 같은 경량화 서비스 구성에 빠른 이행 및 초기 사업 도입/검증을 위해 서버 인력의 AWS 활용 능력이 뒷받침 되어야 한다.

   - 조금은 슬픈게 AWS Lambda를 보면 더 절감이 되는게 이제는 단순 Backend에는 많은 시간와 구조를 요구하려 하지 않는다. 훌륭한 회사들이 앞다퉈 Server-less Architecture를 제공하고 서비스 하려는 걸 보면 이제는 기술 중요도가 Frontend와 Deep-Backend(Big Data, AI)에 편중되고 있다할까? 살아남기 위한 노력의 시기가 왔고 Backend의 기술 중요가 아직 살아있는 메시징 기술에 대한 깊이를 다져야 할 것 같다. 나는 ㅋㅋ

### Infra + Application Service Monitoring/Analysis

* 단순 인프라 모니터링 뿐만 아니라 모바일 플랫폼/서비스는 내/외 연계를 통한 Data Mix 특성으로 인해 외부 연계에 대한 모니터링 및 이력 분석이 중요하다.

* Micro Service 같은 경량화 서비스 구성에 빠른 이행 및 초기 사업 도입/검증을 위해 AWS 활용 능력이 뒷받침 되어야 한다.