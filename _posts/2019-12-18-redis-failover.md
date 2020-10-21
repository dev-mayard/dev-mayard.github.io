---
layout: post
title: 단일 서버에서 Redis 이중화 구성 하기
subtitle: Redis Sentinel과 HAProxy를 사용해 단일 서버에서 Reids 이중화 환경 구성하기
tags: [redis]
comments: true
---

Spring Boot로 되어 있는 웹 서비스에서 <a href="https://spring.io/projects/spring-session-data-redis" target="_blank">세션 스토리지</a>용으로 사용중인 redis의 이중화 구성을 위해,
단일 서버로 되어 있는 개발 환경에서 이중화 테스트를 해봤습니다.

***
### #개발환경
**Linux: CentOS 6**

**Redis: v5.0.4**
***

이중화 환경을 구성하기 위해 총 3개 (master, slave, sentinel) 의 redis 프로세스가 실행 중인 상태로 준비되어야 합니다.
저는 아래와 같이 port를 구분해 master, slave, sentinel 을 실행했습니다.

1. master: **6379**
2. slave: **20000**
3. sentinel: **26379**


위와 같은 환경에서, master 프로세스가 종료될 경우, senitinel은 이를 감지하여, slave를 master로 promote시키게 됩니다.

하지만 애플리케이션에서는 master port와 연결이 되어 있으므로 애플리케이션에서 에러가 발생하게 됩니다.
이를 위해 master 와 slave간에 로드 밸런싱을 해줄 <a href="http://www.haproxy.org/" target="_blank">HAProxy</a>가 필요하게 됩니다.

HAProxy 설치 후, master와 slave 두개의 포트를 연결하도록 설정을 해줍니다.

브라우져로 HAProxy의 Web console에 접속하여 master와 slave의 현재 상태에 대해 모니터링이 가능합니다.

![haproxy_normal](/assets/img/20191218/haproxy_normal.jpg)


--- 작 성 중 ---