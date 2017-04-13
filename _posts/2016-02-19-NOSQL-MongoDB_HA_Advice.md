---
layout: post
title:  "MongoDB HA 구성 조언 기록"
date:  2016-02-19
categories:
- NOSQL
tags:
- MongoDB
published: true
---
*MongoDB HA 구성 관련해서 들은 고수의 조언을 녹음한 내용을 기록함. 다는 잘 모르겠지만 나중을 위해 일단 기록해두고 공부ㅠ...*

##### 참고 이미지
<img src="http://www.kthdaisy.com/wp/wp-content/uploads/2014/11/Figure4_ShardReplica-665x395.png" />

*출처: [kthdaisy.com] 포스퀘어가 MongoDB를 선택한 이유 : Auto-Sharding*

* mongos서버는 Cache기반으로 데이터 정보를 유지하고, config 서버는 메타 정보를 유지하는 역할을 한다.

* mongod서버는 특정크기 만큼 데이터를 Chunk하는 작업을 수행한다. 이과정을 Split 이라고 하며, 이과정은 Document 사이즈가 늘어나면 자연히 발생하며 Disable 할수 없다.
   - Chunk된 조각을 다른 Cluster에 넘기는 과정을 Migration이라고 한다.
   - Chunk는 File Base로 일어난다.

* 데이터 용량이나 사용자 측면에서 소규모라면 그냥 mongod 서버만 쓰고 'Replica Sets'만으로만 구성 하는것도 상황에 따라서는 선택할 수 있다.
   - 'spring-data-mongo'에서 Master-Slave 설정이 가능함.

* Sharding 구성을 하지 않는다면 'spring-data-mongo'의 'master' 옵션 R/W속성 조합을 통해서 Failover 수행이 가능함.

* Sharding을 하는 경우 Sharding Key가 Chunk수행의 단위가 된다. 따라서 키 값의 선택이 중요하므로 응용 프로그램에서 Collection 데이터 키 중 가정 기준 값이라고 할 수 있는 unique 속성을 가지는 값으로 설정해야 한다. 따라서 '_id' 값을 선택하던지 커스텀 유니크 값을 생성하는게 좋다.
   - 커스텀 할 경우 가급적 특정 컬럼의 데이터의 hashing 값을 Sharding Key로 잡는다.
   - Hashing이 중요한 이유는 데이터 탐색이 더 잘되도록 Chunk 데이터가 잘 분배되게 해야하기 때문이다.
   - Collection마다 Hashing 기준을 다르게 할 수 있다.
   - 이러한 구성의 장점 중 하나는 Slave 데이터가 깨져도 복원-초기화가 가능하다는 것이다.

* 단일 서버 구성이면 Chunk 과정이 없다. 따라서 데이터가 늘어나면 탐색속도가 많이 느려진다.

<br>
##### 기타 잡지식
* MongoDB의 'Query-Timeout'은 Query 실행 옵션에 넣는다.
* majority사용은 주의해서 써야하며 '1'이상 설정은 성능저하를 유발하기 때문에 지양하는 것이 좋다.