---
title: 모니터링 시스템 장애(2) - node-oraclelib 버전 호환성
toc: true
date: 2022-06-03 11:44:32
tags:  
  - 모니터링 시스템
  - Prometheus
  - Grafana
  - STIS
  - Expoter
  - AWS lambda
  - nodejs
categories:
  - [모니터링, Prometheus/Grafana]
---
지난 포스팅에서 언급한것처럼 오늘은 nodejs 버전 업그레이드 과정에서 생긴 모니터링 시스템 장애에 대해서 말해보고자 한다.

<!-- more -->

### **Serverless Framework nodejs 버전 업그레이드**
Serverless Framework에서 nodejs 버전업그레이드는 정말 간편하다. serverless.yml 파일의 provider 의 runtime만 nodejs14.x로 변경해주면 된다. 이렇게만 설정해주면 해당 yml을 참고하는 모든 lambda의 nodejs 버전이 업그레이드 된다. 이 간편함때문에 내가 serverless framework를 즐겨쓴다.

이번 nodejs 버전 업그레이드는 위에서 설명한것처럼 간단하게 진행되었다. 버전 명시후 재배포를 진행하였고 문제없이 배포되었다. 하지만 약 1분뒤 lambda를 사용하는 모든 모니터링에서 알람이 울리기 시작했다. 바로 망했다는 생각이 들었고 수많은 lambda 중 하나를 골라 CloudWatch상의 로그를 보기 시작하였다.

<center><img src="/post_images/monitoring/oraclelib_error.png"></center>

### **라이브러리 버전 호환성**
위와 같은 로그가 보이기 시작하였다. 바뀐게 없는데 왜 저런 오류가 날까 한참을 고민하다 사용하던 oracle lib 버전이 현재 nodejs와 호환이 되지 않을수 있다는 생각이 들었다.

<center><img src="/post_images/monitoring/oraclelib_version.png"></center>

예상대로 현재 사용되는 버전은 4년전에 사용되던 버전이었고 최신버전으로 업그레이드 이후 다시 재배포하였다. 이후 모든 수치들은 정상으로 돌아오게 되었다.

### **마치며**
다행히 모든 모니터링 수치는 정상으로 돌아왔지만, 문제가 생긴 1시간정도 정말 식은땀을 흘렸던거 같다. 이번 경험을 통해 느낀바는 두가지이다.

1. AWS의 기술 지원 기간을 잘 확인해야 한다는 점
2. 버전 업그레이드시에는 각 라이브러리와의 호환성 체크가 필수라는 점

이렇게 또 한번의 경험을 통해 성장하는 계기가 되었다.