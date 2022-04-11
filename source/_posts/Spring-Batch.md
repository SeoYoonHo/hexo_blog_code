---
title: Spring Batch
tags:
  - Sping
  - Batch
  - Spring Batch
  - STIS
  - CS
categories:
  - [Spring, Spring Batch]
toc: true
date: 2022-04-09 15:24:59
---

### **Batch Program이란?**
일반적으로 배치(Batch) 프로그램이라 하면, 일련의 작업들을 하나의 작업단위로 묶어 연속적으로 일괄처리 하는 것을 말한다.
예를 들어 하루전날의 집계된 요금데이터를 집계해야한다고 가정해보자. 이 업무는 하루에 1번만 수행하면 된다. 이를 위해 api를 구성하는것은 낭비가 된다. 또한 대량의 데이터를 처리하는 도중 실패가 된다면?? 이에 대한 처리를 어떻게 할 것인가?

이러한 단발성 대용량 데이터를 처리하는 프로그램을 배치프로그램이라 한다. 위에서 한 고민들을 살펴보면 단순히 집계하는 비즈니스 로직 이외에 고려해야할 부분이 많다는 것을 알 수 있다. 개발자가 비즈니스 로직 이외의 다른 부분들에 대한 고민을 덜어주도록 지원하는것을 프레임워크라 한다. 대표적인 프레임워크는 Spring이 있다.

<!-- more -->

그렇다면 Spring에서는 이런 배치프로그램을 지원하는 기능이 없을까?

Spring 진영에서는 Spring Batch를 지원하며, 감히 말하건대 이 프레임워크가 Java개발자에게 있어서는 최선의 배치프레임워크라 자신한다. Spring Batch에 대해서는 아래에서 더 자세하게 다뤄보기로 하고 그 이전에 배치 프로그램이란 무엇인지 알고 넘어가도록 하자.

배치 프로그램이란 다음의 조건들을 만족하는 프로그램이다
  - 대용량 데이터 - 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등의 처리를 할 수 있어야 한다.
  - 자동화 - 사용자의 개입 없이 실행되어야 한다.
  - 견고성 - 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 한다.
  - 신뢰성 - 무엇이 잘못되었는지 추적가능해야 한다(로깅, 알림)
  - 성능 - 지정한 시간 안에 처리를 완료하거나 동시에 실행되는 다른 어플리케이션을 방해하지 않도록 수행되어야 한다.

자 이제는 이러한 배치프로그램의 개발을 지원하는 Spring Batch에 대해 더 자세하게 알아보자

### **Spring Batch**
Spring Batch는 2007년 Accenture와 SpringSource간의 협업으로 탄생하게 되었다. Accenture사의 배치업무에 대한 노하우와 SpringSouce의 기술력이 합쳐진 결과물이라고 할 수 있다.

Spring Batch는 기본적으로 Spring의 특징 그대로를 사용할 수 있다. DI, AOP, 서비스 추상화 등의 스프링 프레임워크의 특징을 사용할 수 있으며, Accenture의 배치 아키텍쳐를 사용 할 수 있다는 의미이다.

