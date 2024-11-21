
# Confluent Kafka 서버 설정 가이드

Confluent Kafka 사이트에서 Kafka 서버를 설정하는 과정을 단계별로 설명합니다.

## 1. 클러스터 생성

Kafka 클러스터는 여러 Kafka 브로커(노드)로 구성되어 있으며, 각 브로커는 메시지를 처리하고 저장하는 역할을 합니다.

- Confluent Cloud에 로그인하여 **Environments** 메뉴로 이동합니다.
- **Create cluster/Add cluster** 버튼을 클릭하여 새 클러스터를 생성합니다.
  - 필요한 유형(표준, 기본, 프리미엄 등)을 선택합니다.
  - 원하는 클라우드 서비스와 리전을 설정합니다.
  - 클러스터의 이름을 지정하고 **Launch Cluster** 버튼을 클릭하여 클러스터를 생성합니다.

> 💡 **Note**: 클러스터 생성 시, 브로커의 개수와 성능에 따라 처리량과 안정성이 달라집니다.

## 2. 토픽 생성

Kafka 토픽은 메시지를 저장하는 단위로, 특정한 메시지 흐름을 관리합니다.

- 클러스터 대시보드에서 **Topics** 메뉴로 이동합니다.
- **Create Topic** 버튼을 클릭하여 새 토픽을 생성합니다.
  - 토픽 이름을 설정합니다.
  - Partitions를 설정하여 토픽의 병렬 처리를 조정합니다.
  - 기타 설정(최대 메시지 크기, 보존 기간 등)을 필요에 따라 구성합니다.
- 설정을 완료하고 **Create with Defaults**를 클릭하여 토픽을 생성합니다.

> 💡 **Tip**: 파티션 수는 토픽의 병렬 처리 성능에 영향을 미치며, 복제 인수는 데이터의 내구성에 영향을 줍니다.

## 3. Schema Registry 페이지로 이동

1. 클러스터 대시보드에서 **Schema Registry** 메뉴를 선택합니다.
2. **Subjects** 페이지로 이동하여 새로운 스키마 주제를 생성할 준비를 합니다.

> 💡 **Note**: `topic-name-value` 형식은 데이터 스키마가 Kafka 토픽 데이터와 연결되도록 규칙화된 명명 방식입니다.

## 4. 새 스키마 주제(Subject) 생성 및 스키마 등록

1. **Create Subject** 버튼을 클릭하여 새 주제를 생성합니다.
2. **Subject Name**에 `topic-name-value` 형식으로 토픽 이름을 입력합니다. (`topic-name` 부분은 실제 Kafka 토픽 이름으로 대체합니다.)
3. **Schema Type**을 선택합니다:
   - **Avro** 또는 **JSON Schema** 중 하나를 선택하여 데이터 스키마 타입을 설정합니다.
4. 스키마 정의를 JSON 형식으로 입력합니다. 예를 들어, 아래와 같은 JSON 스키마를 입력합니다:

   ```json
   {
     "type": "record",
     "name": "GPS",
     "namespace": "com.cop.sp",
     "doc": "gps schema",
     "fields": [
       {
         "name": "latitude",
         "type": "double",
         "doc": "lat gps data"
       },
       {
         "name": "longitude",
         "type": "double",
         "doc": "lon gps data"
       }
     ]
   }
   ```

5. 스키마를 입력한 후, **Create** 또는 **Save** 버튼을 클릭하여 스키마를 등록합니다.

## 5. 스키마 호환성 설정

