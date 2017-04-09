---
layout: post
title:  "MongoDB 이중화 구성 조언 기록"
date:  2016-02-19
categories:
- NOSQL
tags:
- MongoDB
published: true
---
*MongoDB 이중화 구성(Replication) 관련해서 들은 고수의 조언을 다는 잘 모르겠지만 나중을 위해 일단 기록해두고 공부하자...*

* mongos서버는 Cache기반으로 데이터 정보를 유지하고, config 서버는 메타 정보를 유지하는 역할을 한다.

* mongod서버는 특정크기 만큼 데이터를 Chunk하는 작업을 수행한다. 이과정을 Split 이라고 하며, 이과정은 Document 사이즈가 늘어나면 자연히 발생하며 Disable 할수 없다.

