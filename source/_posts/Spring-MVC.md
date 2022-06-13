---
title: Spring MVC
toc: true
date: 2022-06-13 22:19:23
tags:
  - Spring
  - CS
  - MVC
  - MVC2
  - Spring MVC
categories:
  - [Spring, Spring MVC]
---
최근 회사에서 SI프로젝트 지원을 나가게 되었다. 해당 프로젝트에서 사용하는 프레임워크가 Spring MVC였다. 해당 패턴에 대하여 다시한번 리마인드도 해볼겸 대부분의 웹 개발에 있어서 표준처럼 쓰이는 MVC 디자인 패턴에 대해 알아고보, 나아가 Spring MVC의 처리 과정까지 알아보도록 하겠다.

### **MVC 패턴이란?**
<center><img src="/post_images/Spring MVC/mvc.png"></center><br>

<!-- more -->

MVC(Model-View-Controller) 패턴은 소프트웨어의 비즈니스 로직과 화면을 구분하는데 중점을 둔 디자인 패턴이다. 각 모델 뷰 컨트롤러는 다음과 같은 특징들을 가지고 있다.

#### Model(모델)

애플리케이션의 정보, 데이터의 가공을 책임지며 DB와 상호작용하며 비즈니스 로직을 처리하는 모듈, 컴포넌트를 말하며 아래와 같은 규칙을 가지고 있다.

- 사용자가 이용하려는 모든 데이터를 가지고 있어야 한다.
- View(뷰) 또는 Controller(컨트롤러)에 대한 어떤 정보도 알 수 없어야 한다
- 변경이 일어나면 처리 방법을 구현해야 한다.

#### View(뷰)

사용자 인터페이스 요소를 뜻하는데, Client에게 보여지는 결과화면을 반환하는 모듈을 말하며 아래와 같은 규칙을 가지고 있다.

- Model(모델)이 가지고 있는 데이터를 저장하면 안된다.
- Model(모델)이나 Controller(컨트롤러)에 대한 정보를 알면 안되며 닪순히 표시해주는 역할을 맡아야 한다.
- 변경이 일어나면 처리방법을 구현해야 한다.

#### Controller(컨트롤러)

Client 요청이 들어왔을 때 그 입력을 처리하고 어떤 로직을 실행시킬 것인지 Model(모델)과 View(뷰)를 연결해주며 제어하는 모듈을 말한다.

Controller는 아래와 같은 규칙들을 가지고 있다.

- Model(모델) 또는 View(뷰)에 대한 정보를 알아야 한다.
- Model(모델) 또는 View(뷰)의 변경을 인지하여 대처를 해야한다.


### **MVC 패턴 사용이유**

MVC 패턴의 사용목적이자 장점은 각 컴포넌트들이 서로 분리되어 자신의 역할에만 집중할 수 있게끔 개발을 하고 그렇게 만들어진 애플리케이션은 유지보수성, 확장성, 유연성등이 증가하고 중복코딩이라는 문제점이 사라지도록 한다는 것이다.

### **MVC1**
<center><img src="/post_images/Spring MVC/mvc1.jpeg"></center>

MVC1 패턴은 WAS(Web Application Server)내의 JSP에서 View와 Controller 역할을 모두 담당하는 패턴이다. 아키텍쳐가 간단하기때문에 작은 웹 어플리케이션을 제작할 때는 큰 무리가 없지만 대규모 웹 어플리케이션을 제작하는 경우 가독성이 떨어지고 유지보수에 어려움이 있다.

이를 보완하기 위해 MVC2 패턴이 등장하게 되었다.


### **MVC2**
<center><img src="/post_images/Spring MVC/mvc2.jpg"></center>

MVC의 단점을 보완하기 위해 View와 Controller가 분리되었고 JSP는 로직 처리가 없는 단순히 Client에게 보여지는 View만을 담당하게 되었다.

현재의 모든 웹어플리케이션은 MVC2 패턴을 따르고 있고 우리가 알고 있는 Spring MVC 또한 MVC2 패턴을 따르고 있다.

### **Spring MVC**
<center><img src="/post_images/Spring MVC/SpringMVC.png"></center>

