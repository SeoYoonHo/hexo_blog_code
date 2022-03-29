---
title: 모니터링 시스템(Prometheus, Grafana)(2)
toc: true
date: 2022-03-29 10:06:03
tags:
  - 모니터링 시스템
  - Prometheus
  - Grafana
  - STIS
  - Expoter
categories:
  - [모니터링, Prometheus/Grafana]
---
프로메테우스와 그라파나의 전체적인 아키텍처 및 설치과정을 알아보도록 하자

### **프로메테우스/그라파나 아키텍쳐**
지난 게시물에서도 말했듯이 프로메테우스는 데이터소스, 그라파나는 시각화 툴의 역할을 각각 맡고 있다. 또한 프로메테우스는 서버 뿐만 아니라 다양한 노드(컴포넌트)가 수집한 데이터를 pull방식으로 가져오는 구조이다. 여기서 node는 프로메테우스에서 제공하는 서비스일 수도 있고 사용자가 직접 커스텀한 Exporter일수도 있다.
그 구조는 아래와 같다.

![프로메테우스 아키텍처](/post_images/prometheus/prom3.png)

<!-- more -->

프로메테우스를 구성하는 컴포넌트들은 다음과 같다
  
  - **Prometheus Server**
    데이터를 수집하고 저장하는 메인서버
  - **Exporter**
    target metric 데이터 수집
    Http 엔드포인트를 설정하여 서버에서 메트릭을 수집하도록 지원
    HAProxy, StatsD, Graphite와 같은 서비스를 지원
  - **AlertManager**
    설정한 규칙에 따른 알림 처리(Grafana 자체적으로 알람시스템을 가지고 있어 사용하지 않았음)
  - **PushGateway**
    Exporter를 이용하여 수집이 어려운 작업에 대한 매트릭 pushing 지원
  - **Grafana**
    시각화 툴
  - **Client Libraries**
    Java, Scala, Go, Python, Ruby 등의 언어에 대한 프로메테우 연동 라이브러리

현재 운영중인 모니터링 시스템의 경우 대부분은 Exporter로 데이터를 가져오고 있고, 알람은 Grafana의 알람체계를 사용하고 있다. **결국 Exporter를 각 시스템에 맞게 잘 구현하는것이 모니터링 시스템 구축의 핵심**이라 할 수 있다.

### **프로메테우스 설치**

#### **압축파일 다운로드**
현재 우리 서버에 설치된 버전은 2.19.0 버전이다

{% blockquote %}
  $ wget https://github.com/prometheus/prometheus/releases/download/v2.19.0/prometheus-2.19.0.linux-amd64.tar.gz

  $ tar xvfz prometheus-2.19.0.linux-amd64.tar.gz
  $ cd prometheus-2.19.0.linux-amd64
{% endblockquote %}

파일을 받은 후 압축까지 해제하여줍니다. 압축 해제후 설치과정은 따로 필요없습니다. 바로 prometheus 바이너리 파일이 실행 프로그램입니다.

#### **설정파일**
설치된 폴더안에 prometheus.yml라는 YAML파일이 있습니다.

{% blockquote %}
  \# my global config
  global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  \# scrape_timeout is set to the global default (10s).

  \# Alertmanager configuration
  alerting:
  alertmanagers:
  - static_configs:
  - targets:
  \# - alertmanager:9093

  \# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
  rule_files:
  \# - "first_rules.yml"
  \# - "second_rules.yml"

  \# A scrape configuration containing exactly one endpoint to scrape:
  \# Here it's Prometheus itself.
  scrape_configs:
  \# The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

  \# metrics_path defaults to '/metrics'
  \# scheme defaults to 'http'.

  static_configs:
  - targets: ['localhost:9090']
{% endblockquote %}

- **global**
  Prometheus의 전반적인 부분에 대한 설정
  - scrap_interval : 정보를 수집하는 주기를 설정
  - evalutaion_interval : 시계열을 만들고 알람을 발생시키는 주기를 설정

