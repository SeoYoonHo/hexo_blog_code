---
title: AOP
toc: true
date: 2022-09-23 09:29:17
tags:
  - AOP
  - aspectJ
  - JDK Dynamic Proxy
  - CGLib
  - Spring
  - CS
categories:
  - [Spring, Spring AOP]
---
오늘은 지난번 정리못한 Spring AOP에 대한 정리를 해보려 한다. 그동안 스프링을 사용하며 당연하게 사용해왔떤 AOP에 대해 좀 더 깊은 지식과 배경에 대해 알아보려 한다. 물론 이러한 점들이 실제 Spring AOP를 사용하는데 있어서 크게 도움이 되지 않을수도 있지만, 알고 쓰는것과 모르고 쓰는것에는 큰 차이가 있다고 믿는다.

<!-- more -->

### **AOP란?**
Spring AOP를 공부하기에 앞서 AOP에 대한 개념을 짚고 넘어가야한다. AOP에 대한 개념을 Spring으로 처음 접했던 나는 일종의 스프링이 제공하는 기능중 하나로만 생각했었다. 하지만 AOP는 OOP와 마찬가지로 프로그램을 작성하는 하나의 패러다임이며, Spring은 그러한 AOP의 개념중 일부만을 사용하고 있다는 것을 알게되었다.

![aop](/post_images/SpringAOP/aop.png)

AOP란 *Aspect Oriented Programming*의 약자로 관점 지향프로그래밍이라 불린다. 여러분도 메서드를 작성할 때 핵심 로직 이외의 기능들을 중복적으로 개발한 경험이 있을것이다. 예를 들어 로깅처리, 성능을 위한 시간측정등의 핵심로직이라 할 수 없는 부가기능을 *cross-cutting concerns(횡단관심사, 흩어진 관심사)*로 보고 이를 모듈화 하는 것을 AOP라 한다.

즉 AOP를 통해 비본질적인 코드를 모듈화하여 개발자는 핵심로직에만 접근할 수 있도록 작성하는 일종의 프로그래밍 기법인 것이다. 이는 OOP의 단일책임원칙을 더욱 도와주는 개념으로 AOP는 프로그램을 더욱 객체지향적으로 작성할 수 있도록 도와주는 역할을 하게 된다.

### **AOP용어 정리**
본격적으로 AOP에 대해 알아보기전, 용어와 핵심 개념에 대해 알아보도로 하자.

  * Aspect : 애플리케이션 내 여러군데에 흩어져있는 코드/기능이이다. 즉 여러 객체에 공통적으로 적용되는 관심사항을 말한다.
  * Joinpoint : 프로르갬이 실행 중일 떄 발생하는 메서드 실행, 생서자 호출, 필드 값 변경과 같은 특수한 지점을 말한다.
  * Advice : 특정 Joinpoint의 Aspect에 의한 동작을 의미한다. 즉 Joinpint에 weaving되어 동작할 수 있는 코드를 의미한다.
  * Pointcut : Joinpoint의 정규표현식이다. 즉 Joinpint가 Pointcut에 일치할 때마다 Advice가 실행된다.
  * Weaving : Aspect를 대상 객체에 연결시켜 관점지향 객체로 만드는 과정을 말한다.

### **Spring Aop와 AsjpectJ**
앞서 말했듯 AOP는 Spring에 국한된 기능이 아니며 각 프로그램 언어들은 본인들만의 구현체로 AOP를 지원한다. 자바진영에서는 AspectJ를 통해 완전한 AOP솔루션을 제공하는것을 목표로 한다. 그렇다면 Spring AOP와 AspectJ에는 무슨 차이가 있을까? 간단히 말하자면 Spring AOP와 AspectJ는 그 목적이 다르다.

Spring AOP는 개발자가 마주한 공통적인 문제를 해결하고자 Spring IoC를 통한 간단한 AOP구현이 그 목적이다. 완전한 AOP를 의도한 것이 아니며 Spring에서 관리되는 Beans에만 적용가능하다.

반면, AspectJ는 앞서 언급했듯이 완전한 AOP를 지원하는것이 그 목적인 기술이다. Spring AOP보다 성능과 기능적인 측면에서 강력하지만, 사용법이 복잡하다는 단점이 있다.

### **Weaving**
Spring AOP와 AspectJ는 각각 다른 Weaving 방식을 사용한다.

AspectJ는 다음 세가지의 Weaving방식을 사용한다.

  - Compile Time Weaving(CTW) : 컴파일 시점 Waving으로 AsepctJ 컴파일러가 Aspect코드와 애플리케이션 소스를 모두 입력받아 Weaving된 class파일을 생성한다.
  - Post-Compile Weaving : 컴파일 후 Weaving으로 이미 존재하는 class파일과 jar파일을 Weaving하기 위해 사용된다.
  - Load Time Weaving(LTW) : 컴파일 후 Weaving과 유사하나 JVM에 로드될떄까지 연기된다는 점이 다르다.

