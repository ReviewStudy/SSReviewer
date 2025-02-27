기본적으로 Datadog Agent는 DaemonSet으로 실행이된다!

즉 전에 쿠버네테스 강의에서 봤듯이 각 노드에서 하나씩 작동을 하고 있는 것이다
![](https://i.imgur.com/eFSl1IM.png)

여기서 저 모니터링 솔루션 처럼 말이다. 

---

Datadog Agent의 변화를 보았을때 **클러스터 모니터링**이 중요하다고 볼 수 있다
#### 과거 클러스터 모니터링이 존재하지 않았을때는
- 각 워커 노드는 모두 데이터독 에이전트를 실행하여 데이터를 다음의 소스로 부터 모았다
	- Kubelet: 노드에 작업을 생성하는 로컬 데몬
	- Controlplane: api server, controller manager, etcd로 구성됨

![출처: https://www.datadoghq.com/blog/datadog-cluster-agent/?_gl=1*1aif6uh*_gcl_aw*R0NMLjE3NDA0NjA0NjcuQ2p3S0NBaUF6dkM5QmhBREVpd0FFaHRsTjV2TVhwTl9OVER5dDhySFdxMHA3ZjhDbzRHY1ZtOFNOaXI0QVlGZFpPZHJUeEtrV21HRmlSb0N5V0VRQXZEX0J3RQ..*_gcl_au*NzY5OTQzMzE5LjE3MzYyNDcwMjc.*_ga*Mjg4NTA2MjEyLjE3NDA0NjA0Njc.*_ga_KN80RDFSQK*MTc0MDQ2ODk1OS4yLjEuMTc0MDQ3MTA5My4wLjAuMTU3NTMwNTE5Mw..*_fplc*eE1FNzVlSzV6RXJtMW9NJTJGQVJJZkQ3aXpqcW43cjFtUFdud3RueVRvTjR5Slc1QlZiTVB0NGpDcjM4dmoyeUhLNk84ZGlEU3FNeTlRMURQd1RkdXMzdVQ2V0xYNlJSOFFyYm5UVHdYUnUlMkJxY1lKOHY4Y0JxeVo5OHhPWExTQSUzRCUzRA..](https://i.imgur.com/6V9UpqE.png)

Datadog Agent는 
- 각 워커 노드의 **kubelet을 모니터링**하여 **컨테이너 상태 및 스케줄링 이슈등 추적 시스템 데이터를 수집**하며, 실행 중인 애플리케이션을 자동 감지해 모니터링 하였다.
- **Kubernetes API 서버**에서 **클러스터 레벨 데이터를 수집하여 클러스터의 동작을 분석**하고, **서비스-메트릭 간 매핑 및 태깅을 수행**한다. 

> **클러스터 레벨 데이터의 중요 요소**
> 1. 클러스터 상태: 노드 상태, 노드 리소스 사용량, 노드 이벤트
> 2. Deployment, StatefulSet, DaemonSet상태
> 3. Kubernetes 오브젝트의 메타데이터: Namespaces, Services, ConfigMaps
> 4. 클러스터 네트워크, 서비스 디스커버리 등

> **그런데 여기서 왜 노드기반 에이전트가 클러스터 레벨 데이터가 필요한가?**
> - 클러스터 전체의 맥락을 이해하기 위해서 수집해야한다
> 1. 트레이싱 및 메트릭 태깅
> - APM을 수집할때 어떠한 네임 스페이스, 서비스, 워크로드에서 실행되는지 파악해야한다
> - 서비스-POD이 1:1대응이 아니기 때문에 각 파드가 속한 서비스 정보를 가져와서 매핑해야한다
> - 즉 위에서 적은 클러스터 레벨 데이터들이 필요로 하다

하지만 각 노드 에이전트가 매번 API SERVER에 요청을 보내고 응답을 받는것은 API Server에 큰 부하를 주기 때문에 이를 줄이기 위해서 데이터독 클러스터 에이전트를 도입한다

---
### 데이터독 클러스터 에이전트
![](https://i.imgur.com/ffne1eM.png)
- 중앙 제어 접근을 위해서 만들게 되었다
- 크고 몇천개의 노드가 있는 환경에서는 매우 이득이다.
- API Server의 부하를 매우 줄여줌

Cluster Agent는 API 서버와 Node-based Agents 사이에서 **프록시 역할**을 합니다.
이를 통해 **API 서버에 직접적인 부하를 줄일 수** 있을 뿐만 아니라, 
- **노드 기반 에이전트는 노드 수준의 데이터 수집에 집중**
- **클러스터 에이전트는 컨트롤 플레인에서 클러스터 수준 데이터를 수집**

클러스터 에이전트는 클러스터 수준 메타데이터를 노드 기반 에이전트에 전달하여, 각 노드에서 수집한 메트릭이 클러스터 전반에서 일관된 tag로 보강될 수 있도록 함

또한, 노드 기반 에이전트가 더 이상 API 서버에서 직접 데이터를 가져올 필요가 없어지므로, RBAC(Role-Based Access Control) 권한을 kubelet에서 메트릭과 메타데이터만 읽을 수 있도록 제한

---
### 설치 방법

가장 권장되는 설치방법인 Datadog Operator를 사용해서 설치를 해보겠다

먼저 쿠버네테스 오퍼레이터에 대해서 알아보자
#### What is Kubernetes Operator?
![](https://i.imgur.com/Yq9lKGd.png)

- **애플리케이션과 그 구성 요소를 관리하기 위해 커스텀 리소스를 사용하는 쿠버네티스의 확장 기능**
- 오퍼레이터는 **[[제어 루프(Control Loop)]]로 알려진 쿠버네티스 원칙을 따른다.**
- 쿠버네티스에서 컨트롤러는 **클러스터의 현재 상태(Actual State)와 원하는 상태(Desired State)를 지속적으로 감시하는 무한 루프**이다.
- 이 두 상태가 다를 경우, 컨트롤러는 현재 상태를 원하는 상태에 가깝게 만들기 위해 변경 작업을 수행한다.
- 오퍼레이터는 **애플리케이션의 상태를 검증하거나 애플리케이션의 구성 및 상태를 보고하는 것과 같은 복잡한 작업을 단순화하는 데 사용할 수 있다.**

#### What is the Datadog Operator?
- Datadog 오퍼레이터는 쿠버네티스 환경에서 Datadog Agent를 배포하고 구성할 수 있도록 해주는 Kubernetes 오퍼레이터이다.
- Datadog 오퍼레이터를 사용하면, 하나의 커스텀 리소스 정의(CRD)를 통해 노드 기반 에이전트(Node-based Agent), Datadog 클러스터 에이전트(Cluster Agent), 클러스터 체크 러너(Cluster Check Runners)를 배포할 수 있다.
	
Datadog 오퍼레이터의 주요 기능:
- Datadog Agent의 구성 설정을 검증
- 모든 Datadog Agent를 최신 구성으로 유지
- Datadog Agent 리소스를 생성 및 업데이트하는 작업을 자동화
- 오퍼레이터의 CRD 상태를 통해 Datadog Agent의 구성 상태를 보고

라고 한다.

> **커스텀 리소스란?**
> 쿠버네티스 API의 익스텐션 | 쿠버네티스(Kubernetes)를 확장할 수 있도록 사용자가 직접 정의하는 리소스
> **기본적으로 쿠버네티스에 포함되지 않은 새로운 API 객체**를 만들고 관리할 수 있도록 해줍니다.
> 기본 제공 리소스 VS 커스텀 리소스
> - 기본제공 리소스: Pod, Service, Deployment, ConfigMap 등등
> - 커스텀 리소스: 기본 쿠버네테스 설치에는 없지만 운영환경에 맞게 확장 가능

일반적인 HELM차트나 DaemonSet과 다른점은
- 향후 개선 사항에 맞게 더욱 유연하게 구성이 되어있다
- Kubernetes API가 일등급 리소스로 처리한다
- 수동으로 DaemonSet을 구성하면 오류가 발생할 가능성이 높아진다

단순하게 말한다면 특정 어플리케이션을 쿠버네테스에서 자동으로 관리하는 컨트롤러라고 볼 수 있다.
즉 사용자가 직접 해야하는 운영작업을 자동화하는 역할을 한다고 할 수 있다

이러한 오퍼레이터를 사용하면 자동 배포, 스케일링, 백업등을 자동화 할 수 있다는것이다

#### 전체적인 설치 프로세스
![](https://i.imgur.com/D3bWuDz.png)

1. Datadog Operator 설치
```
helm repo add datadog https://helm.datadoghq.com
helm install my-datadog-operator datadog/datadog-operator
```

2. API 및 앱 키로 Kubernetes Secret 생성
```
kubectl create secret generic datadog-secret --from-literal api-key=<DATADOG_API_KEY> --from-literal app-key=<DATADOG_APP_KEY>
```

3. Datadog Agent 배포 설정 사양을 사용하여 `datadog-agent.yaml` 파일을 만듭니다. 
```
kind: DatadogAgent
apiVersion: datadoghq.com/v2alpha1
metadata:
  name: datadog
spec:
  global:
    site: us5.datadoghq.com
    credentials:
      apiSecret:
        secretName: datadog-secret
        keyName: api-key
      appSecret:
        secretName: datadog-secret
        keyName: app-key
  features:
    apm:
      instrumentation:
        enabledNamespaces:
          - default
        enabled: true
    logCollection:
      enabled: true
      containerCollectAll: true
    remoteConfiguration:
      enabled: true
  override:
    clusterAgent:
      image:
        name: gcr.io/datadoghq/cluster-agent:latest
    nodeAgent:
      image:
        name: gcr.io/datadoghq/agent:latest
```

4. 설정으로 Datadog Agent 배포
```
kubectl apply -f /path/to/your/datadog-agent.yaml
```

다음과 같이 배포를 하면

![](https://i.imgur.com/o0eEFoL.png)

다음과 같이 datadog operator, cluster agent가 워커 노드에 배포가 되었고 datadog-agent가 각 노드들에 배포가 되었다. 

![](https://i.imgur.com/ZnFY5ut.png)

서비스를 보면 다음과 같은데 또 이상한 datadog-admission-controller도 존재를 한다 이건다 무엇인가?

#### Kubernetes Admission Controller?
- **Kubernetes Admission Controller(어드미션 컨트롤러)**는 **쿠버네티스 클러스터의 사용을 규제하고 정책을 적용하는 플러그인**이다.
- **API 요청이 인증된 후 이를 가로채고, 요청 객체를 변경하거나 거부할 수 있는 게이트키퍼(Gatekeeper) 역할**

Admission Controller의 역할

Admission Controller는 일반적으로 검증(Validation), 변경(Mutation) 또는 두 가지 모두를 수행
- **Mutating Controller(변경 컨트롤러)** → 요청을 승인하면서 관련 객체를 변경할 수 있음.
- **Validating Controller(검증 컨트롤러)** → 요청을 승인할지 여부만 판단하며, 요청 객체를 변경할 수 없음.

쿠버네티스에는 기본적으로 30개 이상의 Admission Controller가 포함되어 있으며, 그중에서 중요한 두 가지가 있음:
1.	ValidatingAdmissionWebhooks → 요청을 검증하는 역할 수행.
2.	MutatingAdmissionWebhooks → 요청을 변경하는 역할 수행.

이 두 컨트롤러는 자체적으로 결정 로직을 실행하지 않으며, 대신 쿠버네티스 클러스터 내부에서 실행 중인 웹훅(Webhook) 서비스로 요청을 위임하여 실행한다.
#### Datadog Admission Controller
- 주요 기능은 **애플리케이션 Pod을 자동으로 구성하도록 돕는 것**이다.
- **Mutating Admission Controller** 타입이며, Pod의 설정을 변경(Mutation)할 수 있다.

- 환경 변수(`DD_AGENT_HOST`, `DD_TRACE_AGENT_URL` 및 `DD_ENTITY_ID`)를 삽입하여 사용자의 애플리케이션 컨테이너에 DogStatsD 및 APM 추적기 라이브러리를 설정
- 애플리케이션 레이블의 Datadog 표준 태그(`env`, `service`, `version`)를 컨테이너 환경 변수에 삽입합니다

즉 Pod의 설정을 자동으로 변경하여 모니터링을 쉽게 할 수 있도록 한다

![](https://i.imgur.com/gVKK54m.png)

이렇게 로그도 잘나오고 

![](https://i.imgur.com/l3aQkmS.png)


APM도 잘 나옵니다.

![](https://i.imgur.com/ZGAQOmV.png)


근데 APM - 로그 연결이 안될까해서 

트레이스 로그 메트릭등을 어떻게 연결을 하는지 알아보려고 한다

먼저 로그와 트레이스를 어떻게 같이 묶는가?
https://docs.datadoghq.com/ko/tracing/other_telemetry/connect_logs_and_traces/go/

수동 삽입도 있지만 뭐 귀찮으니 Logrus 로그에 로그를 넣으면 로그와 스팬을 연결할 수 있다고 합니다
중요한 부분에서는 

![](https://i.imgur.com/WO113bc.png)

이렇게 

하면 된다고 합니다!