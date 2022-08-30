---
title: Netty Hashed Wheel Timer
toc: true
date: 2022-08-30 11:06:54
tags:
  - Netty
  - Timer
  - Wheel Timer
  - Hashed Wheel Timer
categories:
  - [Netty, Timer]
---
오랜만의 포스팅하는 오늘의 주제는 Netty의 HashedWheelTimer에 대한 분석글이다. 해당 Timer의 장점과 사용이유 Java의 Timer의 한계점 등에 대해서 소개해보려 한다.

<!-- more -->

### **Java.util.Timer**
Java는 기본적으로 Timer 객체를 제공하고 있다. 사용법은 TimerTask를 생성하여 Timer에 등록한 후 시간값을 입력하여주면 된다. 그 예시는 다음과 같다.

{% codeblock lang:java%}
  import java.util.Timer;
  import java.util.TimerTask;

  public class Main {
      public static void main(String[] args) {

          Timer m = new Timer();
          TimerTask task = new TimerTask() {
              @Override
              public void run() {
                  System.out.println("timer!");
              }
          };

          m.schedule(task,5000);// 5초 뒤에 실행
      }
  }
{% endcodeblock %}

사용법은 간단하지만 빈번한 타이머 사용의 경우, 해당 클래스로는 좋은 성능을 내기 힘들다. java.util.Timer 객체는 내부적으로 TimerTask를 binary heap 구조로 관리하기 떄문에 TimerTask 추가, 삭제시 일어나는 동기화시간이 길어질 수 밖에 없다. 다음은 타이머 객체의 동기화 코드이다.

<center><img src="/post_images/Netty/Timer/timer_code.png"></center>

이런 동기화시간의 장기화는 프로그램의 성능을 저하시킬 수 밖에 없다. 최근 Timer 기능을 사용하여 업무를 설계해야 하는일이 발생하였고 위와 같은 문제점을 알고있었기에, TimingWheel 자료구조를 사용하는 Netty의 HashedWheelTimer 객체를 사용하기로 하였다.

### **Timing Wheel**
TimingWheel 의 기본구조는 고정된 크기의 순환배열이다.

<center><img src="/post_images/Netty/Timer/timingWheel.png"></center>

배열의 각 버킷을 타임슬롯(time slot)이라고 부르고, 해당 슬롯은 타임아웃이 발생할 떄 처리해야할 작업에 대한 리스트를 담고있다. 기본적인 동작과정은 슬롯 시간 간격마다 타임슬롯을 순회하며, 해당 타임슬롯에 담긴 내용을 처리하는 방식으로 이루어진다.

해당 타임 슬롯에 타임아웃을 추가하는 경우 단순히 add연산자를 통해 추가할수 있으므로, java의 타이머에 비해 동기화시간히 짧아지게 된다.

### **HashedWheelTimer**
위에서 언급한 휠 타이머는 최대 시간 제한이 있다는 단점이 있다. 타임슬롯을 넘어가는 값을 등록할 수가 없는것이다.
<center><img src="/post_images/Netty/Timer/wheelTimer.png"></center>


이러한 단점을 극복하기 위해 Netty의 HashedWheelTimer는 남은 바퀴수까지 계산하여 등록시간제한문제를 해결하였다.
<center><img src="/post_images/Netty/Timer/hashedWheelTimer.png"></center>

### **마치며**
오늘은 POJO의 등장배경에 대해 알아보았다. 이 포스팅을 정리하며 EJB부터 Spring까지 Java 서버개발의 역사에 대해 전반적으로 알 수 있었다. 사실 EJB에 대해서 정확하게 알 수는 없지만 Spring, Pojo의 등장 배경에 대해 좀 더 잘 이해할 수 있는 시간이어서 의미있는 시간이었다.

이전 포스팅에서도 언급했지만 개발자로 살아간다면, 자신이 사용하거나 사용할 기술들에 대하여 '왜 사용되어야 하는가?' 라는 의문은 계속 제기하면서 살아야 한다고 생각한다. 그런 의미에서 이번 POJO 포스팅은 좀 더 개발다운 사람이 되는 시간이었다.

출처:
- https://d2.naver.com/helloworld/267396