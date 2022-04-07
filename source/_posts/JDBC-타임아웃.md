---
title: JDBC 타임아웃
toc: true
date: 2022-04-06 19:47:25
tags:
  - JDBC
  - Timeout
  - Database
  - STIS
categories:
  - [Java, JDBC]
---
시스템을 운영하는 입장에서 개발을 진행한다면 다양한 상황들을 가정하게 된다. 프로그램이 갑작스럽게 죽었을 경우, 서버가 다운됐을 경우, DDos 공격을 당하게 되었을 경우등이 가정되는 상황들이다. 이러한 이벤트에서도 시스템은 피해를 최소화할 수 있어야 한다. 이와 관련하여 과거 진행되었던 개발건 중 JDBC 타임아웃 설정에 관한 좋은 경험이 있어 공유하고 설명하고자 한다.

### **DB서버의 먹통**
2020년 실시간으로 들어오는 다량의 데이터들을 DB에 적재하는 프로그램 개발을 맡게 되었다. Kinesis로부터 읽어들인 데이터를 DB에 적재하면 되는 단순할 수 있는 개발건이었지만, 기존에 운영중인 시스템의 구조를 바꾸는 일이라 영향도가 꽤나 큰 건이었다. 기능 개발은 단기간에 끝났지만 성능테스트와 발생할 수 있는 다양한 상황에 대응하는 기능들을 추가하는데 공수가 더 들어가게 되었다.

<!-- more -->

다양한 상황중 하나를 소개하고 그 대응책으로 고른 내용을 공유하고자 한다. 그 상황은 다음과 같다.
- DB서버가 장애가 날 경우 프로그램은 어떻게 동작해야하는가?

의식의 흐름대로 개발을 한다면 다음과 같은 프로세스대로 동작하면 된다.
1. DB가 죽을 경우 Kinesis로부터 Read하는 행위를 정지한다.
2. DB Reconnect를 주기적으로 실시한다.
3. DB Connect가 성공하는 순간 Kinesis로부터 Read행위를 재개한다.

이 프로세스에서 가장 핵심이 되는 문제는 1번, DB의 비정상적인 종료를 캐치하는 것이었다.
DB Transaction 프레임워크로 Mybatis를 설정하였기에 statement 타임아웃 설정을 해주었고, 나머지 2,3번 프로세스대로 개발을 진행하였다. 개발 완료 후 DB를 중지시키는 테스트를 진행하였을 때 프로그램은 DB의 비정상적인 종료를 캐치하지 못하였다. DB의 응답을 기다리면서 Kinesis로부터 계속 데이터를 읽어들였고, 차오르는 메모리로 인해 oom이 발생하며 프로그램은 종료되었다.
아래는 mybatis에서 설정해준 statement timeout 설정파일이다.

{% blockquote %}
  ...
  \<setting name="defaultStatementTimeout" value="25000"/>
  ...
{% endblockquote %}

