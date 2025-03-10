# Kubeconfig
![](https://i.imgur.com/M4DMHJp.png)

다음과 같이 매번 이러한 tls 옵션을 입력하는것은 매우 귀찮다 그렇기 때문에 

그렇기 때문에 우리는 KubeConfig File에 이를 저장해두고 매번 이용을 하게 된다

![](https://i.imgur.com/8mIegd4.png)

하지만 또 쿠버네테스에는 클러스터와 다양한 유저들이 존재하고 이러한 옵션들을 연결시쳐주는 존재로 contexts를 사용한다. 
각 유저의 암호 정보와 클러스터의 암호 정보를 저장하고 이를 컨테스트로 연결해서 사용하는 방식이다

current-context옵션을 통해서 현재 컨텍스트를 지정할 수 있따

보통 - .kube/config폴더 안에 존재한다
export KUBECONFIG를 하면 된다

기본 config는
```
kubectl config view
```

로 조회

새로운 파일로 조회하고 싶다면

```
kubectl config view -kubeconfig=my-custom-config
```

---
### API Groups

```
curl https://kube-master:6443/version
curl https://kube-master:6443/api/v1/pods
```
/metrics, /healthz, /version, /api, /apis, /logs

core group
- /api : namespace, pods, rc, events, nodes, binding, pv, pvc, configmaps

named group
- /apis: deployments, replicaset, statefulsets, extension , networking, storage, authentication, certificates

### kubectl proxy vs kube proxy
- kube proxy: enable dconnectivity between pods and services between nodes or pods
- kubectl proxy: actp access kube api server

---
#### Authorization
- what they can do?
- why authorization
- admins - 모든 일을 할 수 있따
- developers | testers | monitoring application/ cd application - 모든 것을 알 필요는 없기에 역할을 제한해야한다. minimum level of access to perform
- users to their naemspaces alone

#### Mechanism

#### 1. Node: 
- node authorizer: kubelte part of system nodes group
- any request from user with system node - authorized by node authorizer and granted privileges
- 노드는 시스템의 특권을 부여하여 kubelet이 api operation (read, write)를 할 수 있는 권한이다.
- **Kubernetes 노드가 수행하는 API 요청을 검증하는 특권적인 인증 메커니즘**으로. 일반 사용자(User Account)나 서비스 계정(Service Account)와 달리, **Kubernetes 노드는 클러스터 내부에서 특별한 역할을 하므로, 기본적으로 특정 API 요청이 자동으로 허용되도록 설계**
- 일반 사용자와 다른 특권적인 인증 시스템 → system:nodes 그룹의 일부로만 작동

#### 2. ABAC: 속성 기반 접근 제어(ABAC, Attribute-Based Access Control)
- json format으로, user or group에 permission attribute를 설정한다.
- ﻿> 개별 유저 or group마다 권한을 설정해주어야 하므로 관리하기가 까다롭다.
- ﻿> 새로운 유저가 들어와 수정하게 되면, kube-api를 재시작해야 한다.
- ABAC같은 경우는 권한에 대한 내용을 파일로 관리하기 때문에 권한을 변경하려면 직접 마스터 서버에 들어가서 파일을 변경하고 apiserver를 재시작해주어야 하기 때문에 관리하기 번거로운 측면이 있습니다. 
![](https://i.imgur.com/ZTHqXNJ.png)

#### 3. RBAC
- ABAC의 관리 문제를 해결하기 위해 등장한 것이 **RBAC(역할 기반 접근 제어, Role-Based Access Control)**
- 사용자가 아니라 **역할(Role)을 기반으로 권한을 부여**함.
- 특정 작업(예: Pod 생성, 삭제, 조회 등)에 대한 권한을 가진 **Role(역할)** 을 생성하고, 이를 사용자에게 할당하는 방식.
- **역할 기반이므로 관리가 훨씬 용이**하고, **보안성도 강화**됨.
	![](https://i.imgur.com/aRNr2IS.png)

Role-based access controls provide a more standard approach to managing access within the Kubernetes cluster

#### 4. Webhook
- **Webhook을 활용하여 외부 시스템과 연동**

**Webhook 기반 접근 제어**
- API 서버가 인증 요청을 받을 때, Webhook을 호출하여 **외부 정책 엔진(예: Open Policy Agent, Kyverno 등)** 에게 정책 검사를 위임할 수 있음.
- 이를 통해 **기업 내부 정책을 적용**하거나, 기존 보안 시스템과 통합하는 것이 가능함.

**대표적인 Webhook 기반 솔루션**
- **Open Policy Agent (OPA)**: 정책 기반 접근 제어를 지원하는 오픈소스 솔루션.
- **Kyverno**: Kubernetes 네이티브 정책 엔진으로, RBAC과 Webhook을 조합하여 세부적인 권한 관리를 가능하게 함.

### 5. Always Allow | Always Deny
- 모든 요청 허락 (Default)
- 모든 요청 거부

##### Kubernetes의 접근 제어 흐름
1.	Node Authorizer:
- 요청이 시스템 노드(system:node)에서 온 경우, Node Authorizer가 검토합니다.
- 승인되면 더 이상의 검사가 필요하지 않음.
2.	RBAC (또는 ABAC, Webhook):
- 일반 사용자 요청의 경우, 먼저 RBAC(Role-Based Access Control)에서 사용자가 속한 역할(Role)을 확인합니다.
- 허용된 권한이면 요청이 승인되며, 거부되면 다음 정책 엔진(예: Webhook)에서 검토할 수도 있습니다.
3.	Always Allow / Always Deny 정책:
- Kubernetes의 기본 정책은 모든 요청을 허용(Allow) 하지만, 필요에 따라 특정 요청을 전부 차단하는 Always Deny 정책을 적용할 수도 있습니다.

### RBAC
#### 1. Role 생성
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: my-namespace  # 특정 네임스페이스에 적용하려면 추가
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
  resourceNames: ["blue", "orange"]  # 특정 Pod에만 적용
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
```
- **Role은 특정 네임스페이스에서만 유효**

그런 다음 link user to role

#### 2. Role Binding
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
  namespace: my-namespace
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

namespace 추가하고싶으면 metadata에 추가해라

### 3. 역할 확인
내 역할이 있는지 확인하고 싶다면

```
kubectl auth can-i create deployments
```

관리자라면 사용자인척 테스트할 수 있다

```
kubectl auth can-i create deployments --as dev-user --namespace test
```

### ClusterRole
- role, rolebinding은 namespace안에 만들어진다
- node는? cluster-wide, cluster-scoped 리소스이다. 
- 리소스는 namespaces or cluster-scoped 로 나누어진다

![](https://i.imgur.com/bJdVj6P.png)

```
kubectl api-resources --namespaced=true
```

clusterroles: cluster-scoped resource에 접근권한을 주는것이다
- cluster admin은 이제 delete role, create node등 하게 해주는것이고
- storage admin - pv, pvcs 를 관련하여 리소스 권한을 준다

![](https://i.imgur.com/9Tukrg2.png)

cluster scoped로도 pod, 이나 이러한 role을 줄수 있는데 이러면 모든 namespace, cluster에 있는 pod에 권한을 주는것이다

## ServiceAccounts
- Kubernetes에서는 사용자 계정(User Account)과 서비스 계정(Service Account)을 구분하여 사용 
- 사용자 계정은 사람이 직접 Kubernetes 클러스터와 상호작용하는 데 사용
- 서비스 계정은 Prometheus 모니터링, Jenkins 배포와 같은 애플리케이션이나 시스템이 Kubernetes API와 상호작용할 때 활용

Kubernetes에서 API 요청을 수행하는 역할을 하기 위해 서비스 계정을 생성할 수 있으며, 

```
kubectl create sa dashboard-sa
kubectl get sa`와 `kubectl describe sa dashboard-sa
```

서비스 계정을 활용하기 위해서는 RBAC(Role-Based Access Control)과 연동해야 하는데

Role부여 -> 서비스 계정과 연결하여 사용

> 기본적으로 서비스 계정을 생성하면 자동으로 토큰이 Secret으로 저장되며, 이를 통해 외부 애플리케이션이 해당 계정을 사용하여 Kubernetes API와 상호작용할 수 있다.

### Kubernetes 내부에서 서비스 계정 사용
Kubernetes 내부에서 실행되는 애플리케이션이 서비스 계정을 사용할 경우, 별도의 설정 없이 Pod 내부에서 자동으로 서비스 계정의 토큰을 마운트할 수 있다. 

Pod 내부에서 `/var/run/secrets/kubernetes.io/serviceaccount/` 경로를 확인하면, `ca.crt` 인증서, 현재 네임스페이스 정보, 그리고 API 접근 토큰을 확인할 수 있다. 

이를 활용하면 Pod 내에서 Kubernetes API와 직접 통신할 수 있으며, 추가적인 Secret 마운트 없이도 인증된 요청을 수행할 수 있다.

### 자동 마운트(AutoMountServiceAccountToken)와 보안 이슈
Kubernetes는 기본적으로 Pod에 서비스 계정의 토큰을 자동으로 마운트하는데, 이 토큰이 JWT(JSON Web Token) 형식으로 생성되며 **만료(expiry date)가 없다는 점에서 보안 및 확장성 문제를 야기**

=> Kubernetes v1.24 이전까지는 서비스 계정을 생성하면 자동으로 Secret이 생성되었으며, 해당 서비스 계정이 존재하는 한 JWT 토큰이 계속 유효했다.

### Kubernetes v1.24 이후의 변화
Kubernetes v1.24 이후부터는 자동 Secret 생성이 비활성화되었고, 명시적으로 토큰을 생성해야 하며, 유효 기간이 추가되었다. 

따라서 `kubectl create token dashboard-sa` 명령어를 실행하여 토큰을 생성하고 이를 화면에 출력하여 확인할 수 있다. 

이러한 변화는 보안성과 확장성을 더욱 강화하는 방향으로 진행되었으며, Pod이 생성될 때마다 새로운 토큰이 할당되고, 서비스 계정과 별개의 라이프사이클을 가지도록 설계되었다. 또한, Projected Volume을 활용하여 동적 토큰 관리가 가능해졌으며, Kubernetes API와 통신할 때 최신 토큰을 자동으로 사용할 수 있도록 지원된다.


### Private Image Pulling
```
apiVersion: vl kind: Pod metadata:
name: nginx-pod spec:
containers:
- name: nginx image:
imagePullSecrets:
- name: regered
```

### Docker Security
1. 프로세스 격리 (Process Isolation)
	•	Docker 컨테이너는 호스트와 같은 커널을 공유하며, 네임스페이스(namespace)를 활용해 격리됩니다.
	•	컨테이너 내부에서 실행된 프로세스는 자체 네임스페이스 안에서만 보이며, 호스트에서는 일반적인 프로세스로 인식됩니다.
	•	컨테이너 내부에서 보면 프로세스 ID가 1이지만, 호스트에서 보면 다른 ID를 가집니다.

2. 사용자 권한 (User Privileges)
	•	기본적으로 Docker 컨테이너 내부의 프로세스는 root 권한으로 실행됩니다.
	•	그러나 보안상 위험할 수 있으므로, --user 옵션을 사용해 비root 사용자로 실행할 수 있습니다.
	•	Dockerfile에서도 USER 지시어를 사용하여 기본 실행 사용자를 변경할 수 있습니다.

3. Root 사용자와 보안 문제
	•	컨테이너 내부의 root 사용자가 호스트의 root 사용자와 동일한 권한을 가지는 것은 아님.
	•	Docker는 Linux Capabilities를 활용하여 root 사용자의 권한을 제한함.
	•	기본적으로 컨테이너에서 호스트를 재부팅하거나, 시스템 설정을 변경하는 등의 위험한 작업을 수행할 수 없음.

4. 추가적인 권한 관리
	•	--cap-add 옵션: 특정 기능을 추가하여 더 많은 권한을 부여할 수 있음.
	•	--cap-drop 옵션: 특정 권한을 제거하여 보안을 강화할 수 있음.
	•	--privileged 옵션: 모든 권한을 부여하여 컨테이너가 호스트와 유사한 수준의 권한을 가짐 (보안상 위험할 수 있음).

### Kubernetes Security Context

```
apiVersion: v1
kind: Pod
metadat: 
	name: web-pod
spec:
	containers:
		- name
		- image
		- securityContext:
			- runAsUser:
			- capabilities:
```

### Network Policy
![iTmuvS4.png](https://i.imgur.com/iTmuvS4.png)
- Kubernetes는 기본적으로 **모든 Pod 간의 연결을 허용**하지만, 보안 강화를 위해 **Network Policy**를 사용하여 트래픽을 제한할 수 있다. 이를 통해 특정 Pod만 접근할 수 있도록 설정하고, 불필요한 네트워크 노출을 방지할 수 있다

### 1. 기본 개념

Network Policy는 **Ingress(수신 트래픽)과 Egress(송신 트래픽)를 제어**할 수 있다. 기본적으로 모든 Pod는 서로 자유롭게 통신할 수 있지만, 보안 강화를 위해 특정 Pod만 통신할 수 있도록 **Ingress 정책을 설정**하는 것이 일반적이다.

```
policyTypes:
- Ingress
ingress:
- from:
  - podSelector:
      matchLabels:
        name: api-pod
  ports:
  - protocol: TCP
    port: 3306
```

위와 같은 설정을 적용하면 **Ingress만 독립적으로 제어되며, Egress는 영향을 받지 않는다**. 즉, Pod 내부에서 외부로 나가는 요청에는 제한이 없고, 특정 Pod에서 오는 **Incoming 트래픽만 허용**된다.

### Pod 식별 및 Ingress 규칙 설정

DB Pod을 식별하기 위해 **podSelector + matchLabels** 를 사용하며, API Pod이 DB Pod에 Query를 요청해야 할 경우 **Ingress 규칙을 열어야 한다.**

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          name: prod
    ports:
    - protocol: TCP
      port: 3306
```
- `podSelector`: 특정 Pod(Label 기반)만 허용한다.
- `namespaceSelector`: 특정 네임스페이스에 속한 모든 Pod를 허용할 수 있다.
- `ports`: 허용할 포트 및 프로토콜을 지정한다.
    
**Note:** Ingress가 허용된 경우 해당 요청에 대한 응답도 자동으로 허용된다.

### 네임스페이스 기반 정책
`namespaceSelector`만 포함하면 해당 네임스페이스에 속하는 **모든 Pod의 접근이 허용**된다. 이를 통해 네임스페이스 수준에서 네트워크 정책을 적용할 수 있다.

```
namespaceSelector:
  matchLabels:
    name: prod
```

위 설정을 추가하면 `prod` 네임스페이스에 있는 모든 Pod가 DB Pod에 접근할 수 있다.

---
### 4. 외부(쿠버네테스 외부) 접근 허용

쿠버네테스 외부(예: 백업 서버)에서 접근해야 하는 경우 **ipBlock**을 사용하여 특정 IP 주소의 접근을 허용할 수 있다.

```
- ipBlock:
    cidr: 192.168.5.10/32
```

위 설정을 적용하면 `192.168.5.10` IP 주소에서 들어오는 트래픽만 허용된다. `cidr` 값에 `/32`를 붙이면 해당 단일 IP만 허용할 수 있으며, CIDR 범위를 조정하여 더 넓은 네트워크를 허용할 수도 있다.

---

![](https://i.imgur.com/88zThRm.png)

