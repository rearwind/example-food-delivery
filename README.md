# 서비스 시나리오

### 기능적 요구사항

1.	고객이 메뉴를 선택하여 주문한다
2.	(수정)고객이 선택한 메뉴에 대해 결제하되, 결제가 되지 않으면 주문되지 않는다. (기존: 고객이 선택한 메뉴에 대해 결제한다)
3.	주문이 되면 주문 내역이 입점상점주인에게 전달된다.
4.	상점주는 주문을 수락하거나 거절할 수 있다
5.	상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다
6.	(수정)고객은 어느 때에도 주문을 취소할 수 있다. 주문취소 시 결제, 요리, 배달이 취소되어야 한다. (기존: 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다)
7.	요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다
8.	라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다.
9.	고객이 주문상태를 중간중간 조회한다.
10.	주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
11.	고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다
12.	(추가)고객은 쿠폰관리 프로그램에 가입할 수 있다.
13.	(추가)2번째 배달완료된 주문마다 쿠폰이 발행된다.

### 비기능적 요구사항

1. 장애격리
    1. 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다. Async(event-driven), Eventual Consistency
    1. 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시 후에 하도록 유도한다. Circuit breaker, fallback
1. 성능
    1. 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다. CQRS
    1. 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다  Event driven


# 분석/설계

### 완성된 모델

