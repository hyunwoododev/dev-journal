# DynamoDB 단일 테이블 아키텍처 이해

DynamoDB의 단일 테이블 아키텍처는 여러 유형의 데이터를 하나의 테이블에 저장하여 쿼리 성능을 최적화하는 설계 방식입니다. 이 문서는 단일 테이블 아키텍처의 핵심 개념과 설계 방법, 그리고 Sort Key(SK)에 문자열을 사용하는 이유에 대해 설명합니다.

## 단일 테이블 아키텍처의 핵심

*   **하나의 테이블에 모든 데이터 저장:** 관계형 데이터베이스처럼 여러 테이블로 분리하는 대신, 애플리케이션에 필요한 모든 데이터를 하나의 DynamoDB 테이블에 저장합니다.
*   **Partition Key (PK)와 Sort Key (SK)의 효과적인 활용:**
    *   **PK (Partition Key):** 데이터를 분산시키는 역할을 하며, 일반적으로 엔티티의 최상위 식별자를 사용합니다 (예: `USER#{user_id}`, `PRODUCT#{product_id}`). PK가 같은 항목들은 같은 파티션에 저장됩니다.
    *   **SK (Sort Key):** 동일한 PK를 가진 항목들을 정렬하고 필터링하는 역할을 합니다. SK를 사용하여 엔티티 유형을 구분하고, 계층적 관계를 표현하며, 범위 쿼리 및 필터링을 수행할 수 있습니다.
*   **Global Secondary Index (GSI)를 활용한 추가적인 쿼리 지원:** PK와 SK만으로는 표현할 수 없는 쿼리를 위해 GSI를 사용합니다.

## Sort Key (SK)에 문자열을 사용하는 이유

SK에 문자열을 사용하는 것은 단일 테이블 아키텍처의 핵심이며, 다음과 같은 중요한 기능을 수행합니다.

1.  **엔티티 유형 구분:** 여러 유형의 엔티티를 하나의 테이블에 저장할 때, SK에 문자열을 사용하여 각 항목이 어떤 유형의 엔티티인지 명확하게 구분할 수 있습니다.

    *   예시:
        *   `PK = USER#{user_id}`, `SK = USER` (사용자 정보)
        *   `PK = USER#{user_id}`, `SK = ORDER#{order_id}` (사용자의 주문 정보)
        *   `PK = PRODUCT#{product_id}`, `SK = PRODUCT` (상품 정보)
        *   `PK = PRODUCT#{product_id}`, `SK = REVIEW#{review_id}` (상품에 대한 리뷰 정보)

2.  **계층적 데이터 표현:** SK에 문자열을 사용하여 계층적인 데이터 관계를 표현할 수 있습니다.

    *   예시:
        *   `PK = USER#{user_id}`, `SK = PROFILE` (사용자 프로필)
        *   `PK = USER#{user_id}`, `SK = ADDRESS#HOME` (사용자의 집 주소)
        *   `PK = USER#{user_id}`, `SK = ADDRESS#WORK` (사용자의 회사 주소)

3.  **범위 쿼리 및 필터링:** SK는 동일한 PK를 가진 항목들을 정렬하는 역할을 합니다. 문자열을 사용하면 접두사(Prefix) 기반의 범위 쿼리 및 필터링이 가능해집니다.

    *   예시: 특정 사용자의 모든 주문을 조회하려면 `PK = USER#{user_id}`이고 `SK`가 `ORDER#`로 시작하는 항목들을 쿼리하면 됩니다.

4.  **복합적인 정보 표현:** SK에 여러 정보를 조합하여 표현할 수 있습니다.

    *   예시: `PK = USER#{user_id}`, `SK = ORDER#{order_id}#ITEM#{item_id}` (특정 주문의 특정 상품 정보)

5.  **GSI(Global Secondary Index)와의 연동:** GSI의 PK만으로는 원하는 데이터를 정확하게 특정하기 어려울 때, GSI의 SK에 문자열을 사용하여 추가적인 필터링을 수행할 수 있습니다.

## 잘못된 예시와 올바른 예시 비교

SK를 `REVIEW#{REVIEW_ID}PROFILE#{PROFILE_ID}`와 같이 구성하는 것은 잘못된 방식입니다. 이렇게 구성하면 하나의 항목에 여러 개의 리뷰와 프로필 정보를 담으려고 하는 것으로 해석될 수 있으며, 데이터 중복, 쿼리 복잡성 증가, 확장성 제한 등의 문제가 발생합니다.

각 엔티티(리뷰, 프로필 등)는 *별도의 항목*으로 저장하고, `partitionKey`와 `SK`를 사용하여 이들 간의 관계를 표현하는 것이 올바른 접근 방식입니다.

## 결론

SK에 문자열을 사용하는 것은 DynamoDB 단일 테이블 아키텍처의 핵심이며, 데이터 모델링의 유연성을 높이고 다양한 쿼리 패턴을 효율적으로 지원하기 위해 필수적입니다. 단순히 데이터를 저장하고 조회하는 것뿐만 아니라, 데이터 간의 관계를 표현하고 효율적인 쿼리를 수행하기 위해 SK에 문자열을 적절히 활용해야 합니다.

## 추가 참고 자료

*   [Fundamentals of Amazon DynamoDB Single Table Design with Rick Houlihan](https://www.youtube.com/watch?v=KYy8X8t4MB8)
