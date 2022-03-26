---
title: 객체지향 5원칙(SOLID)
date: 2022-03-26 19:04:21
tags:
  - CS
  - SOLID
  - 객체지향 5원칙
  - 객체지향
categories:
  - CS
  - 객체지향 5원칙(SOLID)
toc: true
---

2000년대 초 로버트 마틴이 명명한 객체 지향 프로그래밍의 다섯가비 기본원칙을 마이클 페더스가 원칙의 앞글자를 따서 다시 **SOLID**라고 소개하였다.
**SOLID의 5대원칙**은 다음과 같다.

1. 단일 책임 원칙(Single Responsibility Principle)
1. 개방 폐쇄 원칙(Open/Cloed Principle)
1. 리스코프 치환 원칙(Liskov Subsitution Principle)
1. 인터페이스 분리 원칙(Interface Segregation Principle)
1. 의존관계 역전 원칙(Dependency Inversion Principle)

이 다섯가지 원칙애 대해 자세히 알아보도록 하자

<!-- more -->

### **단일 책임 원칙 : SRP(Single Responsibility Principle)**

모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화 해야함을 일컫는다. 
다시말해 **클래스가 제공하는 모든 서비스는 단 하나의 책임을 수행하는데 집중**되어야 한다는 뜻이다.

- SRP를 지키지 못하는 경우
  {% codeblock lang:java%}
    public class 강아지 {
      final static Boolean 수컷 = true;
      final static Boolean 암컷 = false;
      Boolean 성별;

      void 소변보다() {
        if (this.성별 == 수컷) {
          // 한쪽 다리를 들고 소변을 보다.
        } else {
          // 뒷다리 두 개를 굽혀 앉은 자세로 소변을 본다.
        }
      }
    }
  {% endcodeblock %}

메소드에서 수컷, 암컷의 경우를 모두 구현하려고 하여 단일 책임 원칙을 위반하고 있는것을 볼 수 있다.

- SRP를 지키는 경우
{% codeblock lang:java%}
  abstract class 강아지 {
      abstract void 소변보다();
  }

  class 수컷강아지 extends 강아지 {
      void 소변보다() {
          // 한쪽 다리를 들고 소변을 본다.
      }
  }

  class 암컷강아지 extends 강아지 {
      void 소변보다() {
          // 뒷다리 두 개로 앉은 자세로 소변을 본다.
      }
  }
{% endcodeblock %}

그래서 위와 같이 추상클래스를 두고 각각의 클래스가 자신의 특징에 맞게 메소드를 구현해서 사용하는것으로 리팩토링 할 수 있다.

### **개방 폐쇄의 원칙 : OCP(Open Closed Principle)**

개방-폐쇄 원칙은 *소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다*는 프로그래밍 원칙이다.

다시 말하면 요구사항의 변경이나 추가사항이 발생하더라도, 기존 구성요소는 수정이 일어나지 말아야 하며 쉽게 확장이 가능하여 재사용할 수 있어야 한다는 뜻이다.
로버트 마틴은 OCP는 관리가 용이하고 재사용 가능한 코드를 만드는 기반이며, OCP를 가능케 하는 중요한 메커니즘은 추상화와 다형성이라고 설명한다.

비밀번호 암호화를 강화해야한다는 요구사항이 새롭게 들어왔다고 가정하자. 비밀번호 암호화를 강화하기 위해 다음과 같이 SHA-256알고리즘을 사용하는 새로운 PasswordEncoder를 생성하였다.

{% codeblock lang:java%}
  public class SHA256PasswordEncoder{

    public String encryptPassword(final String pw){
      // SHA-256 암호화 메소드
    }

  }
{% endcodeblock %}

그리고 새로운 비밀번호 암호화 정책을 적용하려고 봤더니 UserService를 다음과 같이 수정해주어야 한다는 것이 발견되었다.

{% codeblock lang:java%}
  public class UserService{

    private final SHA256PasswordEncoder passwordEncoder;

    ...

  }
{% endcodeblock %}

