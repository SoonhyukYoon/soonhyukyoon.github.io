---
layout: post
title:  "HttpComponents Connection pool keep-alive"
date:  2019-11-29
categories:
- HttpClient
tags:
- HttpClient
published: true
---
#### HttpComponents 의 Connection pool keep-alive 방식 재정리
- HttpClientBuilder 의 `setConnectionTimeToLive()`는 **응답과 다음 요청 사이에 TTL 을 의미하는게 아니다!!**
  - Connection Pool 생성 이후 소멸까지 생존 시간을 의미함
  - 따라서 이 설정만 사용하고 Keep-alive 정책을 설정하지 않는 경우 아래와 같은 조건에 대해서 오류 발생 가능성 있음
    - 상대 서버가 Nginx 등 웹서버 없는 상태에서 Keep-alive 처리가 전혀 없는 경우
    - 요청에 대한 응답이 Connection Pool TTL 안에 끝난 상황에서 커넥션 유지상태로 판단하고 다음 요청을 처리하려는
- 커넥션에 대한 관리는 3가지 방향으로 처리
  - Connection TimeToLive
    - 커넥션 생존 기간
    - 기본 값은 -1 (영구)
  - ConnectionKeepAliveStrategy
    - 서버 응답 헤더의 Keep-alive 스팩을 확인하고 커넥션 유효 기간을 지정
    - 서버에서 Keep-alive 헤더를 주지 않을 경우 영구적으로 유지로 판단
  - IdleConnectionEvictor
    - HttpClientBuilder.evictExpiredConnections()
    - HttpClientBuilder.evictIdleConnections()
    - IDLE 상태의 커넥션에 대해서 주기적으로 확인하고 제거하는 스케줄러 실행
      - 최대 IDLE 시간 만큼 주기적으로 실행, IDLE 시간 설정 없으면 10초 간격으로 실행
    - 스케줄 주기 및 트래픽에 따라 성능에 영향을 줄 수 있음
  - ConnectionReuseStrategy
    - 'ConnectionKeepAliveStrategy' 보다 상위 개념으로 설정하는 커넥션 재사용 정책
    - Keep-alive disable 을 가장확실하게 설정 가능: NoConnectionReuseStrategy
- 내가 생각하는 방향은
  - `Connection TimeToLive`는 활용하기에 예상치 못한 오류 발생가능성이 너무 높다: 기본 설정 그대로 두는게 좋다
  - `ConnectionKeepAliveStrategy` 는 Keep-alive 헤더를 주지 않을 경우 영구적으로 유지로 판단하는데
    - 대부분 서버 연동 시 Keep-alive 헤더는 응답에 없는 경우가 많기 때문에 기본 값(매우 짧은 시간)을 처리할 필요가 있다
  - `IdleConnectionEvictor` 는 필수가 아니라고 생각한다.
  - `ConnectionReuseStrategy` 를 활용하여 기본적으로 Keep-alive는 사용하지 않도록 처리한다
    - 만약에 연동 대상 서버가 커넥션 유지 정책이 확실하다면 `ConnectionKeepAliveStrategy`를 사용해 보완한다.