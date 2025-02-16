# High-Scale Distributed Tracing with Tempo
#Tempo #DistributedTracing #ObjectStorage #Telemetry

**Summary**: Tempo is a scalable distributed tracing backend that operates efficiently using only object storage. It integrates seamlessly with various telemetry components to provide comprehensive tracing and analysis.

## Content

Tempo는 오브젝트 스토리지 하나만으로 운영이 가능한 고확장성 분산 추적 백엔드입니다.

### Tracing Pipeline Components
1. **클라이언트 계측 (Client Instrumentation)**
2. **파이프라인 (Pipeline)**
3. **백엔드 (Backend - Tempo)**
4. **시각화 (Visualization)**

#### 1. Client Instrumentation
클라이언트 계측은 애플리케이션 내부에 계측 포인트를 추가하여 스팬을 생성하고 전송하는 과정입니다. OpenTelemetry, Zipkin 등의 오픈소스 계측 프레임워크에서 SDK를 제공합니다. 예를 들어, Java에서는 Java Agent Auto Instrumentation을 사용할 수 있습니다.

#### 2. Pipeline
애플리케이션에서 생성된 스팬을 버퍼링하고 백엔드로 전송하는 과정을 구성합니다. 분산 추적 도구는 여러 소스에서 동시에 텔레메트리 데이터를 수신하므로, 애플리케이션 확장에 따라 도구에서 사용하는 리소스를 최적화해야 합니다. 이를 위해 추적 파이프라인은 데이터 버퍼링, 추적 일괄 처리, 스토리지 백엔드로의 후속 라우팅과 같은 기능을 수행합니다.

#### 3. Backend Tempo
수집된 추적은 나중에 시각화 및 분석을 위해 저장해야 합니다. 이 과정에서 너무 많은 리소스를 소비하지 않고 애플리케이션과 함께 확장할 수 있는 스토리지 백엔드가 필요합니다. 인메모리 데이터베이스에 의존하는 대신, Amazon S3, Google Cloud Storage, Microsoft Azure Blob Storage와 같은 널리 사용되는 객체 스토리지 솔루션을 추적 스토리지 백엔드로 사용합니다.

### Trace란 무엇인가?
트레이스는 트리 형태로 구성된 원격 분석 데이터로, 여러 개의 스팬으로 이루어집니다. 트레이스는 루트 스팬이 있고, 여기에서부터 여러 자식 스팬들이 뻗어나옵니다.

#### Span이란?
스팬은 시스템 내에서 서비스에 의해 수행되는 단일 작업 단위입니다.

### Span의 주요 Component
- **name**: span name
- **duration**: 시작 시간과 종료 시간
- **status**: ok, error, unset 등 otel span status를 따름
- **kind**: server, client, producer, consumer, internal, unspecified

### 스팬의 속성 (Attributes)
스팬에는 추가적인 메타데이터를 저장할 수 있으며, 키-값 형식으로 저장됩니다.
1. **Span Attributes**: 특정 작업과 관련된 정보를 저장
2. **Resource Attributes**: 스팬을 생성한 엔터티의 정보 저장
3. **Event Attributes**: 특정 시간에 발생한 이벤트 저장
4. **Link Attributes**: 다른 스팬과의 관계를 생성하는 링크 역할 수행

### Traces and Telemetry
프로파일, 메트릭, 트레이스, 로그 이 4가지의 관계는 매우 중요합니다.

#### 메트릭 (Metrics)
시스템의 상태를 한눈에 파악할 수 있도록 제공되는 숫자 기반 데이터로, 경보의 기초가 되는 데이터입니다.

#### 로그 (Logs)
개별 프로세스에서 발생한 활동을 기록하는 감사 로그로, 애플리케이션 내부에서 어떤 일이 일어나고 있는지 자세한 정보를 제공합니다.

#### 트레이스 (Traces)
데이터가 흐르는 경로에서 각 단계별로 발생하는 이벤트를 그래픽적으로 표현하며, 각 단계가 완료되는데 걸리는 시간을 시각적으로 분석할 수 있습니다.

#### 프로파일링 (Profiles)
특정 시점에서 애플리케이션이 자원을 어떻게 사용하고 있는지 확인할 수 있습니다.

#### 메트릭과 로그의 한계
메트릭과 로그만으로는 서비스 간의 상호작용과 의존성을 파악하기 어려우며, 트레이스는 서비스 간 관계를 시각적으로 표현하여 문제 파악을 쉽게 해줍니다.

### Tempo Configuration

#### Distributor
여러 형식의 스팬을 수락하고, 트레이스 ID를 기반으로 해시링을 사용하여 Ingesters로 라우팅합니다.

#### Ingester
수신된 트레이스를 블록 단위로 저장하고, Bloom filter 및 인덱스를 생성한 후 백엔드에 저장합니다.

#### Query Frontend
검색 공간을 sharding하여 여러 개의 작은 요청으로 나누고 병렬 처리합니다.

#### Querier
특정 트레이스 ID를 조회하고 필요하면 백엔드에서 데이터를 조회합니다.

### 컴팩터 (Compactor)
백엔드 스토리지에서 블록을 가져와 병합하여 전체 블록 수를 줄입니다.

## Key Points
- Tempo는 오브젝트 스토리지를 활용하여 고확장성 분산 추적을 지원합니다.
- 클라이언트 계측, 파이프라인, 백엔드, 시각화로 구성된 추적 파이프라인을 제공합니다.
- 트레이스는 여러 스팬으로 구성되며, 각 스팬은 시스템 내 단일 작업 단위를 나타냅니다.
- 메트릭, 로그, 트레이스, 프로파일링은 시스템 상태를 이해하는 데 중요한 역할을 합니다.

## Follow-up Questions / Discussion Points
1. TSDB 대신 객체 스토리지를 사용하는 이유는 무엇이며, 그로 인해 얻는 이점은 무엇인가요?
2. 로그 파일과 트레이스 파일이 데이터 스토리지에 미치는 영향은 무엇인가요?
3. Tempo의 추적 파이프라인이 애플리케이션 확장에 어떻게 기여하나요?
4. 스팬의 속성은 어떻게 시스템의 성능 분석에 기여할 수 있나요?
5. 메트릭과 로그의 한계를 극복하기 위해 트레이스를 어떻게 활용할 수 있을까요?