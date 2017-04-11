---
layout: post
title:  "Spring Data Redis(spring-data-redis)를 활용한 간단 개발"
date:  2015-10-06
categories:
- NOSQL
tags:
- Redis
- spring-data-redis
published: true
---
### Spring Data Redis를 활용하여 간단한 Redis 연계 방법을 기록한다.

* 환경은 Spring Framework 4.2.x 이상
* Maven Project 환경을 가정한다.

1. pom.xml의 dependency에 라이브러리 의존 추가
```xml
<!-- Spring Data Redis : Server Type -->
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
	<version>2.7.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.7.5.RELEASE</version>
</dependency>
```
*라이브러리 버전은 순전히 예시이며 실제 구성시 Redis 버전 및 Spring Framework 버전의 의존을 명확히 확인하여 적용한다.*

만약에 당장 Redis 서버를 구성하기는 어렵고 당장 테스트한번 해보고 싶다면 좀 오래되었기는 했지만 <a href="https://github.com/kstyrc/embedded-redis">Embedded Redis</a>서버 라이브러리 의존을 추가하여 Java 코드를 통해 Redis 서버를 로컬에서 실행해볼 수 있다.

```xml
<dependency>
	<groupId>com.github.kstyrc</groupId>
	<artifactId>embedded-redis</artifactId>
	<version>0.6</version>
</dependency>
```
*Embedded Redis 라이브러리는 많이 오래된 Redis 서버를 내장하고 있다. 상용목적으로 사용하기에는 무리가 있다.*

2. Spring Context 설정(XML)

* Spring Context 설정 경로에 'context-cache-redis.xml' 파일을 만들어서 아래와 같이 작성해본다.

*각 속성에 대한 설명은 주석 참조 (어디까지나 '간단 개발'을 위한 예시이다)*

```xml
	<!--=======================================================================
    = Spring Data Redis (Jedis Wrapper) 설정
    ==========================================================================-->

	<!--
		Redis Client
	-->
	<bean id="jedisConnFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="hostName" value="127.0.0.1" /> <!-- Redis IP -->
		<property name="port" value="6379" /> <!-- Redis Port -->
<!-- 		<property name="password" value="guest" /> 패스워드가 있을 경우 주석 해제 및 설정 -->
		<property name="timeout" value="5000" /> <!-- Connection Timeout -->
		<property name="usePool" value="true" /> <!-- Connection Pool 사용 여부 -->
		<property name="poolConfig">
			<bean class="redis.clients.jedis.JedisPoolConfig">
				<property name="maxTotal" value="5" /> <!-- 최대 Connection Pool 사이즈 -->
				<property name="maxWaitMillis" value="10000" /> <!-- Connection Pool 최대 대기 시간 -->
			</bean>
		</property>
	</bean>
    <!-- 만약에 Sentinel을 이용한 이중화 구성을 시도할 경우 아래 설정을 참고하자 (두 대의 Sentinel 예시)
	<bean id="jedisConnFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<constructor-arg name="sentinelConfig">
			<bean class="org.springframework.data.redis.connection.RedisSentinelConfiguration">
				<property name="master">
					<bean class="org.springframework.data.redis.connection.RedisNode">
						<property name="name" value="실제 서버의 마스터 노드의 이름" />
					</bean>
				</property>
				<property name="sentinels">
					<set>
						<bean class="org.springframework.data.redis.connection.RedisNode">
							<constructor-arg name="host" value="Sentinel 1번 Host" />
							<constructor-arg name="port" value="Sentinel 1번 포트" />
						</bean>
						<bean class="org.springframework.data.redis.connection.RedisNode">
							<constructor-arg name="host" value="Sentinel 2번 Host" />
							<constructor-arg name="port" value="Sentinel 2번 포트" />
						</bean>
					</set>
				</property>
			</bean>
		</constructor-arg>
		<property name="password" value="패스워드-없으면 주석처리" />
		<property name="timeout" value="5000" />
		<property name="usePool" value="true" />
		<property name="poolConfig">
			<bean class="redis.clients.jedis.JedisPoolConfig">
				<property name="maxTotal" value="최대 Connection Pool 사이즈" />
				<property name="maxWaitMillis" value="10000" />
			</bean>
		</property>
	</bean>-->

	<!--
		JSON Redis template definition: Jackson2를 이용한 JSON 데이터 저장 Template
		(POJO 또는 Map 객체를 JSON 형태로 Redis에 저장할 수 있다)
	-->
	<bean id="jsonRedisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="jedisConnFactory" />
		<property name="enableTransactionSupport" value="false" />
		<property name="keySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="valueSerializer">
			<bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />
		</property>
		<property name="hashKeySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="hashValueSerializer">
        	<!-- 예전 버전은 POJO 타입 고정 이었지만(즉, 타입별로 Template 생성ㄷㄷ) 이제는 타입 자체를 Cache에 같이 저장한다 -->
			<bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />
		</property>
	</bean>

	<!-- Default Redis template definition: 예제 전용 (Redis에 객체를 직렬화하여 저장) -->
	<bean id="defaultRedisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
		<property name="connectionFactory" ref="jedisConnFactory" />
		<property name="enableTransactionSupport" value="false" />
		<property name="keySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="valueSerializer">
			<bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
		<property name="hashKeySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
		<property name="hashValueSerializer">
			<bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
		</property>
	</bean>

	<!--
		Redis Cache Manager를 이용한 개별 Cache 관리 및 @Cache Annotation 연동 예시
	-->
	<bean id="redisCacheManager" class="org.springframework.data.redis.cache.RedisCacheManager">
		<constructor-arg ref="jsonRedisTemplate" />
		<property name="cacheNames">
			<list value-type="java.lang.String">
				<description>@Cache Annotation과 연계하려는 Cache를 아래에 등록한다 (자동생성)</description>
				<value>examlpleCache</value>
			</list>
    		</property>
		<property name="expires">
			<map>
				<description>위에 등록한 Cache이름 별로 Cache Expire Timeout을 설정해야 한다 (단위: 초)</description>
				<entry key="examlpleCache" value="60" />
			</map>
		</property>
	</bean>

	<!--======================================================================
	= Redis Cache 설정
	=========================================================================-->

	<!--
		응용에서 사용하는 Cache Bean은 아래에 정의한다
		('redisOperations' 속성은 객체 및 데이터 특성에 따라 설정한다: 객체를 JSON 형태로 Cache에 저장하는 경우 'jsonRedisTemplate' 설정)
	-->

	<!-- Sample Cache Bean -->
	<bean id="redisSampleCache" class="org.springframework.data.redis.cache.RedisCache">
		<!-- Cache Name -->
		<constructor-arg name="name" value="sampleCache" />
		<!-- Prefix: 반드시 Cache Name + ":" 으로 설정 -->
		<constructor-arg name="prefix" value="sampleCache:" />
		<!-- Rest Template 참조 -->
		<constructor-arg name="redisOperations"><ref bean="defaultRedisTemplate" /></constructor-arg>
		<!-- Cache 만료 시간: 초 -->
		<constructor-arg name="expiration" value="60" />
	</bean>

```

