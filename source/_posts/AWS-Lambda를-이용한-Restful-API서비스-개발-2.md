---
title: AWS Lambda를 이용한 Restful API서비스 개발(2)
toc: true
date: 2022-07-12 16:06:33
tags:
  - AWS
  - Serverless
  - API Gateway
  - Lambda
  - Restful api
categories:
  - [AWS, Lambda]
---
지난 포스팅서는 Lambda의 개념과 과금체계에 대해서 알아보았다. 오늘은 이 람다를 이용한 Restful API 개발방법에 대해 알아보도록 하겠다.

<!-- more -->

### **AWS ApiGateway**
람다는 서버가 필요없고 자동으로 Scale Up, Scale Down된다는 점에서 Rest API를 개발하는데 굉장히 유리하다. 하지만 람다는 그 자체만으로는 동작할 수 없기에 실행을 촉발히켜주는 트리거가 필요하다. API Gateway는 URL 요청을 받아 람다를 실행시켜주는 AWS의 서비스이다. 이 서비스를 이용하면 추가적인 장점들이 따라오는데 우선 https통신방식을 기본으로 설정해주기 때문에 SSL에 대해 고민할 필요가 없다. 또한 권한부여자 기능을 통해 api인증기능을 아름답고 쉽게 구현할 수 있도록 지원하고 있다.

최종적으로 람다를 이용한 REST API아키텍쳐는 다음과 같을것이다.
<center><img src="/post_images/lambda/lambda_archi.png"></center>

### **Serverless Framework**
위와 같은 구조로 개발을 진행하기 위해서는 각 서비스들을 생성하고 설정해주는 작업들이 필요하다. 우선 콘솔상에서 직접 각 서비스들을 만들기 위해서는 다음과 같은 과정이 필요하다.

1. AWS 콘솔 로그인
2. Lambda 생성
3. 트리거 API Gateway 지정
4. API Gateway 생성
5. 테스트

이와 같은 과정이 새로운 API가 추가될때마다 N번 반복되게 된다. 위와 같은 방식은 테스트가 용이하지도 않으면서, 서비스가 커지게 된다면 너무나 비효율적이라 할 수 있다. 이 문제를 해결하기 위해 Lambda개발을 지원하는 다양한 프레임워크가 있다. 대표적으로 AWS에서 제공하는 SAM과 Serverless Framework가 존재한다. 이번 프로젝트에서는 기존에 사용했던 Serverless Framework를 사용하기로 결정하였다.

Serverless Framework 공식페이지에는 Lambda와 API Gateway를 이용한 Rest API 구현 예제를 제공하고 이 예제를 통해 초기 셋업을 진행하면 된다.
(https://www.serverless.com/examples)

### **Serverless Offline**
Lambda 개발을 진행하다보면 매번 테스트를 배포해서 해야하는 번거로움이 있다. Serverless Framwork진영에서는 이 문제를 해결하기 우해 Serverless Offline이라는 플러그인을 제공하고, 이를 통해 로컬에서 API테스트를 진행 할 수 있다.
(https://www.npmjs.com/package/serverless-offline)

### **마치며**
오늘은 API Gateway와 Serverless Framework에 대한 내용을 정리하였다. Lambda의 런타임을 선택할 떄는 보통 파이썬, Go, Node.Js를 선택하는데 이는 Lambda의 Cold Start시간과 연관이 있다. 다음 포스팅에서는 Lambda의 Cold Start시간에 대해서 알아보도록 하겠다.