# 프로젝트 요구사항 분석 및 정리
### 요약 : 온라인으로 상품을 등록하고 판매하는 e-commerce 시스템으로, 다량의 데이터와 높은 동시 접속자 수 환경에서 안정적으로 요청을 처리할 수 있다.

- 요구사항 수집
  - 판매자는 상품을 등록할 수 있다.
  - 판매자는 상품의 설명, 수량 등 정보를 관리할 수 있다.
  - 구매자는 상품을 구매할 수 있다.
  - 구매자는 상품을 검색할 수 있다.
  - 구매자는 결제수단을 등록하고 주문 시 사용할 수 있다.
  - 구매자는 배송지를 등록하고 주문시 사용할 수 있다.
  - 구매자는 완료된 주문의 상태를 조회할 수 있다.
  - 구매자는 완료된 주문의 배송 상태를 조회할 수 있다.
  - 구매자는 주문 내역 리스트를 볼 수 있다.
  - 구매자는 회원으로 등록할 수 있다.
  - 회원은 로그인을 할 수 있다.
  
### 주요 컴포넌트 설계
#### Catalog
- 역활: 상품 정보 관리
- 기능 : 
  - 상품 등록: POST /catalog/products
  - 상품 수정: PUT /catalog/products/{productId}
  - 상품 수량 변경: PUT /catalog/change-inventory-count
  - 상품 조회: GET /catalog/products/{productId}
  - 상품 검색: POST /catalog/search

### Payment
- 역활 : 결제 처리와 관련된 작업
- 기능 : 
  - 결제 수단 등록: POST /payment/methods
  - 결제 수단 변경: PUT /payment/methods/{methodId}
  - 결제: POST /payment/process-payment
  - 결제 결과 조회: GET /payment/payments/{paymentId}

### Order
- 역활 : 주문 처리를 수행하고 상태를 관리
- 기능 : 
  - 상품 주문: POST /order/process-order
  - 주문 상태 조회: GET /order/orders/{orderId}
  - 주문 내역 보기: GET /order/orders

### Delevery
- 역활 : 주문 완료 된 제품 배송, 상태 관리
- 기능:
  - 배송지 등록: POST /delevery/addresses
  - 배송 처리: POST /delevery/process-delevery
  - 배송 상태 조회: GET /delevery/deleveries/{deleveryId}

### Member
- 역활 : 회원 등록 관리와 인증
- 기능:
  - 회원 등록: POST /member/registration
  - 회원 정보 관리: PUT /member/members/{userId}
  - 로그인: POST /member/login
 
## 주문 처리과정 시퀸스 다이어그램
![image](https://github.com/user-attachments/assets/1a6c1adf-d457-4d32-8ede-e4f7d52efd1b)

## 아키텍처 고려사항을 설계에 적용하기

### 아키텍처 고려사항
- 대용량 트래픽 처리 : 동시에 많은 요청을 처리
- 탄력성 : 급증하는 트래픽 처리
- 안정성 : 트래픽 급증 또는 일부 장애 시 에러 최소화

### 요구 성능 계산
- 주요 목표 : 100만명 동시 접속자 처리
  - 동시 접속자 수 100만명
  - 100만명이 10분에 한번씩 구매
  - 분당 10만건 구매 처리
  - 초당 1666건의 구매 처리
 
- 대용량 트래픽 처리 => scale-out을 이용한 분산 처리, 일부 데이터 NoSQL적용
- 탄력성 => scale-out을 이용해서 대응, EDA를 사용해 보완
- 안정성 => EDA를 사용해 비동기 처리함으로써 장애 전파 최소화

- 병목 지점
  - 빈번한 상품 정보 변경 -> NoSQL 사용
  - 주문 처리 시 결제, 배송 관련 외부 시스템등의 병목 -> EDA 사용
 
- EDA 적용 가능한 부분
  - 결제처리(외부 연동을 통해야 하기 때문에 처리가 오래 걸릴 수 있음)
  - 배송 처리(배송 요청 시 응답이 필요 없고 수행해야 하는 동작만 있어서 비동기 처리에 적합)
  - 검색어에 대한 캐시(색인)업데이트(약간의 지연이 허용되기 때문에 비동기 처리 & 대량 처리가 용이함)
 
## 시스템 다이어그램
![image](https://github.com/user-attachments/assets/8f9617be-71ff-40e1-a34b-eca0e62fa9bf)

### 연계 서비스 리포지토리
1. blackFriday-OrderService
2. blackFriday-DeliveryService
3. blackFriday-PaymentService
4. blackFriday-SearchService
5. blackFriday-CatalogService
6. blackFriday-MemberService