* RedisCache.prefix 속성: Redis에 저장되는 Cache 데이터들의 Key 중복 위험을 막기 위해 Cache Key앞에 붙이는 Group 구분자. 포스트 작성자는   Cache name + “:” 값으로 설정할 것을 권장한다.
   - 단일 Redis Database에서 여러 캐시를 상호 구분하여 사용해야 하는 경우 데이터 관리하기 용이하다.

3. Redis 연계 서비스 클래스를 만드는 예시는 아래와 같다.

*포스트 작성자는 간단한 개발에는 Spring Cache(org.springframework.cache.Cache == RedisCache) 추상 타입을 더 선호한다...*

*주석을 꼭 읽어봅시다*

```java
# CacheServiceImpl.java
/**
 * <PRE>
 * CacheService(Redis) 구현 클래스
 * </PRE>
 *
 * @author    윤순혁
 * @version   1.0
 * @see       CacheService
 */
@Service( "redisCacheService" )
public class CacheServiceImpl implements CacheService {

	private static final Logger LOGGER = LoggerFactory.getLogger( CacheServiceImpl.class );

	/**
	 * Redis Cache(sampleCache) Resource (Spring Cache 기본 인터페이스: 일반적인 업무기능 개발편의를 제공)
	 * 주의: RedisCache는 ValueOperation으로 동작한다. 
	 */
	@Resource( name = "redisSampleCache" )
	private Cache cache;

	/**
	 * Redis Template Resource (Redis 연동과 관련된 직접적인 기능을 활용 가능)
	 */
	@Resource( name = "defaultRedisTemplate" )
	private RedisTemplate<String, Object> redisTemplate;

	/**
	 * Cache 초기화
	 */
	@PostConstruct
	public void initializeCache() {
		try {
			// TODO 이곳에 Sample Cache 초기화 데이터를 추가하면 된다.
			putData( "키 입니다", "값 입니다" );
		} catch ( Exception ex ) {
			LOGGER.error( "[Sample Cache] Cache Initialize Error", ex );
		}
	}

	/* (non-Javadoc)
	 * @see com.lgcns.msp.guide.cache.service.CacheService#putData(java.lang.String, java.lang.Object)
	 */
	@Override
	public <T> void putData( String key, T value ) {
		// Spring Cache를 활용한 Redis 연동 예시
		cache.put( key, value );

		// Redis Template을 활용한 Value Operation 수행 예시
		/*
		 * (필독) 
		 * Redis Cache Key를 ':' 문자열을 붙인 패턴으로 저장하는 이유
		 * Redis Key를 구분자 및 인덱스로 활용 가능하기 때문에
		 * 만약 단일 Redis Database에서 여러 캐시를 상호 구분하여 사용해야 하는 경우 데이터 관리하기 용이하다.
		 * (참조: https://redislabs.com/blog/5-key-takeaways-for-developing-with-redis#.WDeob7KLQ1s)
		 * PS.
		 * 데이터 타입을 'Hash'로 하여 서로 다른 Cache를 키로 구분하고 데이터관리를 Map 형태로 처리하는 방법도 있다
		 * (단, 데이터가 급증 할 경우 Index Time 증가한다)
		 */
		redisTemplate.boundValueOps( Joiner.on( ':' ).join( cache.getName(), key ) ).set( value, 60, TimeUnit.SECONDS );
	}

	/* (non-Javadoc)
	 * @see com.lgcns.msp.guide.cache.service.CacheService#getData(java.lang.String, java.lang.Class)
	 */
	@Override
	public <T> T getData( String key, Class<T> paramClass ) {
		// Cache에서 Data를 가져온다
		return cache.get( key, paramClass );
	}

	/* (non-Javadoc)
	 * @see com.lgcns.msp.guide.cache.service.CacheService#removeData(java.lang.String)
	 */
	@Override
	public void removeData( String key ) {
		// Spring Cache를 활용한 Redis 연동 예시
		cache.evict( key );

		// Redis Template을 활용한 Value Operation 수행 예시
		redisTemplate.delete( Joiner.on( ':' ).join( cache.getName(), key ) );
	}
}

```