- **rule_files**
  Prometheus 에서 불러올 rules 파일에 대한 경로를 지정하는 설정

- **scrape_configs**
  Prometheus에서 어떤 자원을 모니터링할지 설정
  기본값으로 'prometheus'이 지정되어있는데 Prometheus 프로그램에서 발생된 시계열데이터를 수집
  결국 이 설정값에 exporter를 연결시켜 데이터를 수집하므로 주된 설정값임

#### **프로메테우스 실행**
실행시에는 설정파일을 지정해줘야 한다.

{% blockquote %}
  $ ./prometheus --config.file=prometheus.yml
{% endblockquote %}

실행을 확인하귀 위해 브라우저를 열어 **http://localhost:9090** 으로 접속하면 프로메테우스 UI 화면이 나타난다.

![프로메테우스 UI](/post_images/prometheus/prom4.png)

또한 프로메테우스가 어떤 메트릭 정보들을 제공하는지 확인하고 싶다면 **http://localhost:9090/metrics** 경로에서 확인할 수 있다.

![프로메테우스 Metric](/post_images/prometheus/prom5.png)

위의 정보들은 실제로 운영중인 시스템의 모니터링 metric 값들이다. 해당 값들을 프로메테우스에 제공하기 위한 node exporter에 관한 이야기를 해보자

### **Custom Node Exporter**
Prometheus에서 제공되는 오픈소스 프로그램중 Node Exporter라는 것이 있다. Node Exporter의 종류는 굉장히 다양하고 Oracle, Mysql 등 DB에서 쿼리만 제공하면 데이터를 추출해주는 Exporter도 존재한다. 관련 Exporter는 공식 프로메테우스 공식 홈페이지를 통해 링크가 제공되니 참고해보기 바란다.

![Node Exporter](/post_images/prometheus/prom6.png)

하지만 우리 시스템에서는 좀 더 커스텀된 데이터 가공이 필요하였고 결국 커스텀 Exporter를 만들기로 결정하였다. 어짜피 방식은 HTTP(REST) 방식이면 되니 개발은 간단하였다. 우리가 선택한 기술은 aws lambda 였고 이를 위해 serverless framework를 사용하여 exporter를 개발하였다.

Serverless Framework과 lambda에 대해서는 추후에 더 자세하게 다뤄보도록 하겠다.
api 구현방식이 중요한것이 아니라 프로메테우스가 원하는 metric 포맷으로 데이터를 리턴해주는 api가 존재하기만 하면 된다. api를 개발하는 방식은 굉장히 여러가지이니 개발자 재량껏 편한 방식으로 개발하면 된다.

Lambda 언어는 node js를 선택하였고 프로메테우스 metric 데이터 형식으로 변환하기위해 'prom-client' 를 패키지 관리자를 통해 설치하여 간편하게 metric 데이터 포맷으로 변환하였다.

해당 소스는 다음과 같다

![Custom Exporter](/post_images/prometheus/prom7.png)

소스 일부분만 가져왔지만 내용을 보면 sql라는 설정파일에 쿼리를 작성하면 원하는 내용으로 metric 데이터로 변환하여 준다. 위으 소스에서는 데이터를 Guage 형태로 가져왔는데 프로메테우스의 Metric 데이터 타입은 4가지로 나눠진다.

1. Counter
1. Guague
1. Histogram
1. Summary

해당 타입별 정의와 쓰임새는 공식홈페이지를 참고하기 바란다.(https://prometheus.io/docs/instrumenting/exporters/)

어쨌든 이런식으로 개발된 api의 엔드포인트를 prometheus.yml에 설정하여주면 위에서 보여준것과 같이 metric 데이터가 수집된다. 이제는 해당 데이터를 Grafana를 통해 시각화 해주면 된다.
이에 대한 내용은 다음 포스트에서 다뤄보기로 한다.
