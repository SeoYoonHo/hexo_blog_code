---
title: Spring DI(의존성주입) & IoC(제어의역전)
toc: true
date: 2022-04-22 13:42:47
tags:
  - Spring
  - CS
  - DI
  - 의존성주입
  - IoC
  - 제어의역전
categories:
  - [Spring, Spring DI & IoC]
---
Spring에서의 핵심 개념인 의존성주입(Dependency Injection, DI)과 제어의 역전(Inversion of Control, IoC)에 대해 알아보도록 하자. 두 개념은 사실 거의 같은 의미라고 할 수 있다. **의존성 주입은 외부에서 두 객체간의 관계를 결정해주는 디자인 패턴이고, 이 객체들의 사용에 대한 책임이 프레임워크에 있다는 것이 바로 제어의 역전이기 때문이다.** 즉, Spring은 IoC를 통해 객체들의 의존성을 분리시켜 결합도를 낮춰주는 역할을 하고 있다.

<center><img src="/post_images/Spring/DI.png"></center>

<!-- more -->

### **의존성 주입(Dependency Injection)이란?**
위에서 DI에 대해 간략하게 소개하였지만 구체적인 예시를 들어 좀 더 명확하게 이해해보도록 하자. 예를 들어 다음과 같이 Cafe 객체가 coffee 객체를 사용하고 있는 경우에 우리는 Cafe객체가 Coffee객체에 의존성이 있다고 표현한다.

{% codeblock lang:java%}
  public class Cafe { 
    private Coffee coffee; 

    public Cafe() {
      this.coffee = new Coffee();
    }
  }
{% endcodeblock %}

- #### **문제점**
  위와 같은 Cafe와 Coffee클래스는 강하게 결합되어 있는 문제점이 있다. 만약 Cafe클래스에서 Coffee가 아닌 다른 Juice같은 상품이 필요하게 되면 Cafe클래스의 생성자에 변경이 필요하다.이는 유연성이 떨어진다는 의미이다.

  또한 위의 Cafe와 Coffee는 객체들간의 관계가 아니라 클래스들간의 관계가 맺어져 있다는 문제도 있다.  올바른 객체지향적 설계라면 객체들간의 관계를 맺는것을 지향해야 한다.

- #### **해결책**
  위와 같은 문제를 해결하기 위해서 우선 다형성을 활용해야 한다. Coffee, Juice등 여러가지 제품을 하나로 표현하기 위해 Beverage라는 Interface를 선언한다.

  {% codeblock lang:java%}
    public interface Beverage {}
  {% endcodeblock %}
  {% codeblock lang:java%}
    public class Coffee implements Beverage {}
  {% endcodeblock %}

  이제 Coffee와 Cafe의 강한 결합을 제거해주도록 하자. 이를 제거하기 위해서 **외부에서 Coffee를 주입(Injection)** 받아야 한다.

  {% codeblock lang:java%}
    public class Cafe { 
      private Beverage beverage; 
      
      public Store(Beverage beverage) { 
        
        this.beverage = beverage; 

      } 
    }
  {% endcodeblock %}

  {% codeblock lang:java%}
    public class CafeFactory { 
      public void cafe(){
        //Bean 생성
        Coffee coffee = new Coffee();

        //의존성 주입
        Cafe cafe = new Cafe(coffee);
      }
    }
  {% endcodeblock %}

  위와 같이 CafeFactory라는 외부에서 객체를 주입해줌으로써 의존관계를 재설정할 수 있다. Spring에서 CafeFactory의 역할은 Application Context가 수행하고 있다.

  그리고 위에서 설명했듯이 **객체를 사용하고 책임지는 역할이 사용자가 아닌 Spring Framework에 넘어가 있고 이러한 개념을 제어의 역전(Inversion of Control, IoC)이라고 한다.**

### **정리**
Spring에서는 DI 컨테이너를 통해 서로 강하게 결합되어 있는 두 클래스를 분리하고, 두 객체간의 관계를 결정해줌으로써 결합도를 낮추고 유연성을 확보하고자 한다. Spring에서 이러한 객체들을 Bean이라 정의하고 있다. 또한 Spring에서는 이러한 Bean들을 성능을 위해 기본적으로 Sigleton으로 관리하고 있다.

### **마치며**
오늘은 DI와 IoC에 대해서 알아보았다. 오늘은 의존성 주입과 제어의 역전의 기본적인 개념에 대해서만 알아보았지만 좀 더 파고든다면 의존성 주입에도 여러 종류가 있다는 것을 알 수 있다. 이러한 개념들에 대해서는 좀 더 찾아보길 바라며 포스팅을 마치도록 하겠다.