# E-Commerce Ontology

온라인 쇼핑몰 도메인 온톨로지 - 실습 및 Data Mesh용 (2025)

## 개요

이 온톨로지는 전자상거래 도메인의 핵심 개념들을 정의합니다.

## 주요 클래스

- **Party**: 사람 또는 조직을 포괄하는 상위 클래스
  - **Person**: 개인
  - **Organization**: 조직
  - **Customer**: 쇼핑몰 고객
- **Product**: 상품
- **Order**: 주문
- **OrderItem**: 주문 항목
- **Payment**: 결제
- **Review**: 리뷰
- **Address**: 주소

## 주요 속성

### Object Properties
- `placedBy`: 주문자 (Order → Customer)
- `contains`: 주문 포함 항목 (Order → OrderItem)
- `refersTo`: 참조 상품 (OrderItem → Product)
- `paysFor`: 결제 대상 (Payment → Order)
- `writtenBy`: 리뷰 작성자 (Review → Customer)
- `about`: 리뷰 대상 (Review → Product)
- `deliveredTo`: 배송지 (Order → Address)

### Data Properties
- `customerId`: 고객 ID
- `orderId`: 주문 ID
- `orderDate`: 주문 날짜
- `totalAmount`: 총액
- `rating`: 평점 (1-5)

## 사용 방법

Protégé나 다른 OWL 편집기로 `ecommerce-ontology.owl` 파일을 열어서 사용할 수 있습니다.

## Ontology IRI

`https://github.com/sechan9999/ecommerce-ontology`

## 작성자

Grok 실습

## 날짜

2025-11-18