무엇이 문제였을까? mybatis의 타임아웃 설정이 잘못되었던건가? 이를 이해하기 위해선 JDBC의 타임아웃 과정을 보면 이해할 수 있다.
앞으로 나올 내용은 Naver O2에 포스팅 된 아주 좋은내용의 블로그를 참고하였다(https://d2.naver.com/helloworld/1321)

### **JDBC type**
JDBC는 자바 프로그램이 DBMS에 일관된 방식으로 접근할 수 있도록 API를 제공하는 자바 클래스들의 모임이다. 즉 데이터베이스에 연결 및 작업을 하기 위한 JAVA의 표준 인터페이스이다.
JDBC의 유형은 총 4가지가 있는데 현재 재직중인 회사와 운영중인 시스템에선 주로 Type 4를 사용하고 있으므로 이에 대해서 다룬다.
(추후에 JDBC에 대해서도 자세하게 다루도록 하겠다.)
Type 4는 순수 자바로 이루어져 있으며 Java Socket으로 DB에 직접 교신하는 방식이다. 때문에 Socket Timeout값을 제대로 설정하지 않았을 경우 DB의 문제를 늦게 감지하고 장애로 이어질 수 있다. 위에서 설명한 상황이 바로 이런 상황에 해당된다.

### **DBMS 통신 타임아웃 계층**
 <center><img src="/post_images/jdbc_timeout/timeout_layer.png"></center>

위의 그림은 WAS와 DBMS의 통신시 타임아웃 계층을 단순화한 것이다. 여기서 Instance를 기본 Java Application으로 대체해도 문제가 없다. 그림에서 보이듯 상위레벨의 타임아웃은 하위레벨의 타임아웃에 의존성을 가지고 있다. Socket Timeout이 정상적으로 동작하지 않으면, 그보다 상위인 Statment Timeout과 Transaction Timeout도 정상적으로 동작하지 않는다.

JDBC Socket Timeout의 default값은 OS의 Socket timeout에 영향을 받는다. 별도의 설정이 없을 경우 OS의 Default Socket Timeout값이 적용된다. 즉, 위의 상황에서 30분이 지난다면 Exception이 발생되었을 것이다. 하지만 실시간 프로그램에서 30분은 너무나 긴 시간이고 별도의 시간 설정이 필요해 보인다.

이제부터 각 타임아웃의 의미와 Mybatis에서 Socket Timeout 설정하는 방법에 대해 다뤄보자

  - #### **Transaction Timeout**
    Transaction Timeout은 프레임워크나 애플리케이션 레벨에서 유효한 타임아웃이다.

    간단히 말하면 전체 Statement를 수행하는데 걸리는 시간의 Timeout이라고 할 수 있다. 즉 StatementTimeout * N(Statement 수행수) 정도로 생각해볼 수 있다.

    Mybatis의 경우 Trasaction Timeout설정은 존재하지 않고 Statement Timeout설정만 존재한다.
  
  - #### **Statement Timeout**
    Statement 하나가 얼마나 오래 수행되어도 괜찮은지에 대한 한계값이다. Mybatis의 설정이 위의 예제에서 보여준 방법을 이용하면 된다.

  - #### **Socket Timeout**
    위에서도 말했듯이 Type 4는 Socket을 이용한 DB통신 방식을 이용하고, 이는 DBMS가 Connection Timeout을 처리하지 않음을 의미한다.

    즉, 애플리케이션과 DBMS 사이의 장애를 감지할 수 있는 방법은 Socket Timeout값을 설정하는 방법밖에 없다. 이를 간과하였기에 우리 시스템에서는 DB장애를 감지할 수 없었던 것이다.

    단, Socket Timeout 값을 Statment의 수행시간을 제한하기 위해 사용하는것은 바람직하지 않으므로, Statement Timeout 값보다는 큰 값을 사용하기를 권장한다.

    Socket Timeout에는 두가지 옵션이 있고, 드라이버별로 설정방법이 다르다.
    
    1. Socket Connect 시 타임아웃(connectTimeout): Socket.connect(SocketAddress endpoint, int timeout) 메서드를 위한 제한 시간
    1. Socket Read/Write의 타임아웃(socketTimeout): Socket.setSoTimeout(int timeout) 메서드를 위한 제한 시간

    드라이버별 설정방법을 여기서 모두 다루지는 않고 Oracle의 경우 Mybatis에서 어떤식으로 설정해줬는지 내용을 공유하려 한다.

    {% blockquote %}
      ...
      \<property name="driver.oracle.jdbc.ReadTimeout" value="10000"/>\
      \<property name="driver.oracle.net.CONNECT_TIMEOUT" value="10000"/>
      ...
    {% endblockquote %}

    위의 두 내용이 내가 Mybatis에 설정해준 Socket Timeout 값이다.

### **끝으로**
해당 내용을 공부하고 운영환경에 적용하면서 이 설정이 실제로 쓰일까에 대한 고민이 많이 있었다. 하지만 프로그램 반영 후 알 수없는 이유로 DB에 로그파일이 쌓이기 시작하며 스토리지가 모두 소비되었고, DB 서버가 죽는 상황이 실제로 발생하게 되었다. 해당 장애상황에서 대비되었던 설정으로 인해 모든 데이터를 유실 없이 복구할 수 있었다.

역시 운영환경에서 생각할 수 있는 모든 변수는 대비하는게 맞다는 경험을 한번 더 쌓게 되는 계기가 되었다.