해당 코드의 문제가 보이는가?
위 코드는 나중에 비밀번호 암호화 정책을 변경해야 한다는 요구사항이 온다면 또 다시 UserService에 변경이 필요해진다. 이는 기존의 코드를 수정하지 않아야 하는 개방 폐쇄 원칙에 위배된다.
결국 이러한 문제를 해결하고 **개방 폐쇄 원칙을 지키기 위해서는 추상화에 의존**해야 한다.

{% codeblock lang:java%}
  public interface PasswordEncoder { 
    String encryptPassword(final String pw); 
  }

  public class SHA256PasswordEncoder implements PasswordEncoder {
    
    @Override
    public String encryptPaswword(final String pw){
      ...
    }
  }

  public class UserService{

    private final PasswordEncoder passwordEncoder;

    public void adduser(final String email, final String pw){
      final String encryptedPassword = passwordEncoder.encryptPassword(pw);
      ...
    }

  }
{% endcodeblock %}

개방 폐쇄 원칙이 본질적으로 얘기하고자 하는것은 결국 추상화이다. 이는 런타임 의존성과 컴파일 의존성이 객체 지향 프로그래밍에서는 동일하지 않다는 것을 의미한다.


### **리스코프 치환 원칙 : LSP(Liskov Substitution Principle)**

리스코프 치환 원칙은 1988년 바바라 리스코프가 올바른 상속 관계의 특징을 정의하기 위해 발표한 내용으로, 하위 클래스는 상위클래스의 모든 기능들을 수행 할 수 있어야함을 의미한다.
즉 상위클래스을 사용하는 객체는 그 객체가 하위클래스로 변경되어도 문제없이 수행되어야 한다.

정말 당연한 원칙이지만 실무에서 좀처럼 지켜지지 않는 원칙중에 하나이다.

대부분의 경우 상위클래스의 메소드를 override하면서 문제가 발생하게 된다. 상위클래스의 기존 메소드를 하위클래스에서 잘못 수정하게 되면서 문제가 생기게 되는것이다.

LSP를 잘 지키기 위해서는 override를 안하면 되는것이지만, 이는 절대적인 방법이 아니다.
결국 상속을 할 떄 override가 필요하다면 상위클래스의 기능을 충실히 수행하고 기능의 추가만 신중하게 수행하면 된다.

LSP는 결국 현재 *하위클래스가 상위클래스의 기존 메소드의 의미를 해지지 않는지* 신중히 고민을 하고 올바르게 상속하라는 의미이다.

### **인터페이스 분리의 원칙 : ISP(Interface Segregation Principle)**
인터페이스 분리 원칙은 클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다는 원칙이다. 다시 말하면 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다는 원칙이다.

하나의 큰 인터페이스를 상속받기보다는 인터페이스를 구체적이고 작은 단위들로 분리시켜 *꼭 필요한 인터페이스만 구현*하다는 의미이다.

SRP가 클래스의 단일책임을 강조했다면 ISP는 인터페이스의 단일책임을 강조한다.


### **의존 역전 원칙 : DIP(Dependency Inversion Principle)**

의존 역전의 원칙은 다음과 같은 내용을 담고 있다.

1. 상위 모듈은 하위 모듈에 의존해서는 안된다. 상위 모듈과 하위 모듈 모두 추상화에 의존해야한다.
1. 추상화는 세부 사항에 의존해서는 안된다. 세부사항이 추상화에 의존해야한다.

의존 역전 원칙은 클래스 사이에는 의존관계가 존재하기 마련이지만, **구체적인 클래스에 의존하지말고 추상화에 의존하는 설계**를 의미한다.

우리는 위의 예시들을 살펴보면서 의존 역전 원칙에 준수하도록 코드를 수정한 경험이 있다. 바로 OCP를 설명하기 위해 살펴봤던 SimplePasswordEncoder 예제가 DIP에 부합하는 리팩토링 과정이다.

![DIP](/post_images/dip.png)

예시에서 살펴보았든 의존 역전 원칙(DIP)는 개방 폐쇄 원칙(OCP)와 밀접한 관련이 있으며, 의존 역전 원칙이 위배되면 개방 폐쇄 원칙 역시 위배되게 될 가능성이 높다.