1. 등록한 주제(Subject)를 선택하여 상세 페이지로 이동합니다.
2. **Compatibility Settings** 섹션에서 호환성 설정을 확인하거나 변경할 수 있습니다.
   - **Compatibility Level**을 `BACKWARD`, `FORWARD`, `FULL`, `NONE` 중에서 선택합니다.
   - 예를 들어, **Backward Compatibility**를 유지하려면 `BACKWARD`를 선택합니다.
   - 각 호환성 옵션의 의미는 다음과 같습니다:
     - `BACKWARD`: 새 스키마가 이전 스키마로 작성된 데이터를 읽을 수 있도록 보장합니다.
     - `FORWARD`: 이전 스키마가 새 스키마로 작성된 데이터를 읽을 수 있도록 보장합니다.
     - `FULL`: 새 버전과 이전 버전이 양방향으로 데이터를 읽을 수 있도록 보장합니다.
     - `NONE`: 호환성 검사를 비활성화합니다.
3. 설정을 변경한 후, **Save** 버튼을 클릭하여 적용합니다.

## 6. 프로듀서 생성 (Go 언어 예시)

프로듀서는 Kafka 클러스터에 메시지를 전송하는 역할을 합니다. Confluent Kafka에서는 다양한 방법으로 프로듀서를 설정할 수 있습니다. 아래는 Go 언어를 사용해 프로듀서를 생성하는 예제입니다.

```go
package main

import (
    "fmt"
    "github.com/confluentinc/confluent-kafka-go/kafka"
)

func main() {
    p, err := kafka.NewProducer(&kafka.ConfigMap{"bootstrap.servers": "your_kafka_broker_address"})
    if err != nil {
        panic(err)
    }
    defer p.Close()

    topic := "your_topic_name"
    for _, word := range []string{"Hello Kafka", "Confluent Kafka Guide"} {
        err = p.Produce(&kafka.Message{
            TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
            Value:          []byte(word),
        }, nil)

        if err != nil {
            fmt.Printf("Failed to produce message: %v", err)
        }
    }
    p.Flush(15 * 1000)
}
```

> 💡 **Note**: `bootstrap.servers`에는 클러스터의 브로커 주소를 입력하며, 이는 프로듀서가 브로커에 연결하는 데 사용됩니다.

## 7. 컨슈머 생성 (Go 언어 예시)

컨슈머는 Kafka 클러스터에서 메시지를 수신하여 처리하는 역할을 합니다.

```go
package main

import (
    "fmt"
    "github.com/confluentinc/confluent-kafka-go/kafka"
)

func main() {
    c, err := kafka.NewConsumer(&kafka.ConfigMap{
        "bootstrap.servers": "your_kafka_broker_address",
        "group.id":          "your_consumer_group_id",
        "auto.offset.reset": "earliest",
    })
    if err != nil {
        panic(err)
    }
    defer c.Close()

    c.SubscribeTopics([]string{"your_topic_name"}, nil)

    for {
        msg, err := c.ReadMessage(-1)
        if err == nil {
            fmt.Printf("Received message: %s", string(msg.Value))
        } else {
            fmt.Printf("Consumer error: %v", err)
        }
    }
}
```

> 💡 **Tip**: `group.id`는 컨슈머 그룹을 지정하며, 같은 그룹에 속한 컨슈머는 서로의 메시지를 공유하지 않습니다.

## 8. 프로듀서를 통한 메시지 전송

프로듀서가 생성된 후, 특정 토픽으로 메시지를 전송할 수 있습니다. 메시지를 전송할 때는 `Produce` 메서드를 사용하며, 전송한 메시지의 성공 또는 실패 여부는 콜백을 통해 확인할 수 있습니다.

## 9. 컨슈머를 통한 메시지 수신

컨슈머가 생성된 후, 메시지를 수신하여 처리할 수 있습니다. 예제에서는 `ReadMessage` 메서드를 사용해 메시지를 가져오며, 수신된 메시지를 처리 후 종료합니다.

---

이 가이드를 통해 Confluent Cloud 웹 UI를 사용해 Kafka 클러스터와 Schema Registry를 설정하고, 기본적인 메시지 송수신이 가능한 프로듀서와 컨슈머를 구성할 수 있습니다. 각 단계에서 발생할 수 있는 오류나 세부 설정은 Kafka 클라이언트 문서를 참조하면 도움이 됩니다.
