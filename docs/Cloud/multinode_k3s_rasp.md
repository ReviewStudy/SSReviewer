# K3S Raspberry Pi MultiNode Setup

#K3S #RaspberryPi #Cluster #Kubernetes #Setup

**Summary**: This document provides a step-by-step guide on setting up a K3S cluster on Raspberry Pi, including server and agent node installation, configuration, and node registration.

## K3S 설치 및 동작 방식

1. **cgroup 및 메모리 설정**
   ```
   sudo nano /boot/firmware/cmdline.txt
   ```
   - cgroup과 메모리 설정을 수정합니다.

2. **K3S 서버 노드 설치**
   ```
   curl -sfL https://get.k3s.io | sh -
   ```

3. **K3S 에이전트 노드 설치**
   ```
   curl -sfL https://get.k3s.io | K3S_URL=https://{마스터노드IP}:6443 K3S_TOKEN={마스터노드 토큰} K3S_NODE_NAME={NODE이름|다른노드와는 이름이 달라야함} sh -
   ```
   - **K3S_URL**: K3S 서버 API 주소
   - **K3S_TOKEN**: 마스터 노드에서 확인한 토큰
     ```
     sudo cat /var/lib/rancher/k3s/server/node-token
     ```
   - **K3S_NODE_NAME**: 워커 노드의 이름

   ![](https://i.imgur.com/Q1iOmfO.png)

   - KUBECONFIG 설정이 필요합니다. `/etc/rancher/k3s/k3s.yaml` 파일을 사용하여 클러스터에 접근합니다.

4. **k3sconfig 설정**
   ```
   sudo cat /etc/rancher/k3s/k3s.yaml
   ```
   - 인증 정보를 워커 노드로 전송하고 설정합니다.
   ```
   scp -r /etc/rancher/k3s/k3s.yaml user@workernode:/home/user/k3s.yaml
   export KUBECONFIG=/home/user/k3s.yaml
   ```

5. **설정 확인**
   ```
   kubectl get nodes
   kubectl get pods -A
   ```

## K3S 노드 구성 및 등록 방식

- **서버 노드(K3S 서버)**: `k3s server` 실행, 컨트롤 플레인 및 API 서버 관리
- **에이전트 노드(K3S 에이전트)**: `k3s agent` 실행, 컨테이너 실행 및 관리

> 서버와 에이전트는 모두 kubelet, 컨테이너 런타임, CNI 플러그인을 실행합니다.

### 임베디드 DB가 있는 단일 서버 설정

- 단일 노드에서 컨트롤 플레인이 동작하며, K3S 에이전트가 이를 참조합니다.
- **단점**: K3S 서버가 중단되면 클러스터 관리가 불가능합니다.

### Embedded DB를 이용한 Multiple K3S 서버 구성

- 3개 이상의 홀수 대수의 K3S 서버를 구성하여 etcd를 사용합니다.
- API 서버 및 컨트롤 플레인을 분산시켜 자동 Failover를 지원합니다.

### External DB를 이용한 Multiple K3S 서버 구성

- 외부 데이터베이스(MySQL, PostgreSQL, 외부 etcd)를 활용하여 클러스터를 구성합니다.
- 컨트롤 플레인 노드의 확장성이 뛰어나고 고가용성을 보장합니다.

## K3S 에이전트 노드 등록 방식

### 에이전트 노드 등록 과정

- K3S 에이전트는 `k3s agent` 프로세스를 실행하며, 웹소켓 연결을 통해 서버와 통신합니다.
- 클라이언트 측 로드밸런서를 이용하여 클러스터의 모든 서버와 안정적인 연결을 유지합니다.

### 노드 인증 및 보안

- 에이전트는 노드 클러스터 시크릿 및 개별 비밀번호를 사용하여 서버에 등록됩니다.
- 비밀번호는 `/etc/rancher/node/password`에 저장되며, 노드 ID 무결성을 보호합니다.
- 서버는 개별 노드의 비밀번호를 Kubernetes Secret으로 저장하며, 인증 시 동일한 비밀번호를 요구합니다.

### 노드 인증 및 삭제 방법

- 기존 이름으로 노드를 재가입하려면, 기존 노드를 삭제해야 합니다.
  ```
  kubectl delete node <노드이름>
  ```
- 이후, 에이전트를 다시 클러스터에 등록합니다.

### 요약

- K3S는 각 에이전트 노드가 올바른 서버에 등록되었는지 확인하기 위해 개별 비밀번호를 사용합니다.
- 비밀번호는 `/etc/rancher/node/password`에 저장되며, 노드가 클러스터에 참여할 때 서버에 이를 제출하여 인증합니다.
- 서버는 이 비밀번호를 kube-system 네임스페이스의 Kubernetes Secret으로 저장하여 이후 인증 시 동일한 값인지 검증합니다.
- 이를 통해 노드의 ID가 위조되거나 임의의 노드가 클러스터에 무단으로 가입하는 것을 방지합니다.

![](https://i.imgur.com/ZuA7blA.png)

## Key Points

- K3S 설치 및 설정 방법
- K3S 서버 및 에이전트 노드의 역할
- 임베디드 및 외부 DB를 이용한 클러스터 구성
- 에이전트 노드의 등록 및 인증 과정

## Follow-up Questions / Discussion Points

1. K3S 클러스터에서 임베디드 DB와 외부 DB의 장단점은 무엇인가요?
2. K3S 에이전트 노드의 인증 과정에서 보안이 중요한 이유는 무엇인가요?
3. K3S 클러스터의 고가용성을 보장하기 위한 방법은 무엇인가요?
4. K3S 서버가 중단되었을 때 클러스터 관리가 불가능한 이유는 무엇인가요?
5. K3S 클러스터에서 노드의 ID 무결성을 보호하는 방법은 무엇인가요?