---
title: Spring Framework란?
tags:
  - Sping
  - CS
categories:
  - - Spring
    - Spring Framework
toc: true
date: 2022-04-19 11:28:14
---


### **Spring의 역사**
Spring Framework이 등장하기 전에는 EJB라는 기술을 통해 웹 애플리케이션을 개발하였다. 이 기술은 여러가지 복잡성으로 인해 사용하기 까다로웠고, 이러한 단점들을 보안하기 위한 기술들이 연구되었다. 그 과정에서 가장 호평을 받은 기술이 Spring Framework이다.

물론 Spring Framework가 처음부터 바로 나온것은 아니다. 2002년 로드 존슨은 자신이 출판한 저서(Expert One-on-One J2EE Design and Development)에 스프링의 핵심 개념과 기반 코드들을 소개하였다.

책 출간 직후 유겐 휠러와 얀 카로프가 로드 존슨에게 오픈 소스 프로젝트를 제안하였고, **전통적인 EJB라는 겨울을 넘어 새로운 시작이라는 뜻으로 Spring이라고 명칭을 짓게 되었다.**

### **Framework란?**
Spring은 항상 Fraemwork와 결합되어 **Spring Framework**라 불린다. 여기에서 사용된 Framework란 무엇일까?? Framework의 개념에 대해 설명할 떄 항상 Library와 비교되곤 한다. 두 개념이 유사한 점이 많아서일 것이다. 여기선 두 개념에 대한 공통점과 차이점에 대해 알아보고자 한다.

Framework는 뼈대나 기반 구조를 뜻하고, 제어의 역전(IoC) 개념이 적용된 대표적인 기술이다. 소프트웨어에서의 프레임워크는 **소프트웨어의 특정 문제를 해결하기 위해서 상호 협력하는 클래스와 인터페이스의 집합**이라 할 수 있다. 
Library는 개발에 필요한 것들을 미리 구현해놓은 대상, 도구들의 집합을 말한다. 즉, 개발자가 만든 클래스에서 호출하여 사용하는 방식을 취하고 있다.

Framework와 Library 모두 애플리케이션을 개발하는데 있어 쉽고 빠른 생산성을 위해 사용한다는 공통점을 가지고 있다. 이 둘의 가장 큰 차이는 바로 **제어역전**이다

<center><img src="/post_images/Spring/framework.jpeg"></center>

관건은 애플리케이션의 흐름을 누가 쥐고 있느냐이다. Framework는 전체적인 흐름을 스스로 쥐고 있으면서 사용자는 그 안에서 필요한 코드를 짜 넣지만, Library는 사용자가 전체적인 흐름을 만들며 라이브러리를 가져다 쓴다.