---
layout: post
title:  "Spring RestTemplate의 사설인증서 연계를 위한 SSLConnectionSocketFactory 생성"
date:  2016-09-08
categories:
- Spring
tags:
- Spring RestTemplate
- Apache HttpComponents
published: true
---
### Spring RestTemplate를 이용하여 외부 연계 개발시 활용 가능한 모듈 정리

* 비용의 문제겠지만 개발 및 테스트 웹 시스템 환경의 TLS/SSL 인증서는 사설 인증서 또는 원래 호스트와 다른 도메인의 인증서가 적용되어 있는 경우가 많다.

* 위와 같은 환경에 대해 RestTemplate(org.springframework.web.client.RestTemplate - Apache HttpComponents 기반)를 활용하여 HTTPS 연계를 수행할 경우 기본적으로 수행하는 인증서 검증(호스트명과 인증서 대조) 기능 때문에 오류를 발생하기도 한다.

* 아래 클래스의 명세는 RestTemplate의 기반인 Apache HttpComponents의 'SSLConnectionSocketFactory' 클래스를 커스텀 하여 구현된 Spring Factory Bean 클래스이다.

* 이 클래스를 RestTemplate에 적용하고 property 설정을 통해 사설 인증서 및 잘못된 Host 정보의 인증서에 대해서도 허용할지 여부를 결정할 수 있다.

* 검증 버전: Apache HttpComponents 4.5.x

#### 클래스

```java

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLContext;

import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.TrustSelfSignedStrategy;
import org.apache.http.ssl.SSLContexts;
import org.springframework.beans.factory.FactoryBean;

/**
 * <PRE>
 * SSLConnectionSocketFactory 생성 클래스
 * - HTTPS VERIFIER 수정: NoopHostnameVerifier
 * - TRUST CA 전략 수정 : TrustSelfSignedStrategy
 * SSLConnectionSocketFactory.getSocketFactory() 커스텀을 위해 FactoryBean 형태로 구현
 * </PRE>
 *
 * @author    윤순혁
 * @version   1.0
 * @see       SSLConnectionSocketFactory
 */
public class CustomSSLConnectionSocketFactory implements FactoryBean<SSLConnectionSocketFactory> {

	private boolean allowAllHostname;

	private boolean allowSelfSignedCa;

	/**
	 * @param allowAllHostname the allowAllHostname to set
	 */
	public void setAllowAllHostname( boolean allowAllHostname ) {
		this.allowAllHostname = allowAllHostname;
	}

	/**
	 * @param allowSelfSignedCa the allowSelfSignedCa to set
	 */
	public void setAllowSelfSignedCa( boolean allowSelfSignedCa ) {
		this.allowSelfSignedCa = allowSelfSignedCa;
	}

	/* (non-Javadoc)
	 * @see org.springframework.beans.factory.FactoryBean#getObject()
	 */
	@Override
	public SSLConnectionSocketFactory getObject() throws Exception {
		/*
		 * SSLConnectionSocketFactory.getSocketFactory() 커스텀
		 *  - HTTPS VERIFIER 수정: NoopHostnameVerifier
		 *  - TRUST CA 전략 수정 : TrustSelfSignedStrategy
		 */
		// HTTPS 요청시 인증서 오류(개발/검증 서버의 사설 인증서 적용 상황)를 막기위한 SSLSocket Context 생성 과정
		HostnameVerifier hostnameVerifier = this.allowAllHostname ? NoopHostnameVerifier.INSTANCE : SSLConnectionSocketFactory.getDefaultHostnameVerifier();
		SSLContext sslContext = allowSelfSignedCa ? SSLContexts.custom().loadTrustMaterial( new TrustSelfSignedStrategy() ).build() : SSLContexts.createDefault();
		SSLConnectionSocketFactory sslSocketFactory = new SSLConnectionSocketFactory( sslContext, hostnameVerifier );

		return sslSocketFactory;
	}

	/* (non-Javadoc)
	 * @see org.springframework.beans.factory.FactoryBean#getObjectType()
	 */
	@Override
	public Class<?> getObjectType() {
		return SSLConnectionSocketFactory.class;
	}

	/* (non-Javadoc)
	 * @see org.springframework.beans.factory.FactoryBean#isSingleton()
	 */
	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

#### 설정 (XML)

* 아래는 RestTemplate을 선언한 Spring Context XML 파일이며 'PoolingHttpClientConnectionManager' Bean 내부의 HTTPS Socket 생성 설정에 위 클래스를 적용한다.
   - allowAllHostname: HTTPS 연동시 TLS/SSL 인증서의 모든 Hostname 허용 여부 (true/false)
   - allowSelfSignedCa: HTTPS 연동시 TLS/SSL 사설인증서 허용 여부 (true/false)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

<!--====================================================================================================
= 외부 시스템과 REST 연동을 위한 REST Template 설정 (Apache HttpComponents Wrapper)
=====================================================================================================-->

<!-- RestTemplate -->
<bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
	<property name="requestFactory">
		<bean id="httpComponentClientFactory" class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory">
			<constructor-arg><bean id="httpClient" factory-bean="httpClientBuilder" factory-method="build" /></constructor-arg>
		</bean>
	</property>
</bean>

<!-- HTTP Client Builder -->
<bean id="httpClientBuilder" class="org.apache.http.impl.client.HttpClientBuilder" factory-method="create">
	<property name="connectionManagerShared" value="true" />
	<property name="connectionManager">
		<!-- Connection Pool Manager : PoolingHttpClientConnectionManager 클래스 + 별도 SSL Socket Factory 적용 -->
		<bean id="connManager" class="org.apache.http.impl.conn.PoolingHttpClientConnectionManager">
			<constructor-arg name="socketFactoryRegistry">
				<bean class="org.apache.http.config.Registry">
					<constructor-arg>
						<map>
							<entry key="http">
								<bean class="org.apache.http.conn.socket.PlainConnectionSocketFactory" factory-method="getSocketFactory" />
							</entry>
							<entry key="https">
								<bean class="package.CustomSSLConnectionSocketFactory">
									<!-- HTTPS 연동시 TLS/SSL 인증서의 모든 Hostname 허용 여부 -->
									<property name="allowAllHostname" value="true" />
									<!-- HTTPS 연동시 TLS/SSL 사설인증서 허용 여부 -->
									<property name="allowSelfSignedCa" value="true" />
								</bean>
							</entry>
						</map>
					</constructor-arg>
				</bean>
			</constructor-arg>
			<!-- 각 host(IP와 Port의 조합)당 Connection Pool에 생성가능한 Connection 수 -->
			<property name="defaultMaxPerRoute" value="10" />
			<!-- Connection Pool의 수용 가능한 최대 사이즈 -->
			<property name="maxTotal" value="50" />
			<!-- 특정 시간 이후 유휴 커넥션 정리 (ms) -->
			<property name="validateAfterInactivity" value="60000" />
		</bean>
	</property>
</bean>
</beans>
```