여기에선 Spring Batch의 큰 줄기만 다뤄보도록 하겠다. 자세한 사용가이드는 공식 doc을 참고하면 된다.(https://docs.spring.io/spring-batch/docs/current/reference/html/)

<center><img src="/post_images/SpringBatch/spring-batch-reference-model.png"></center>

위의 그림은 스프링 배치의 Job 구성에 관한 그림이다. 해당 그림에서 스프링의 큰 줄기개념들이 나오게 된다. 각각의 용어와 내용에 대해서 알아보도록 하자


  - #### **Job**
    Spring Batch에서 Job은 전체 배치 프로세스를 캡슐화하는 객체이다. 말이 어려울 수 있지만 쉽게 생각하면 결국 배치 프로그램을 이루는 업무단위의 최상위 계층이라고 생각하면 된다. Job은 사용자가 정의하기 나름이지만 최소한 1개의 Step으로는 이루어져 있으며, 복잡한 Job이 아닌 이상 2~10개의 Step을 권장한다.
    
    배치 프로그램은 결국 여러개의 Job과 그 Job에 속해있는 여러개의 Step들로 이루어진다고 보면 된다.

  - #### **Step**
    위에서 말했듯이 Step은 Job을 구성하는 요소이다. Step은 job의 독릭접으로 실행되는 순차적인 단계를 캡슐화한 도메인 객체이며, 실제 배치 처리를 정의하고 컨트롤하는 데 필요한 모든 정보를 가지고 있다.
    설명이 굉장히 모호하게 느껴질 수 있는데, 이는 Step의 모든 내용은 Job을 만드는 개발자의 재량이기 때문이다. 즉 개발자에 따라 Step은 간단할 수 도 있고 복잡할 수도 있다. 데이터베이스에서 데이터를 읽어서 파일을 쓰는 간단한 작업이 될 수도 있고, 프로세싱의 일부를 처리하는 복잡한 비즈니스 로직일 수도 있다.

    Step의 데이터 처리방법은 크게 두가지가 있다
    - Chunk지향 방식
    - Tasklet 방식

    Spring Batch에서 추천하는 방식은 Chunk 지향방식이고 많은 사람들도 이 방식을 추천한다. 하지만 비즈니스로직이 Chunk 지향 방식으로 처리가 안되는 경우에는 Tasklet 방식을 사용할 수밖에 없다.

    1. ##### **Chunk 지향 방식**
        Spring Batch 구현체의 대부분은 '청크 지향'으로 개발되었다. 청크 지향 프로세싱이란 한 번에 데이터를 하나씩 읽어와서 트랜잭션 경계 내에서 쓰여질 '청크'를 만드는 것이다. 읽은 항목 수가 커밋 간격과 같아지면 ItemWriter가 청크 전체를 write하고, 트랜잭션이 컴ㅅ된다. 다음은 이 절차를 도식화한 그림이다.

        <center><img src="/post_images/SpringBatch/chunk-oriented-processing.png"></center>

        다음 의사코드는 동일한 개념을 단순화된 형식으로 보여준다.
        {% codeblock lang:java%}
          List items = new Arraylist();
          for(int i = 0; i < commitInterval; i++){
              Object item = itemReader.read();
              if (item != null) {
                  items.add(item);
              }
          }
          itemWriter.write(items);
        {% endcodeblock %}
    2. ##### **Tasklet 방식**
        위에서 말했듯 청크 기반 처리로 구현할 수 없는 비즈니스 로직을 위해 Spring Batch는 TaskletStep을 제공한다. Tasklet은 excute 메소드 하나를 가진 심플한 인터페이스인데, 이 메소드는 RepeatStatue.FINISHED가 리턴되거나 실패했단 뜻으로 exception이 던져지기 전까지 TaskletStep을 반복적으로 호출한다. 각 Tasklet 호출은 트랜잭션으로 감싸져 있다.

        Tasklet을 실행하려면 빌더의 tasklet 메소드에 Tasklet 인터페이스를 구현한 빈을 넘겨야 한다.

        {% codeblock lang:java%}
          @Bean
          public Step step1() {
              return this.stepBuilderFactory.get("step1")
                    .tasklet(myTasklet())
                    .build();
          }
        {% endcodeblock %}

### **Quartz vs SpringBoot Scheduler**
Spring Batch의 큰 줄기는 위에서 다뤄봤다. 좀 더 자세한 설명과 예제가 필요하다면 공식 doc을 참고하길 바란다. 위의 내용을 읽었다면 배치 프로그램에서 중요한 부분이 빠졌다는 것을 알 수 있다. 바로 스케줄링 부분이다. 매일 또는 몇 시간 단위로 일정하게 프로그램을 실행시켜줄 스케줄러 기능이 Spring Batch에는 제공되지 않는다.

그래서 보통은 Quartz 라이브러리의 스케줄링 기능과 결합하여 많이들 사용하였다. 하지만 최근들어서는 SpringBoot에서 Scheudler 기능을 제공하기 시작했고, 이 기능과 묶어서 사용하는 케이스가 많아지게 되었다.

현재 내가 운영하는 시스템의 배치프로그램도 SpringBoot + Spring Batch 로 배치 프로그램이 동작하고 있다.

### **마치며**
서버개발자의 삶을 살아가다보면 배치프로그램의 개발은 피할 수 없는 일이 될 것이다. 위에서 소개한 Spring Batch는 그 개발 프레임워크로써 최고의 도구가 되기를 바란다.