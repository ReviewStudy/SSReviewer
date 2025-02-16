
Tempo - easy-to-use, and high-scale distributed tracing backend.

Tempo는 오브젝트 스토리지 하나만으로 운영이 가능한 고확장성 분산 추적 백엔드

![image](https://github.com/user-attachments/assets/cd16daad-d090-4652-95af-7e82896317ec)


## Tracing Pipeline Components
1. **클라이언트 계측 (Client Instrumentation)**
2. **파이프라인 (Pipeline)**
3. **백엔드 (Backend - Tempo)**
4. **시각화 (Visualization)**
   
### 1. Client Instrumentation
![image](https://github.com/user-attachments/assets/f645e584-2f1d-44d5-8fe8-e04ec0d22752)


클라이언트 계측이란 애플리케이션 내부에 계측 포인트를 추가하여 스팬(span)을 생성하고 전송하는 과정
- 예시) OpenTelemetry, Zipkin등 오픈소스 계측 프레임워크에서 SDK제공
- Java - Java Agent Auto Instrumentation
https://github.com/open-telemetry/opentelemetry-java-instrumentation

### 2. Pipeline
- 애플리케이션에서 생성된 스팬을 버퍼링하고 백엔드로 전송하는 과정을 구성
- 분산 추적 도구는 일반적으로 여러 소스에서 **동시에 텔레메트리 데이터를 수신**하므로 애플리케이션 확장에 따라 도구에서 사용하는 리소스를 최적화해야 합니다. 이를 위해 추적 파이프라인은 데이터 버퍼링, 추적 일괄 처리, 스토리지 백엔드로의 후속 라우팅과 같은 기능을 수행
### 3. Backend Tempo
- 추적이 수집된 후에는 나중에 시각화 및 분석을 위해 저장해야 합니다. 이 과정에서 너무 많은 리소스를 소비하지 않고 애플리케이션과 함께 확장할 수 있는 스토리지 백엔드가 필요
- 인메모리 데이터베이스에 의존하는 대신 Amazon S3, Google Cloud Storage, Microsoft Azure Blob Storage와 같은 널리 사용되는 객체 스토리지 솔루션을 추적 스토리지 백엔드로 사용

### Trace란 무엇인가?
- 트리 형태로 구성된 원격 분석 데이터
- 트레이스는 여러개의 스팬으로 이루어짐
- 트레이스들은 루트 스팬이 있고 여기에서 부터 여러가지 자식 스팬들이 뻗어나온다. 
![image](https://github.com/user-attachments/assets/ecc68ee9-a783-490d-8983-dea5cd9819da)

#### Span이란?
- 시스템 내에서 서비스에 의해 수행되는 단일 작업 단위

### Span의 주요 Component
- name: span name
- duration: 시작 시간과 종료 시간
- status: ok, error, unset등 otel span status를 따름
	- https://opentelemetry.io/docs/concepts/signals/traces/#span-status
- kind: server, client, producer, consumer, internal, unspecified

### 스팬의 속성 (Attributes)

스팬에는 추가적인 메타데이터를 저장할 수 있으며, 키-값(key-value) 형식으로 저장
1. Span Attributes: 특정 작업과 관련된 정보를 저장 (예: 쇼핑 앱에서 장바구니 추가 시 user_id, item_id, cart_id 저장)
2. Resource Attributes: 스팬을 생성한 엔터티(애플리케이션, 클러스터, 네임스페이스, 컨테이너 등)의 정보 저장
3. Event Attributes: 특정 시간에 발생한 이벤트 저장
4. Link Attributes: 다른 스팬과의 관계를 생성하는 링크 역할 수행

### Traces and Telemetry
- profile, metrics, trace, logs 이 4가지의 관계를 가지는 것은 매우 중요하다
![image](https://github.com/user-attachments/assets/5e4d482f-42fe-45af-9e66-1dd527f2c133)

#### 메트릭 (Metrics) - "무언가가 발생하고 있다"
- 시스템의 상태를 한눈에 파악할 수 있도록 제공되는 숫자 기반 데이터
- 경보(Alert)의 기초가 되는 데이터로, 정해진 임계값과 비교 가능

#### 로그 (Logs) - "어떤 일이 발생하고 있는가"
- 개별 프로세스에서 발생한 활동을 기록하는 감사(audit) 로그
- 애플리케이션 내부에서 어떤 일이 일어나고 있는지 자세한 정보 제공

#### 트레이스 (Traces) - "각 단계에서 무슨 일이 발생하는가"
- 데이터가 흐르는 경로에서 각 단계별로 발생하는 이벤트를 그래픽적으로 표현
- 각 단계가 완료되는데 걸리는 시간을 시각적으로 분석 가능

#### 프로파일링 (Profiles) - "애플리케이션이 CPU 및 메모리를 어떻게 활용하는가"
- 특정 시점에서 애플리케이션이 자원을 어떻게 사용하고 있는지 확인 가능

#### 메트릭과 로그의 한계
- 메트릭과 로그만으로는 서비스 간의 상호작용과 의존성을 파악하기 어려움
- 트레이스는 서비스 간 관계를 시각적으로 표현하여 문제 파악을 쉽게 해줌
- 예를 들어, 특정 서비스가 다운되었을 때, upstream 서비스와 downstream 서비스에 미치는 영향을 분석 가능

### Tempo Configuration
![image](https://github.com/user-attachments/assets/34ce6868-e6eb-4771-ada4-849158c1b11c)


### Distributor
- 여러 형식의 스팬을 수락하고, 트레이스 ID를 기반으로 해시링을 사용하여 Ingesters로 라우팅

#### Ingester: 
- 수신된 트레이스를 블록(block) 단위로 저장
- Bloom filter 및 인덱스를 생성한 후, 백엔드에 저장
###### 저장 형식
```
<bucketname> / <tenantID> / <blockID> / <meta.json>
                                      / <index>
                                      / <data>
                                      / <bloom_0>
                                      / <bloom_1>
                                        ...
                                      / <bloom_n>
```

#### Query Frontend
- 검색 공간을 sharding하여 여러 개의 작은 요청으로 나누고 병렬 처리 수행
- shard 단위로 Querier에 요청을 전달
#### Querier
- 특정 트레이스 ID를 조회하고 필요하면 백엔드(Object Storage)에서 데이터 조회
- 블룸 필터와 인덱스를 활용하여 빠르게 검색 수행

**Query Frontend**는 요청을 받은 후, **검색 공간(Block ID)을 샤딩**하여 Querier들에게 나누어 준다.
• **Querier**는 할당된 샤드에서 트레이스 ID를 찾고, 필요하면 백엔드(Object Storage)에서 데이터를 불러온다.
• 최종적으로 Querier가 찾은 데이터를 Query Frontend가 모아서 사용자에게 반환한다.

### 컴팩터 (Compactor)
- 백엔드 스토리지에서 블록을 가져와 병합하여 전체 블록 수를 줄임


## 용우의 질문
1. TSDB를 사용한 이유가 있을텐데 그 이유는 무엇이고 OBject Storage를 사용함으로써 얻는 이득은??
2. 로그파일 트레이스 파일 -> 데이터독 순식간에 쌓인다 스토리지 먹는다!
3. 뤼튼 - 2주 lifecycle -> s3 glacier 