![image](https://user-images.githubusercontent.com/119660065/206334467-77656bc8-53c9-4e46-a6c6-c3401325d29a.png)



### 완성된 모델에 대한 기능적/비기능적 요구사항을 커버하는지 검증

 
![image](https://user-images.githubusercontent.com/119660065/206336266-7feb99b5-9534-4a23-a771-764808f8385f.png)

    - 고객이 메뉴를 선택하여 주문한다 (order) (ok)
    - 고객이 선택한 메뉴에 대해 결제하되, 결제가 되지 않으면 주문되지 않는다. (pay 를 req/res 로 호출, sync) (ok)
    - 주문이 되면 주문 내역이 입점상점주인에게 전달된다 (copy order info)
      (주문이 되면 미리 배송 서비스에도 주문정보를 전달한다)

# 

![image](https://user-images.githubusercontent.com/119660065/206340338-27c3d094-686b-4a43-907e-6292af7fa823.png)

    - 상점주는 주문을 수락하거나 거절할 수 있다 (1. 주문수락, 2. 주문거절) (ok)
    - 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다 (3. 요리시작 입력, 4. 요리완료 입력) (ok)
    - 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다 (4. 요리완료 입력 -> 배송 서비스에서 주문상태 업데이트) (ok)
    - 라이더가 해당 요리를 Pick한 후, 앱을 통해 통보한다. (4. pick -> notify) (ok)
    - 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다 (4. confirm delivered) (ok)
   
# 
       
![image](https://user-images.githubusercontent.com/119660065/206341757-2062fd1e-f67f-403f-9eda-072eb38ed6a2.png)
         
    - 고객이 주문상태를 중간중간 조회한다. (1. cqrs(MyPage) 이용) (ok)
    - 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다 (ok) 
      => 2. 주문수락, 주문거절, 요리시작, 요리완료, 배달시작(picked), 결제취소 이벤트 발생 시 notify
    
# 

![image](https://user-images.githubusercontent.com/119660065/206352401-12b3343a-3add-4547-b8d6-0f2a55cd7945.png)
      
    - 고객은 쿠폰관리 프로그램에 가입할 수 있다. (1. sign up) (ok)
    - 2번째 배달완료된 주문마다 쿠폰이 발행된다. (2. 배달완료 -> issue coupon -> 매 2번째 주문이면 쿠폰 발행 -> 알림) (ok)
    
# 

![image](https://user-images.githubusercontent.com/119660065/206352957-5ccd877f-ff30-4ac8-83b7-7bfd65732de5.png)
        
    - 고객은 어느 때에도 주문을 취소할 수 있다. 주문취소 시 결제, 요리, 배달이 취소되어야 한다. (ok)
      => cancel -> 결제취소 -> 요리취소 -> 배달취소 



### 비기능 요구사항에 대한 검증


    - 상점관리 기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다. Async(event-driven), Eventual Consistency (ok)
      => 주문(order)과 상점관리(store) 서비스는 별도의 마이크로서비스로, Req-Res 가 아닌 Pub-Sub 을 이용, Async 로 설계

![image](https://user-images.githubusercontent.com/119660065/206354021-4d9772b6-ab4c-4b09-871a-0eaa8cc2bd6f.png)

# 

    - 결제시스템이 과중되면 사용자를 잠시동안 받지 않고 결제를 잠시 후에 하도록 유도한다. Circuit breaker, fallback (ok)

1. order 에서 payment 서비스로 결제 요청 구간에 Circuit breaker 설정 

![image](https://user-images.githubusercontent.com/119660065/206360037-36e77232-1039-4a65-ae31-4ea82f3a8be0.png)

2. order 서비스의 application.yml 에 Circuit breaker enable = true 로 설정

![image](https://user-images.githubusercontent.com/119660065/206361607-ff64c2ad-2e16-46cb-af30-eae0bb9a9d33.png)

3. order 의 external/PaymentService.java 에 fallback (PaymentServiceImpl) 구현체 설정

![image](https://user-images.githubusercontent.com/119660065/206364976-945ef0c9-e5d8-4751-900a-06c6e0a17786.png)

4. PaymentServiceImpl.java 에 fallback 구현
     
![image](https://user-images.githubusercontent.com/119660065/206362897-9b971964-d920-4422-8314-d4c5bfaa82b2.png)


# 

    - 고객이 자주 상점관리에서 확인할 수 있는 배달상태를 주문시스템(프론트엔드)에서 확인할 수 있어야 한다. (ok)
      => 주문시스템 내 고객서비스(customer)에 MyPage 를 CQRS 로 모델링하여 Read 성능 고려 (기능 요구사항 9번 참고)
      
    - 배달상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다 Event driven (ok)
      => 주문 및 배달 이벤트 발생 시마다 알림 (기능 요구사항 10번 참고)
    

- 모델은 모든 기능/비기능 요구사항을 커버함.


# 체크포인트

## 1. Saga (Pub / Sub)

    주문(order) 서비스에서 결제 후 OrderPlaced event 를 Publish
    
![image](https://user-images.githubusercontent.com/119660065/206369736-7dff1c68-fcac-4f70-94db-0b45d59a8bff.png)
    
    상점(store) 서비스는 이 이벤트를 Subscribe 하고, 이벤트 수신 시 주문정보 생성

1.
![image](https://user-images.githubusercontent.com/119660065/206370247-be2b7d43-3f1e-49a9-8167-285758de6049.png)
    
2.
![image](https://user-images.githubusercontent.com/119660065/206370509-0753ab39-eac7-44cc-a92d-41ec5bcb3417.png)


=> 작동

    주문 서비스에서 주문
    
![image](https://user-images.githubusercontent.com/119660065/206383269-9a33d3bc-7900-46ed-b2ca-207b9580994f.png)

    상점 서비스에 주문정보 생성됨
    
![image](https://user-images.githubusercontent.com/119660065/206383565-6dbab611-5c36-4b0f-92a1-d1aea9eac462.png)


## 2. CQRS

    MyPage (CQRS) : orderId , 상태(status) 
![image](https://user-images.githubusercontent.com/119660065/206371658-eb13da07-90be-4164-89fd-7e2207d885e1.png)

    OrderPlaced 이벤트 발생 시 Create
![image](https://user-images.githubusercontent.com/119660065/206371997-dfa8f6c4-0844-4136-a735-21beebc87e71.png)

    주문상태 변경 이벤트 발생 시 Update
![image](https://user-images.githubusercontent.com/119660065/206372168-cdaa3beb-8f31-4c56-b6e3-dd55158f2e88.png)


=> 작동

    주문 후 MyPage 조회하면 1번 주문이 "주문됨"으로 조회됨
    
![image](https://user-images.githubusercontent.com/119660065/206384456-3458707a-8fdc-4054-aa63-3d95872052ff.png)

    상점에서 주문 수락하면 "수락됨"으로 조회됨
    
![image](https://user-images.githubusercontent.com/119660065/206385182-e61603b3-884d-468e-9d58-0b61757ba7d7.png)

    상점에서 요리를 시작하면 "요리시작됨"으로 조회됨
    
![image](https://user-images.githubusercontent.com/119660065/206385613-77635db8-9c93-4e56-b832-d530a395e082.png)



## 3. Compensation / Correlation

    고객이 주문 후 취소 시 결제, 요리, 배달도 모두 취소하도록 Pub-Sub 으로 구현, orderId 를 키값으로 서비스 간 Correlation 구현
    
1. 주문 취소는 Rest(DELETE) 가 아닌 Command 로 하여, 삭제가 아닌 취소 상태로 변경하는 것으로 구현
![image](https://user-images.githubusercontent.com/119660065/206376073-57ff644d-fb7a-496b-ad40-518ed2b2f7de.png)

2. 주문을 취소하고, 해당 주문으로 OrderCancelled 이벤트를 생성하여 publish
![image](https://user-images.githubusercontent.com/119660065/206376426-e9f50bda-d875-4600-add5-1d81252a032a.png)

3. 결제 서비스는 OrderCancelled 이벤트 수신 시 해당 orderId 를 가진 결제의 상태를 "결제취소됨"으로 변경 후 PayCancelled 이벤트 발생
![image](https://user-images.githubusercontent.com/119660065/206376745-6966af2d-179c-403d-8921-a9a904b68b5f.png)

4. 요리, 배달 서비스는 상동


=> 작동

    현재 주문(1번 주문)은 배달 시작된 상태 (상점에서는 "요리완료됨", 배달에서는 "배달시작됨" 상태임)
    
    1번 주문에 대해 주문 서비스에서는 "주문취소됨", 결제 "결제취소됨", 상점 "요리취소됨", 배달 "배달취소됨"으로 상태 변경됨
![image](https://user-images.githubusercontent.com/119660065/206388090-a1e98633-253a-4aef-9480-802773d3db3d.png)
    
![image](https://user-images.githubusercontent.com/119660065/206388368-3f457569-4fcd-4593-911f-323f059b95c0.png)

![image](https://user-images.githubusercontent.com/119660065/206388458-c186b2b3-cd3a-43a4-9cc2-a5946ad84577.png)


## 4. Request / Response

    주문 직후 주문 서비스에서 결제 서비스로 결제 요청을 Req / Res 로 구현
    
    
1. 주문(OrderPlaced 이벤트 발생) 직후 Payment Proxy(PaymentService)의 pay(결제) 호출 - Sync (Req/Res)
![image](https://user-images.githubusercontent.com/119660065/206377640-719f105e-272a-40f1-b1d3-aa63660f060d.png)


2. 주문 서비스의 external 의 PaymentService (FeignClient 로 결제 대행 인터페이스 정의 => 인터페이스를 통해 payment 의 pay 가 호출됨)
![image](https://user-images.githubusercontent.com/119660065/206378328-933abdde-97d3-461b-9d3f-7e0a500ffb7c.png)


=> 작동

    order 는 구동하고 payment 는 구동하지 않은 상태에서 주문하면 주문 실패
    
![image](https://user-images.githubusercontent.com/119660065/206389600-817226a5-324a-4c6e-9fcf-e22c01f597fa.png)


    payment 구동 
    
![image](https://user-images.githubusercontent.com/119660065/206390188-d770b81d-b9e7-4a1f-88ae-f085d5692e7d.png)


    다시 주문하면 주문 성공
    
![image](https://user-images.githubusercontent.com/119660065/206390394-5389eb8d-e90f-49b8-bb1f-387df49e3182.png)


## 5. Circuit Breaker

서킷 브레이커 구현 - 상단의 "비기능 요구사항에 대한 검증" 참고



    결제 서비스를 호출(Sync) 시 결제 서비스 처리 성능이 느려지도록 딜레이 발생 코드 추가

    @PrePersist
    public void onPrePersist(){
        
    ...    
        try {
            Thread.currentThread().sleep((long) (400 + Math.random() * 250));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    
    
=> 동작

    timeout 임계치를 600 으로 준 상태에서 부하 발생
    
    - siege -c2 -t10S  -v --content-type "application/json" 'http://localhost:8081/orders POST {"customerId":"khl", "qty":1, "foodId":1, "address":"seoul"}'
    
![image](https://user-images.githubusercontent.com/119660065/206393549-8241e0a6-2710-4873-b133-4309359e9aca.png)

 
    timeout 임계치를 520 으로 준 상태에서 부하 발생하면 실패 건수가 늘어나면서 서킷 브레이커 발동

![image](https://user-images.githubusercontent.com/119660065/206396953-cdfce711-57db-4e4b-baf0-0597a6b5c4ab.png)

![image](https://user-images.githubusercontent.com/119660065/206394420-80e38199-ac3a-4bac-8925-60c381060b92.png)


## 6. Gateway / Ingress

    Gateway 서비스는 :8088 포트로 코드 제너레이션할 때 자동으로 서비스 및 application.yml 생성되었음
    
    - gateway 의 application.yml
    
![image](https://user-images.githubusercontent.com/119660065/206397376-567b8e73-4a33-4899-bd85-79e12601f086.png)

    쿠폰관리(couponMgmt)(http :8085/couponMgmts) 서비스 조회
    
![image](https://user-images.githubusercontent.com/119660065/206402016-6a6f7115-3fec-44fd-a59a-2b5e9461a5e2.png)

    쿠폰관리 서비스를 통해 서비스에 가입해본다. (http :8085/couponMgmts customerId="khl")
    
![image](https://user-images.githubusercontent.com/119660065/206403686-74b56c29-eecb-44dd-9afa-7becdc664c47.png)

    gateway를 통해 쿠폰관리 서비스에 가입한다. (http :8088/couponMgmts customerId="lee" orderCount=0)
    
![image](https://user-images.githubusercontent.com/119660065/206404232-b904b5d7-4607-45d4-a71a-6532caaa02ea.png)

    새로운 브라우저를 열고 쿠폰관리 서비스에 접속해본다 => 접속할 수 없음

![image](https://user-images.githubusercontent.com/119660065/206405274-2b9080e8-c3f3-448b-9fa1-5bd827e459b1.png)

    게이트웨이를 public 으로 설정 후 새 브라우저에서 게이트웨이를 통해 쿠폰관리 서비스에 접속해본다 
    
![image](https://user-images.githubusercontent.com/119660065/206405566-8f830581-4756-452b-b553-bec0524484ba.png)

![image](https://user-images.githubusercontent.com/119660065/206406221-8ec5bb7b-259b-4802-8d7a-ccb9264f4ac8.png)


    나머지 서비스들도 게이트웨이를 거쳐 호출 가능
