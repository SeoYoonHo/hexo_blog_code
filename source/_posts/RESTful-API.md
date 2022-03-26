---
title: RESTful API
date: 2022-03-26 01:12:53
tags:
  - CS
  - REful API
  - REST
categories:
  - CS
  - RESTful API
toc: true
---

![restful_api](/post_images/restful_api.png)

<!-- more -->

현대 애플리케이션은 대부분 프론트엔드라 부르는 클라이언트와 백엔드라 부르는 서버가 상호간 데이터를 주고받는 형태의 아키텍처를 가지고 있다.

이러한 구조의 가장 큰 특징중 하나는 서버와 클라이언트가 독립적으로 진화할 수 있다는 것이다.
독립적으로 진화한다는 뜻은 각각의 업데이트 내용이 서로에게 영향을 미치지 않는 다는 것이다.

대표적인 예는 웹(web)이다. 웹 페이지의 변경이 브라우저에 영향을 미치지 못하고, 브라우저의 변경또한 페이지에 영향을 미치지 못한다. 서버와 클라이언트가 서로에게 영향을 끼치지 않고 독립적으로 진화하기 때문이다.

이러한 클라이언트 서버간의 독립적 진화를 위한 분산시스템 아키텍처로 가장 널리 쓰이는게 REST _(Representatioin State Transfer)_ 아키텍쳐이다

### **REST(Representation State Transfer)**

REST는 2000년 HTTP의 주요 저자중 한 사람인 로이 필딩에 의해 제안되었다.
그는 웹의 장점을 최대한 활용할 수 있는 아키텍처로써 REST를 발표하였고, 이를 통해 웹기술이 지속적으로 발전할 수 있게 되었다.

많은 사람들이 REST가 URI 패턴과 GET,POST 등의 HTTP Method 의 조합으로 이루어져 있다고 생각하지만,REST 아키텍처는 아키텍처라는 그 이름답게 결국 제약조건들의 집합이다.

REST는 다음 여섯가지 제약조건을 가지고 있다.

1. Client-Server
1. Stateless
1. Cache
1. Uniform Interface
1. Layered System
1. Code On Demand(Optional)

각각에 대하여 살펴보도록 하자

#### **Client-Server**

흔히들 이야기하는 서버와 클라이언트의 관계를 의미한다. 대부분 현대 애플리케이션은 서버-클라이언트 기반으로 만들어지고 사실상 이 구조를 벗어나는 것 자체가 힘든일이기 때문에 자연스럽게 만족되는 제약조건이라 할 수 있다.

#### **Stateless**

서버는 클라이언트의 각각의 요청을 모두 별개의 것으로 인식해야하고 이를 위해 모든 요청은 각자가 필요한 정보를 모두 담고 있어야 한다.
요청 하나만 봐도 그 요청의 내용을 바로 알아볼 수 있어야 한다는 것이다.

#### **Cache**

HTTP라는 존재하는 프로토콜을 사용하기 때문에 웹에서 사용되는 기술인 캐시 역시 사용 가능하다.
모든 서버응답은 캐시의 사용가능여부를 알고있어야하여, 캐시 사용을 고려한 설계가 필요하다.

#### **Uniform Interface**

구성요소들(서버,클라이언트)간의 인터페시으가 균일애햐 한다는 의미이다. 구체적으로 미디어 유형이나, 리소스 식별자 등을 구별하는 문법이 상호간 동일해야 한다는 뜻이다.
이를 위해 REST는 네가지 제약사항을 제시하고 있다.

- identification of resources
  리소스를 식별하는 방법이 동일해야 한다. 흔히들 URI를 통해 리소스를 식별한다
- manipulation of resources through representation
  리소스 자체를 전송하는 것이 아닌 리소스의 표현을 전송한다는 뜻. 서버에 있는 리소스가 아니라 현재 리소스의 상태를 반영하는 표현체를 전송함으로써 서버의 리소스 상태가 변해도 클라이언트에는 영향을 끼치지 못한다.
- self-descriptive messages
  요청이나 응답에는 스스로를 설명하는 정보를 포함하고 있어야 한다. 따라서 수신자는 이 정보를 통해 메시지를 이해할 수 있다.
- hypermedia as the engine of application state
  애플리케이션의 상태는 하이퍼미디어에 의해 변경된다는 것이다. 따라서 서버는 하이퍼미디어를 통해 다음 액션에 대한 선택지를 클라이언트에게 제공해야 한다.

#### **Layered System**

클라이언트 혹은 서버 모두 미들웨어 구성을 추가할 수 있는 구조를 가지고 있어야 한다는 의미이다.

#### **Code On Demand(Optional)**

서버는 클라이언트로 실행 가능한 프로그램을 전달할 수 있어야 한다. 쉽게 Javascript를 생각하면 된다. 그러나 이 조건은 선택사항이며 필수적이지는 않다.

### **RESTful API**

RESTful API(REST API)란 위의 제약조건에 따르는 애플리케이션 프로그래밍 인터페이스를 뜻한다. 그러나 우리가 RESTful API라고 부르는 것들은 사실 REST하지 않은 경우가 대부분이다. 특히, *self-descriptve messages와 hypermedia as the engine of application state 원칙을 지키는 것이 상당히 까다로운 편이기 때문에 이 두원칙을 지키지 못하는 경우가 많다.*

물론 로이 필딩은 이런 API를 REST라 불러선 안된다고 주장하지만 이미 많은 개발자들과 기업들은 REST 원칙을 지키지 못한 API들을 RESTful API라고 부르고 있다.

### **개인적인 생각**

원칙적인 REST 아키텍처 스타일을 알고있는 많은 사람들이 현대에 많은 글들과 회사에서 단지 몇가지 URL 규칙과 HTTP 메소드가 필요했을 뿐인 많은 사람들이 REST란 용어를 납치했다고까지 표현한다. 사실 개인적으론 이러한 논쟁이 머리아프게만 느껴지고 무슨 의미가 있나 싶기는 하지만, 이런 논쟁들이 좀 더 견고하고 표준화된 RESTful API 개념을 정립하는데 도움이 되겠거니 생각한다.

이런한 논쟁을 뒤로 하고 나는 결국 나만의 RESTful API를 정의 내렸다. Restful API란 _HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)를 명시하고, HTTP Method(POST,GET,PUT,DELETE)를 통해 해당 자원에 대한 CRUD Opertaion을 적용한것_.
논란이 끝나면 새로운 정의가 나올 수 있겠지만 일단은 이렇게 알고있는게 마음편하다.
