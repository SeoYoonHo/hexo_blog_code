---
title: 모니터링 시스템(Prometheus, Grafana)(3)
toc: true
date: 2022-03-29 16:12:06
tags:
  - 모니터링 시스템
  - Prometheus
  - Grafana
  - STIS
  - Expoter
categories:
  - [모니터링, Prometheus/Grafana]
---
앞서 말했듯이 Grafana는 시계열 데이터를 시각화하는데 가장 최적화된 대시보드를 제공해주는 오픈소스 툴킷이다. 주로 InfluxDB, Prometheus, Graphite등의 시계열 데이터베이스와 함께 사용되며 실시간 데이터분석, 모니터링등에서 많이 사용되고 있다. 특히 자체적인 Alert기능 제공은 해당 툴을 선택하는데 가장 큰 이유가 되었다.

### **Grafana 설치**
Grafana를 설치과정은 공식사이트를 통해 자세하게 나와있다.(https://grafana.com/grafana/download)

![Grafana 설치](/post_images/prometheus/grafana1.png)

<!-- more -->

공식문서에서 제공하는것처럼 설치과정을 따르고 나면 압축 해제된 폴더의 bin 디렉토리로 이동하여 grafana 서버를 실행시킬 수 있다.

{% blockquote %}
  $ cd [압축 파일을 해제한 폴더]/bin
  $ ./grafana-server or grafana-server start
{% endblockquote %}

운영체제가 Redhat 계열이라면 아래 서비스 실행 명령어로 바로 실행시킬 수 있다.

{% blockquote %}
  $ sudo service grafana-server start
{% endblockquote %}

실행 후 3000번 포트로 접근하면 아래와 같은 대시보드 화면이 나오게 된다.

![Grafana 대시보드](/post_images/prometheus/grafana2.png)

Grafana 웹 서버의 초기 username/password 는 admin/admin 이다. 로그인이 완료되면 비밀번호를 변경하를 페이지가 뜨는데 원치 않는다면 skip을 누르면 된다.

### **대시보드 구성**
그라파나 설치 후 모니터링 대시보드를 구성하기 위한 방법을 여러가지이지만, 꽤나 번거로운 작업이기에 외부 대시보드 포맷을 사용한다면 좀 더 쉽게 구성할 수 있다.

아래 사이트에서 각종 대시보드 포맷을 둘러보고 선택해서 사용하면 된다
https://grafana.com/grafana/dashboards/

위와같은 대시보드 포맷을 사용하지 않고 각자 데이터 소스를 선택하고, 데이터를 편집하여 고유 화면을 구성할 수도 있다.

현재 우리 시스템같은 경우는 Prometheus와 CloudWatch를 데이터소스로 사용하여 대시보드 화면을 구성하고 있다.

![Grafana 구성도](/post_images/prometheus/grafana3.png)

위와 같은 아키텍쳐를 기반으로 설계된 모니터링 대시보드는 다음과 같다

![Grafana 대시보드](/post_images/prometheus/grafana4.png)

각자의 시스템에 맞는 대시보드를 꾸며 최적의 모니터링 화면을 설계해보도록 하자!