Spring 진영에서는 MVC2 패턴을 조금 더 발전시켜 Spring MVC 패턴을 선보였다.

위의 아키텍처에서 각 컴포넌트의 구조와 역할을 간단히 살펴보면, 프론트 컨트롤러가 우선적으로 클라이언트로부터 모든 요청을 받게 되며, 실제 요청의 처리는 개별 컨트롤러 클래스로 위임한다.

개별 컨트롤러 클래스는 핸들러라고도 하며, DI를 통해 생성해둔 Bean을 이용하여 비즈니스로직 처리 결과를 Model에 담아 다시 프론트 컨트롤러로 보낸다. 프론트 컨트롤러는 받은 Model을 알맞은 View 템플릿으로 전달하여 반영시키고, 최종적으로 클라이언트로 보낼 화면을 응답 결과로 전송한다.

### **Spring MVC 실제 동작구조**
<center><img src="/post_images/Spring MVC/SpringMVC_Real.png"></center>

위의 아키텍처는 Spring MVC의 실제 구조를 도식화 한것이다. 각각의 컴포넌트의 역할에 대해서 살펴보자


#### Dispatcher Servlet
- 사용자의 모든 요청을 받아서 처리하는 프론트 컨트롤러이다.
- 모든 Request를 각각의 컨트롤러에게 위임한다.
- Dispathcer Servlet을 프론트 컨트롤러가 가능하도록 설정하기 위해서는 web.xml에 명시하거나, org.springframework.web.WebApplicationInitializer 인터페이스를 구현하는 두가지 방식을 사용할 수 있다.

#### Handler Mapping
- 요청을 직접 처리할 컨트롤러를 탐색한다.
- 구체적인 mapping은 xm파일이나 java config 관련 어노테이션 등을 통해 처리할 수 있다.

#### Handler Adapter
- 매핑된 컨트롤러의 실행을 요청한다.

#### Controller
- 직접 요청을 처리하며, 처리 결과를 반환한다.
- 컨트롤러에서 결과가 반환되면 Handler Adapter가 ModelAndView 객체로 변환되며, 여기에는 View Name과 같이 응답을 통해 보여줄 View에 대한 정보와 관련된 데이터가 포함되어 있다.

#### View Resolver
- View Name을 확인한 후, 실제 컨트롤러부터 받은 로직 처리결과를 반영할 View파일(jsp)를 탐색한다.

#### View
- 로직 처리 결과를 반영한 최종 화면을 생성한다.

### **Spring MVC 동작 순서**

1. 클라이언트가 서버에 요청을 하면, 프론트 컨트롤러인 DispatcherServlet 클래스가 요청을 받는다.
2. DispatcherServlet은 프로젝트 파일 내의 servlet-context.xml 파일의 @Controller 인자를 통해 등록한 요청 위임 컨트롤러를 찾아 매핑(mapping)된 컨트롤러가 존재하면 @RequestMapping을 통해 요청을 처리할 메소드로 이동한다.
3. 컨트롤러는 해당 요청을 처리할 Service(서비스)를 받아 비즈니스 로직을 서비스에게 위임한다.
4. Service(서비스)는 요청에 필요한 작업을 수행하고, 요청에 대해 DB에 접근해야 한다면 DAO에 요청하여 처리를 위임한다.
5. DAO는 DB정보를 DTO를 통해 받아 서비스에 전달한다.
6. 서비스는 전달받은 데이터를 컨트롤러에 전달한다.
7. 컨트롤러는 Model(모델) 객체에게 요청에 맞는 View(뷰) 정보를 담아 DispatcherServlet에 전송한다.
8. DispatcherServlet은 ViewResolver에게 전달받은 View 정보를 전달한다.
9. ViewResolver는 응답할 View에 대한 JSP를 찾아 DispatcherServlet에 전달한다.
10. DispatcherServlet은 응답할 뷰의 Render를 지시하고 뷰는 로직을 처리한다.
11. DispatcherServlet은 클라이언트에게 Rending된 뷰를 응답하며 요청을 마친다.

출처:
- https://ss-o.tistory.com/160
- https://velog.io/@gillog/a-j5c0h49n