4. 테스트는 Controller 개발 또는 Mock 등을 통해 수행한다.

0. 기타: 테스트를 위한 로컬에서 Embedded Redis 실행

'1번' 과정에서 Embedded Redis 라이브러리 의존을 추가했다면 아래와 같이 Java를 개발하여 실행 가능하다.

* 주의: Embedded Redis 라이브러리 원리는 linux/windows redis 파일을 내장하고 있다가 임시 경로에 복사하고 실행하는 Java Process Wrapper이다. 따라서 Java 프로그램만 Kill할 경우 Redis 프로세스는 수동으로 Kill 해야한다. 그러니 Redis 정지도 함수 호출을 통해 정지한다.*

```java
# EmbeddedRedisServer.java
/**
 * <PRE>
 * Simple Embedded Redis Server
 * 주의1: 로컬 테스트 목적 이외에는 사용 금지 (프로젝트 서버 목적으로 사용할 수 없음)
 * 주의2: 반드시 명령어 입력을 통해서 프로그램을 종료해야 합니다.
 * </PRE>
 *
 * @author    윤순혁
 * @version   1.0
 */
public class EmbeddedRedisServer {

	private static final int REDIS_PORT = 6379;

	public static void main( String[] args ) {
		try (Jedis jedis = new Jedis( "127.0.0.1", REDIS_PORT ); Scanner input = new Scanner( System.in )){
			RedisServer redisServer = RedisServer.builder()
												.port( REDIS_PORT )
												.setting( "daemonize no" )
												.setting( "appendonly no" )
												.build();

			redisServer.start();

			System.out.println( "\nEmbedded Redis Server Started" );
			System.out.println( "==============================================================================" );
			System.out.println( "주의사항: 반드시 명령어 입력을 통해서 프로그램을 종료해야 합니다." );
			System.out.println( "          (그렇지 않으면 Redis 데몬을 '작업관리자'에서 프로세스 종료 시켜야 함)" );
			System.out.println( "==============================================================================" );
			System.out.println( "기능 (콘솔 창에 명령어 입력 후 Enter)" );
			System.out.println( "Redis 상태보기: status" );
			System.out.println( "Redis 종료: stop" );
			System.out.println( "==============================================================================" );
			String inputLine = "";
			jedis.connect();
			while ( !inputLine.equals( "stop" ) ) {
				System.out.print( "[기능 입력] # " );
				inputLine = input.nextLine();

				switch ( inputLine ) {
					case "stop":
						redisServer.stop();
						System.out.println( "\nEmbedded Redis Server Stopped" );
						break;
					case "status":
						System.out.println( jedis.info() );
						break;
					default:
						break;
				}
			}
			System.exit( 0 );
		} catch ( Exception redisError ) {
			System.err.println( "Embedded Redis Server error: " + redisError.getMessage() );
		}
	}
}
```