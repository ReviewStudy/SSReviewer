# MultiNode in K3S Raspberry Pi

### K3S 설치 및 동작 방식 (https://docs.k3s.io/quick-start)

1. Edit cgroup and memory
```
sudo nano /boot/firmware/cmdline.txt

## cgroup , memory설정을 해준다
```
2. K3s Server Node 설치
```
curl -sfL https://get.k3s.io | sh -
```
3. K3S Agent Node 설치
```
curl -sfL https://get.k3s.io | K3S_URL=https://{마스터노드IP}:6443 K3S_TOKEN={마스터노드 토큰} K3S_NODE_NAME={NODE이름|다른노드와는 이름이 달라야함} sh -
```
- K3S_URL: K3s 서버(마스터) API 서버 주소 (https://<SERVER_IP>:6443)
- K3S_TOKEN: 마스터에서 확인한 node-token
	-  마스터 에서 토큰 확인
```
sudo cat /var/lib/rancher/k3s/server/node-token
```
- K3S_NODE_NAME: 워커 노드의 이름 지정

![](https://i.imgur.com/Q1iOmfO.png)

하지만 바로 동작하지 않는다 왜냐하면 KUBECONFIG를 설정하지 않았기 때문이다

> KUBECONFIG?
> /etc/rancher/k3s/k3s.yaml에 저장된 kubeconfig 파일은 쿠버네티스 클러스터에 대한 액세스를 구성하는 데 사용되는 것
> 원격에서 K3s 클러스터를 관리하려면 KUBECONFIG파일을 획득하여 사용하여야한다

1. k3sconfig
```
sudo cat /etc/rancher/k3s/k3s.yaml
```

이거를 하면 나의 인증정보가 나온느데 이것을 나의 worker node에 옮겨주고 이 워커 노드에서 이 kubeconfig를 통해서 동작하도록 해야한다 왜?

```
scp -r /etc/rancher/k3s/k3s.yaml user@workernode:/home/user/k3s.yaml
export KUBECONFIG=/home/user/k3s.yaml
```

정상적으로 설정되었는지 확인:
```
kubectl get nodes
kubectl get pods -A
```

### K3s 노드 구성 및 등록 방식
![](https://i.imgur.com/YT1YNxr.png)

- **서버 노드(K3s 서버)**: `k3s server` 실행, **컨트롤 플레인 및 API 서버 관리**
- **에이전트 노드(K3s 에이전트)**: `k3s agent` 실행, **컨테이너 실행 및 관리

> 서버와 에이전트는 모두 **kubelet, 컨테이너 런타임, CNI 플러그인**을 실행

### 임베디드 DB가 있는 단일 서버 설정
![](https://i.imgur.com/1c8WeLN.png)

- 컨트롤 플레인이 단일 노드에서 동작하며, K3s 에이전트가 이를 참조함.
- **단점:** K3s 서버가 중단되면 클러스터 관리 불가능.

## Embedded DB를 이용한 Multiple K3s server 구성
![](https://i.imgur.com/HPAh4vf.png)
- **3개 이상의 홀수 대수의 K3s 서버를 구성**하여 **etcd를 사용**.
- API 서버 및 컨트롤 플레인을 분산시켜 **자동 Failover 지원**

## External DB를 이용한 Multiple K3s server 구성
![](https://i.imgur.com/qhdn6b8.png)

- 외부 데이터베이스 (**MySQL, PostgreSQL, 외부 etcd**)를 활용하여 클러스터 구성.
- 컨트롤 플레인 노드 확장성이 뛰어나고 **고가용성(HA) 보장**.

https://devocean.sk.com/blog/techBoardDetail.do?ID=165375

#### K3s 에이전트 노드 등록 방식

##### 에이전트 노드 등록 과정
- K3s 에이전트는 k3s agent 프로세스를 실행하며, 웹소켓 연결을 통해 서버와 통신.
- 클라이언트 측 로드밸런서를 이용하여 클러스터의 모든 서버와 안정적인 연결 유지.

##### 노드 인증 및 보안
에이전트는 노드 클러스터 시크릿 및 개별 비밀번호를 사용하여 서버에 등록됨.
비밀번호는 /etc/rancher/node/password에 저장되며, 노드 ID 무결성 보호 역할을 수행함.
서버는 개별 노드의 비밀번호를 Kubernetes Secret으로 저장하며, 인증 시 동일한 비밀번호를 요구함.

##### 노드 인증 및 삭제 방법
⚠ 기존 이름으로 노드를 재가입하려면, 기존 노드를 삭제
```
kubectl delete node <노드이름>
```
이후, 에이전트를 다시 클러스터에 등록합니다.

#### 즉
- K3s는 **각 에이전트 노드가 올바른 서버에 등록되었는지 확인하기 위해** 개별 비밀번호를 사용
- 비밀번호는 /etc/rancher/node/password에 저장되며, **노드가 클러스터에 참여할 때 서버에 이를 제출하여 인증**
- 서버는 이 비밀번호를 kube-system 네임스페이스의 **Kubernetes Secret으로 저장**하여 이후 인증 시 동일한 값인지 검증
- 이를 통해 **노드의 ID가 위조되거나 임의의 노드가 클러스터에 무단으로 가입하는 것을 방지**

![](https://i.imgur.com/ZuA7blA.png)



