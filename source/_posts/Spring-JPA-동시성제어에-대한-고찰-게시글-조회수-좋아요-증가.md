---
title: Spring + JPA 동시성제어에 대한 고찰(게시글 조회수, 좋아요 증가)
toc: true
date: 2022-10-25 10:18:30
tags:
  - Spring
  - SpringBoot
  - JPA
  - MultiThread
  - Transaction Lock
  - CQRS
  - CS
categories:
  - [Spring, JPA]
---
서버를 운영하다 보면 다양한 동시성제어 상황을 다루게 된다. 그 중 가장 대표적인 이슈는 조회수나 좋아요 count 문제가 있을것이다. 오늘은 이 문제에 대해서 다뤄보는 시간을 가지도록 하겠다.

<!-- more -->

### **문제소개**
조회수 증가는 단순하게 생각하면 꽤 쉬운 코드가 될 것이다. 게시글 조회후 조회수를 Update시켜주는 로직만 추가시켜주면 된다. 구현에 앞서 테스트에 사용될 Post 엔티티는 다음과 같다.

{% codeblock lang:java%}
    @Entity
    @Data
    public class Post extends BaseTimeEntity{
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "post_id")
        private Long id;

        private String contents;

        private int cnt;

        @Enumerated(EnumType.STRING)
        private BoardTypeEnums boardType;

        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "members_id")
        private Member member;
    }
{% endcodeblock %}

위에서 말한대로 단순히 구현하게 된다면 다음과 같은 코드가 될 것이다.
{% codeblock lang:java%}
    public void findPostByIdTest(Long id) {
        Post testPost = postRepository.findById(id)
                                      .map(post ->
                                      {
                                          post.setCnt(post.getCnt() + 1);
                                          return post;
                                      })
                                      .orElseThrow(
                                              () -> new NoSearchException("not search post"));
    }
{% endcodeblock %}

위 메소드를 테스트해보자. 테스트코드는 다음과 같다.
{% codeblock lang:java%}
    @Test
    public void multiThreadTest() throws Exception {
        //given
        ExecutorService service = Executors.newFixedThreadPool(10);

        int count = 100;
        CountDownLatch latch = new CountDownLatch(count);

        //when
        for (int i = 0; i < count; i++) {
            service.execute(() -> {
                postService.findPostByIdTest(1L);
                latch.countDown();
            });
        }

        latch.await();

        //then
        Post testPost = postRepository.findById(1L).get();
        assertThat(testPost.getCnt(), equalTo(count));
    }
{% endcodeblock %}

조회수 100을 예상했지만 결과는 다음과 같다...
![TestResult1](/post_images/Spring_MultiThread/testResult1.png)

동시성 문제로 인해 86의 조회수가 날아가버렸다. 동시성을 고려하지 않은 업데이트가 어떤 영향을 미치는지 알았으니 이제 해결방법을 알아보도록 하자.

### **UPDATE문 사용**
조회수 1만 증가시키는 간단한 예제이기 때문에 단순 update쿼리를 이용하여 해결할 수 있다.

{% codeblock lang:java%}
    @Modifying
    @Query("update Post p set p.cnt = p.cnt + 1 where p.id=:postId")
    int increasCount(Long postId);
{% endcodeblock %}

위의 쿼리는 PK로 탐색했기때문에 단 하나의 레코드만 Lock된다.

테스트 결과는 다음과 같다.
![TestResult2](/post_images/Spring_MultiThread/testResult2.png)

동시성 문제가 해결됐음을 확인할 수 있다.
하지만 단순 조회수 증가기능이기때문에 해결가능했지만, 그렇지 않은 경우도 많다.
그러므로 다음단계로 나아가 비관적 Lock으로 해결해보자.

### **비관적 Lock**
Data JPA에서 다음과 같이 비관적 락을 사용할 수 있다.

{% codeblock lang:java%}
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select p from Post p where p.id = :postId")
    Optional<Post> findPostById_Locked_Pessimistic(Long postId);
{% endcodeblock %}

LockModeType을 설정할 수 있는데 PESSIMISTIC_WRITE이 베타락, PESSIMISTIC_READ가 공유락이다.

해당 동시성 제어는 읽기 후 업데이트이게 베타락을 설정해준다.

service메소드는 다음과 같다.

{% codeblock lang:java%}
    public void findPostByIdTest(Long id) {
        Post testPost = postRepository.findPostById_Locked_Pessimistic(id)
                                      .map(post ->
                                      {
                                          post.setCnt(post.getCnt() + 1);
                                          return post;
                                      })
                                      .orElseThrow(
                                              () -> new NoSearchException("not search post"));
    }
{% endcodeblock %}

테스트 결과는..
![TestResult3](/post_images/Spring_MultiThread/testResult3.png)

성공이다.

### **낙관적 Lock**
낙관전 Lock은 @Version을 사용하는데, 버전이 다르면 rollback시키기 떄문에 완벽한 조회수 증가를 얻을 수 없다.