반면 Spring AOP는 다음 Weaving 방식을 사용한다
  
  - Runtime Weaving(RTW) : Proxy 객체를 생성해 실제 타킷 오브젝트의 변형 없이 런타임 중 메서드 호출이 일어나는 시점에 Weaving을 수행한다. 메소드 호출에 대해서만 Advice를 적용할 수 있다는 단점이 있다.

### **Proxy Pattern이란?**
여태까지 AOP의 개념과 이를 구현하기 위한 AspectJ과 SpringAOP에 대해 알아보았다. 먼 길을 왔지만 아직 갈길이 멀다. 이제부터는 Spring AOP에 사용되는 Proxy에 대해 더 깊게 알아보고자 한다.

앞서 말했듯 Spring AOP는 런타인 시점 위빙방식을 사용하며 이를 위해 Proxy를 사용한다고 하였다. 그렇다면 Proxy란 무엇인가? Proxy는 Proxy Pattern에서 나온 개념이며 Proxy Pattern의 정의는 다음과 같다.

    프록시 패턴이란 소프트웨어 디자인 패턴중 하나로 오리지널 객체(Real Object)대신 프록시 객체(Proxy Obejct)를 사용해 로직의 흐름을 제어하는 디자인 패턴이다.
  
즉 개발자는 프로그램은 실제 구현객체에 접근하지 않고 프록시객체를 통해 간접실행하는 방식을 말합니다. 이 과정에서 프록시 객체에서 부가적인 기능을 수행하는 방식으로 AOP를 구현한다. 이러한 프록시패턴을 지원하기 위해 Spring은 JDK Dynamic Proxy나 CG Library를 사용한다. 이제부터 이 두 방식에 대해 알아보도록 하겠다.

### **JDK Dynamic Proxy**
JDK에서 제공하는 Dynamic Proxy는 Interface를 기반으로 Proxy를 생성해주는 방식이다. 즉 인터페이스의 존재가 필수적이다. 또한, Java의 리플렉션 기능을 활용하여 Proxy기능을 제공하여준다.

  - **Reflection**?
    Reflection이란 구체적인 클래스타입을 알지 못해도 그 클래스의 정보(메서드, 타입, 변수 등)에 접근할 수 있게 해주는 자바 API이다. 클래스 로더에 의해 로딩된 클래스 정보는 컴파일되어 Heap영역과 Meataspace에 저장된다. 리플렉션은 런타임시에 해당 영역에 접근하여 정보들을 알아내는 것이다.

위에서 설명한 Java의 Reflection API는 값비싼 API이기 떄문에 Dynamic Proxy는 성능이 떨어진다는 단점이 있다.

### **CGLib**
CGLib는 JDK Dynamic Proxy와는 다르게 인터페이스가 아닌 클래스 기반으로 바이트코드를 조작하여 프록시를 생성하는 방식이다. CGLib는 바이트코드 조작을 위해 ASM이라는 프레임워크를 사용하여 클래스를 동적으로 생성하거나 수정한다.

인터페이스가 아닌 클래스 대상으로 동작이 가능하고 바이트코드를 조작해 프록시를 만들기 떄문에 성능이 우수하다는 장점이 있지만, 상속을 이용하여 Proxy메서드를 오버라이딩 하는 방식인 만큼 final이나 private와 같이 객체에서 오버라이딩을 지원하지 않는 경우 적용할 수 없다.

하지만 성능상의 우수함 떄문에 Spring 4.3, Spring Boot 1.3이후부터는 CGLib가 Spring의 default방식으로 지정되었다.

### **끝으로**
오늘의 AOP정리의 처음은 직장동료와 CGLib에 대한 대화에서 시작하였다. 단순 Proxy패턴에서 AOP전반적인 내용까지 학습을 거슬러 올라가면서 하다보니 새로 알아가는 즐거움이 더욱 컸던거같다. 결국 개발자란 혼자 성장하는것도 중요하지만 대화와 토론을 통해서 배우는 부분이 정말 많다는 것을 다시한번 느끼게 되었다.


참조:
- https://velog.io/@suhongkim98/AOP
- https://velog.io/@suhongkim98/JDK-Dynamic-Proxy%EC%99%80-CGLib
- https://velog.io/@suhongkim98/Intro-To-AspectJ
- https://logical-code.tistory.com/118
- https://huisam.tistory.com/entry/springAOP