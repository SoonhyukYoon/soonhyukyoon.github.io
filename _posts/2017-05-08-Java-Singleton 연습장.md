---
layout: post
title:  "Singleton 및 잡다 연습장"
date:  2017-05-08
categories:
- Java
tags:
- Singleton
published: true
---
#### initialization on demand holder idiom

* Inner Class 로드 시점을 이용해서 멀티 스레드 상황의 Lazy 초기화 문제 해결 (처음에는 DLC-dual lock check- 나옴)
- 내장 클래스의 private static final로 Singleton 클래스 선언

#### enum 이 singleton pattern 으로 사용

* INSTANCE 가 생성될 때, multi thread 로 부터 안전하다. (추가된 methed 들은 safed 하지 않을 수도 있다.)
* 단 한번의 인스턴스 생성을 보장한다.
* 사용이 간편하다.
* enum value는 자바 프로그램 전역에서 접근이 가능하다.

#### Map
* Map: K/V, 중복 불가
* HashMap: Null Key 허용, 비동기, Key 값을 Hashing해서 보관, 인덱싱 성능 우수 - Hashing 기준으로 탐색하기 때문에 순서 보장 안됨
* HashTable: Null Key 허용 안함, 동기
* TreeMap: 정렬
* LinkedHashMap: 입력 순서 보장
* ConcurrentHashMap: HashMap의 톡성에 동기화 보장

#### Collection
* Set: 중복 불가, 탐색만가능 --> HashSet: 비동기, 순서없음, NULL허용 --> LinkedHashSet: 비동기, 순서보장, NULL 허용 --> TreeSet: 비동기, 정렬
* List: 중복허용, 순서보장 --> ArrayList: 비동기, 단방향 포인터-순차적접근빠름- --> Vector: 동기 --> LinkedList: 비동기, 양방향 포인터-추가/삭제빠름-, Stack/Queue/Deque 동작
* Queue: 중복허용, 우선순위에 따른 요소 순서 존재

#### Iterator
* Collection 객체 탐색을 위한 표준 인터페이스 (hasNext, next, remove)

#### HashMap 메모리 할당

* HashMap 객체가 최초 생성되면 HashMap 객체와 더불어 HashMap$Entry 객체 배열이 생성되고 배열의 기본 크기가 16개다.
* 텅 빈 HashMap 객체가 128byte의 메모리를 소비
* 추가: 입력 요소마다 추가 36 bytes 씩 증가

#### LinkedList 메모리 할당
* LinkedList의 기본 용량은 1인데, 새로운 요소가 추가되거나 기존의 요소가 제거될 때마다 즉각적으로 이 값이 변경된다.

#### ArrayList 메모리 할당
* ArrayList가 생성되었을 때, ArrayList 객체를 위해 32byte, 그리고 크기가 10인 Object 객체 배열을 위해 56byte가 할당되어 총 88byte가 할당된다.
* 데이터 추가시 새로운 사이즈의 배열(+1)을 다시 할당 (메모리 재할당)

#### ArrayList/LinkedList 저장 차이
* n개의 자료를 저장할 때, ArrayList는 자료들을 하나의 연속적인 묶음으로 묶어 자료를 저장하는 반면, LinkedList는 자료들을 저장 공간에 불연속적인 단위로 저장

#### GC 종류 별 특징 추가 정리

* Java8에서는 perm 영역이 metaspace로 변경되었으며 native 영역이므르 dump로 확인할 수 없다.
* young generation이 작으면 short live object가 old generation으로 넘어갈 확률이 높아지고, major full gc가 자주 발생하므로 느려진다.
* JSP 페이지가 너무 과다하게 deploy되면 Permanent 영역 누수있음
* Parallel GC: old generation gc는 단일 프로세서로 동작하여 serial collector와 동일함
* CMS GC:
   - young generation gc는 parallel Collector와 동일함 (full gc에서 4단계 수행)
   - 전체적인 gc에 걸리는 시간은 결코 짧지 않음, 짧게 멈추면서 자주 gc하는 방식
   - 여러 개의 cpu를 가지고, client에게 빠른 응답을 줘야 할 프로그램에서 사용하기 좋음
   - stop-the-world pause 가 자주 일어나지만 시간 자체를 짧게함
   - 메모리가 가득차면 Mark-Sweep-Compact 로 full gc를 수행하므로 old generation의 남은 용량이 중요함
   - CMS Collector의 단점
      - gc를 수행하는데 표시 작업을 3번이나 진행함
      - 중요: 다른 collector와 달리 compaction을 진행하지 않으므로 "메모리 단편화"가 발생함 (다른 collector는 compaction을 하므로 빈 공간 시작 주소값이 있지만, CMS는 그렇지 않음)
      - 다른 collector 보다 heap 영역을 더 사용함
      - 이미 marking했지만 더이상 사용하지 않는 객체도 발생함 : floating garbage

#### TPS

* TPS = Concurrent Users / Res. Time + Think Time (모바일은 대략 3초)
* Active User = TPS (위에서 구한) * Res. Time

#### tmpC

* TPC-C 벤치마크 시나리오에 대한 1분당 최대처리건수
* 동시사용자수×분당 트랜잭션(사용자수×트랜잭션 복잡도(50%))+인터페이스(가중치%)×네트워크 보정(30%)×피크 타임 보정(50%)×I/O 부하(20%)×년간 업무증가 및 여유율(연 20%)
* 메모리 용량 = {(OS 커널(100M)+[SGA()]+사용자수×5MB)+[Webserver()]+인터페이스(가중치%)}+여유율(30%)

#### Tomcat Connector

* Class: Http11NioProtocol		Http11Nio2Protocol		Http11AprProtocol
* Servlet 자체가 Blocking이기 때문에 NIO라도 어차피 Connection과 Thread는 비슷
* APR: APR은 Apache Web Server의 io module을 사용한다. 그래서 C라이브러리를 JNI 인터페이스를 통해서 로딩해서 사용하는데, 속도는 APR이 가장 빠른것으로 알려져 있지만, JNI를 사용하는 특성상, JNI 코드 쪽에서 문제가 생기면, 자바 프로세스 자체가 core dump를 내면서 죽어 버리기 때문에 안정성 측면에서는 BIO나 NIO보다 좋다.

#### Design Pattern

* Singleton, Builder, Factory Method, Composite, Template Method