---
title: 모니터링 시스템 장애(1) - AWS Lambda 런타임 지원기한
toc: true
date: 2022-06-03 09:55:50
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
### **모니터링 시스템 요약**
현재 우리 시스템은 Prometheus와 Grafana를 통하여 모니터링 시스템이 구축되어 있다. Prometheus는 다양한 데이터를 직접 pooling하여 데이터를 저장하고, Grafana는 이 데이터를 시각화한다. 이 Prometheus가 pooling을 하는 방식으로 rest api를 사용하였고, 이 api를 제공하는 프로그램을 Exporter라고 한다. Promethues에서 제공하는 다양한 Exporter가 있지만 우리 시스템에서는 AWS Lambda를 Serverless Framework를 이용하여 이 Exporter를 커스텀하게 구축하였다.

<!-- more -->

### **Exporter 배포 실패**
얼마 전 새로운 업무가 추가되고 모니터링 항목을 추가해야하는 작업이 진행되었다. 단순히 쿼리만 추가하여 Lambda를 재배포하면 되는 단순한 일이었다. 평소처럼 배포를 진행하던 중 cloud formation에서 해당 Lambda의 업데이트가 실패하였고 롤백을 시도하였으나 그마저도 실패하였다. 다시 재배포하기 위해서 *sls deploy -v* 를 계속해서 시도하였으나 실패 로그만 출력될 뿐이었다.

급하게 AWS 콘솔로 접근하여 cloud formation의 해당 스택의 작업을 수동으로 실행하고자 하였으나, 권한이 없어 시간이 지연되었다. 결국 문서에서 추천해주는 방법대로 aws cli를 실행하여 강제 롤백을 시도하였다.(*https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudformation-update-rollback-failed/*)

하지만 역시 해당 lambda 리소스에서 실패가 반복되었고, 명령어를 변경하여 리소스를 건너뛰고 시도하여 롤백에 성공하였다.

### **AWS Lambda 런타임 지원 기한**
급한 불을 껐으니 이제 원인을 파악해야 할 때였다. Cloudformation 로그를 보기 시작하였고 해당 로그를 발견하였다.

<center><img src="/post_images/monitoring/runtime.png"></center>


결국 AWS Lambda의 nodejs 런타임 지원기한을 간과하고 있었던 것이다. 해당 지원 기간 종료는 공식 문서를 참고하면 된다.(*https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-runtimes.html*)


<center><img src="/post_images/monitoring/lambda_runtime.png"></center>
해당 기한을 참고하면 사용하던 nodejs10.x는 2022년 2월 14일 부로 지원이 완전종료되었다.

문제가 파악되었으니 이제 해결만 하면 되었다. 하지만 이를 해결하는 과정에서도 문제가 발생하였는데 이는 다음 포스팅에서 다뤄보도록 하겠다.