만약 동시성 문제가 자주 발생하지 않고 롤백후 에러메세지 출력이 가능한 비즈니스라면 낙관적 Lock을 이용해도 되겠다.

### **Lcok 성능**
Lock을 이용하여 동시성 문제는 해결했지만, 필연저긍로 성능저하가 발생하게 된다.
각 방법들의 성능을 비교해보자

성능 비교방법은 각 방법별로 수행시간으로 비교해보겠다.(조회수 10000번)

- 동시성 고려 X
![](/post_images/Spring_MultiThread/performTime1.png)

- Update
![](/post_images/Spring_MultiThread/performTime2.png)

- 비관적 Lock
![](/post_images/Spring_MultiThread/performTime3.png)

동시성을 고려하지 않았을 떄 가장 빠르고, Update문이 비관적 Lock보다는 빠른것을 볼 수 있다.

### **성능 개선**
동시성 제어는 위의 방법들을 통해 어느정도 해결하였다. 하지만 동시성제어를 근본적으로 Lock을 통해 해결하다보니 조회서비스에 성능적인 이슈가 발생할 가능성이 높다. 

이를 해결하기 위해 두가지 측면으로 개선포인트를 잡아볼 수 있다.
- 업데이트 성능
- 조회와 업데이트 결합도 감소
위의 관점에서 개선을 진행해보려 한다.

    #### **업데이트 성능**
    업데이트가 자주 일어나서 성능 이슈가 발생할 가능성이 높다면, 업데이트를 덜 하면 된다는 아이디어이다.
    즉, 조회수를 1씩 증가시키는게 아닌 10 혹은 100씩 증가시키거나, 일정한 주기별로 증가시켜주는 방법이다.하지만, 이는 내부 변수에 대한 동시성 제어가 일어나야하므로, Thread Safe한 변수를 활요하는것이 좋다.

    또한 더욱 간단하게 redis같은 서비스를 이용하여 해결하는 방법도 있다. 기본적으로 redis는 단일 Thread이므로 동시성 제어에 대해 고민할 필요가 없고, 일반 데이터베이스에 비해 in/out속도가 월등히 빠르니 좋은 해결방법이 될 것이다.

    #### **CQRS**
    위 조회수 증가 로직들은 업데이트에서 락이 결리면 주 서비스는 조회에도 성능 이슈가 발생한다. 이를 위해 조회 서비스와 데이터에 변경이 가해지는 서비스들을 분리하여 실행하는 CQRS기법을 사용해 볼 수 있다.

    카프카나 rabbitmq등의 메시지 서비스를 이용하여 조회후 업데이트 이벤트를 발행하고, 이 이벤트를 구독하는 업데이트 서비스를 통해 조회와 업데이트 로직을 분리하여 둘의 결합도를 낮추는 방법이다.

    Spring에도 Event발행을 통해 해당 기능을 어느정도 구현할 수 있다.

위의 방법들 중 조회수를 10마다 증가시키는 방법과, Spring의 Event를 활용한 방법으로 서비스를 구현해보자

우선 Event클래스를 정의해주고
{% codeblock lang:java%}
    @Getter
    public class CountEvent extends ApplicationEvent {
        private final Long id;

        public CountEvent(Object source, Long id) {
            super(source);
            this.id = id;
        }
    }
{% endcodeblock %}

Publisher와 Listener를 정의해준다
{% codeblock lang:java%}
    @Component
    @RequiredArgsConstructor
    @Slf4j
    public class CountEventPublisher {
        private final ApplicationEventPublisher applicationEventPublisher;

        public void publish(final Long id){
            CountEvent countEvent = new CountEvent(this, id);
            applicationEventPublisher.publishEvent(countEvent);
        }
    }

{% endcodeblock %}
{% codeblock lang:java%}
    @Service
    @RequiredArgsConstructor
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    @Slf4j
    public class CountEvenetListener {
        private final PostRepository postRepository;

        private final ConcurrentHashMap<Long, AtomicInteger> postCountHashMap = new ConcurrentHashMap<>();

        @Async
        @EventListener
        public void handleCountEvent(CountEvent event) {
            Long postId = event.getId();

            postCountHashMap.compute(postId, (aLong, integer) -> integer == null ? new AtomicInteger(1) : integer);
            int count = postCountHashMap.get(postId).incrementAndGet();
            if (count % 10 == 0) {
                postCountHashMap.get(postId).updateAndGet(n -> n - count);
                postRepository.increasCountParam(postId, count);
                log.debug("count : " + count);
            }
        }
    }
{% endcodeblock %}

그리고 서비스 메소드를 다음과 같이 변경해준다
{% codeblock lang:java%}
    public void findPostByIdTest(Long id) {
        Post testPost = postRepository.findById(id)
                                      .orElseThrow(
                                              () -> new NoSearchException("not search post"));

        countEventPublisher.publish(id);
    }
{% endcodeblock %}

위와 같은 방식으로 성능도 잡고, 결합도도 낮춤으로써 서비스 품질을 올릴 수 있다.