
### Authentication: Who can access
- Files – Username and Passwords
- Files – Username and Tokens
- Certificates
- External Authentication providers - LDAP
- Service Accounts
### Authorization: What they can do
- RBAC Authorization
- ABAC Authorization
- Node Authorization
- Webhook Mode

Securing Cluster with Authentication 
- Admins
- Developers
- Application
- EndUsers

Accounts
```
kubectl create serviceaccount sa1
```
- account를 만들 수 있따
모든 User access는 kube-apiserver가 관리한다
모든 요청은 kube-apiserver -> authenticate -> process

### How authenticate?
- Static password file
	- user-details.csv에 유저, 패스워드를 저장
	- kube-apiserver.service에서 --basic-auth-file=user-details.csv를 추가한다

```
curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"
```
- Static token file
	- user-token-detilas.csv로 저장
	- --token-auth-file=user-details.csv 를 지정하여 한다

```
curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer KpjCVbI7rCFAHYPkBzRb7gu1cUc4B"
```
- certifcates
- identity services

> 보안적으로 좀 위험하다!

### TLS Certificates
TLS 인증서는 두 당사자가 **안전하게 통신할 수 있도록 신뢰를 보장하는 역할**을 한다. 예를 들어, 사용자가 은행 웹사이트에 접속하면 TLS 인증서는 해당 웹 서버가 실제로 신뢰할 수 있는 서버인지 확인하고, 사용자와 서버 간의 통신이 암호화되도록 보장한다. 

1. 인증서 없이 PlainText로 데이터를 주고 받을 경우
- 해커가 바로 탈취 가능

2. (대칭키)방지하기 위해 암호화키로 Encrypt 해서 송신을 하면 안전
	1. 서버가 해독하기 위해서 복호화키를 같이 전달을 해야하고
	2. 하지만 복호화키를 같이보내면 결국 해커가 탈취

3. (비대칭키) 다른 Pair of Key를 사용하여 암호화 복호화를 수행하게 된다
	1. 공개키로 암호화 - 오직 개인키로만 복호화할 수 있도록 반대로도 가능 SSH에 적용
	2. ssh-keygen을 통해서 생성을 한다고 할때 , 서버의 접근 권한을 보호하기 위해서 문을 공개키로 잠그고, 사용자만 개인키를 가지고 있어서 문을 연다

4. (대칭키+비대칭키) 서버가 공개키 + TLS 인증서를 사용자에게 제공 -> 사용자는 대칭키 생성 -> 공개키로 대칭키 암호화 -> 서버는 개인키로 복호화해서 데이터 주고받기

5. 하지만 만약 공격자가 가짜 사이트 + 자체적인 공개키+개인키 생성 -> HTTPS 설정시 정보를 유출
	1. **Self-Signed Certificate(자가 서명 인증서)를 사용**하면, 사용자가 속을 가능성이 높아
	2. 그렇기 때문에 브라우저는 모든 인증서를 검증하며 신뢰할 수 없다면 경고 메세지 표시

![](https://i.imgur.com/eIBZJmS.png)

but not in your application, internal, you can host your own private CAs

6. 여기서 CA라는 개념이 등장하게 되는데 웹서버가 이러한 HTTPS를 적용하기 위해서는 사용자가 웹서버의 공개키를 신뢰할 수 있어야한다! -> 인증 기관(CA, Certificate Authority)에게 CSR(Certificate Signing Request, 인증서 서명 요청)을 보낸다 -> CA는 자신의 개인 키를 사용해 이 요청을 서명한 후, 서명된 인증서를 서버에게 반환 -> 사용자는 모든 CA의 공개 키를 기본적으로 가지고 있으므로, 웹 서버가 제공한 인증서가 신뢰할 수 있는 CA에 의해 서명되었는지 확인
7. 사용자가 웹서버 접속 -> 서버가 인증서 + 서버의 공개키 보냄 -> 브라우저는 CA의 공개키를 이용해 서버 인증서 검증 -> 서버의 공개키 가져온다. -> 브라우저는 대칭키를 생성, 키를 공개키로 암호화후 서버에 전달 -> 복호화후 대칭키 획득
8. 서버의 신원은 확인을 하였는데 사용자의 신원은? -> 피싱, 비밀번호 유출로 접근 가능 -> 고로 Client Certificate요구할 수 있다! -> 클라이언트도 공개키 +개인키 생성하여 인증서를 발급

> 일반적인 웹 사이트에서는 이런 클라이언트 인증서 방식이 잘 사용되지 않는다. 보통은 ID와 비밀번호를 입력하는 방식으로 사용자 인증을 처리하지만, **클라이언트 인증서는 기업 환경에서 보안이 중요한 내부 시스템에서 사용**되기도 한다. 일반 사용자는 직접 클라이언트 인증서를 생성할 일이 거의 없으며, 만약 필요할 경우 기업에서 이를 자동화하여 사용자에게 제공하는 방식으로 운영된다.

이러한 전체적인 보안 인프라, 즉 CA, 서버, 사용자, 키 생성 및 인증서 관리 프로세스 전체를 포함하는 시스템을 공개 키 기반 구조(PKI, Public Key Infrastructure)라고 한다. PKI는 네트워크 보안의 핵심 요소로, HTTPS, SSH, VPN 등 다양한 분야에서 활용되며, 우리가 인터넷에서 안전하게 통신할 수 있도록 돕는 중요한 기술이다.

### 쿠버네테스에서 TLS
![](https://i.imgur.com/VB0imEo.png)

root certificate : **Kubernetes는 기본적으로 자체 CA(Certificate Authority) 역할을 수행**/ 자동으로 ca.crt, ca.key를 생성하고 api 서버와 모든 인증서를 여기서 생성한다
client certificate
server certificate 

보통 Private key는 key라는 단어가 있고 없으면 certificate public key인 경우가 대부분이다

쿠버네테스에서는 모든 컴포넌트끼리도 secure
api server- scheduelr도 tls connection으로 연결되어야한다

고로 server certificates, client certificates 가 필요하다

kube-apiserver - api server exposes https services

![](https://i.imgur.com/F7aoNiR.png)

- etcd server와 통신하는 유일한 서버이므로 apiserver.crt, apiserver.key로 하면 되는데
- 추가적으로 apiserver-etcd-client.crt, apiserver-etcd-client.key로 해도된다
- 이렇게 많은 certificate를 다 CA가 sign 해야한다. 
- CA.crt, CA.key로 다 한다

### how to create certificate
```
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj \
	"/CNkube-admin/OU=system.:masters" -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey
```

1.	개인 키 생성 (admin.key)
```
openssl genrsa -out admin.key 2048
```
- 개인 키는 자신만이 소유해야 하며, 공개되면 보안이 취약해짐.

2. CSR(인증서 서명 요청) 생성 (admin.csr)
```
openssl req -new -key admin.key -subj "/CN=kube-admin/OU=system:masters" -out admin.csr
```
- CSR은 인증서를 발급받기 위해 CA에 제출하는 요청 파일.

3.	CA(인증 기관)에서 인증서 서명 및 발급 (admin.crt)
```
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out admin.crt -days 365
```
- CA가 CSR을 확인하고, 해당 키가 신뢰할 수 있도록 인증서를 발급해줌.

개인 키(admin.key): 암호화 및 복호화를 위한 비밀 키.
CSR(admin.csr): CA에게 인증서 발급을 요청하는 파일.
CA 서명된 인증서(admin.crt): 최종적으로 발급된 인증서로, 이 인증서를 사용해 보안 통신을 수행.

KUBE API SERVER
```
ExecStart=/usr/local/bin/kube-apiserver \\
--advertise-address=${INTERNAL_IP} \\
--allow-privileged=true \\
--apiserver-count=3 \\
--authorization-mode=Node,RBAC \\
--bind-address=0.0.0.0 \\
--enable-swagger-ui=true \\
--etcd-cafile=/var/lib/kubernetes/ca.pem \\
--etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \\
--etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \\
--etcd-servers=https://127.0.0.1:2379 \\
--event-ttl=1h \\
--kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
--kubelet-client-certificate=/var/lib/kubernetes/apiserver-etcd-client.crt \\
--kubelet-client-key=/var/lib/kubernetes/apiserver-etcd-client.key \\
--kubelet-https=true \\
--runtime-config=api/all \\
--service-account-key-file=/var/lib/kubernetes/service-account.pem \\
--service-cluster-ip-range=10.32.0.0/24 \\
--service-node-port-range=30000-32767 \\
--client-ca-file=/var/lib/kubernetes/ca.pem \\
--tls-cert-file=/var/lib/kubernetes/apiserver.crt \\
--tls-private-key-file=/var/lib/kubernetes/apiserver.key \\
--v=2
```

KUBELET 의 이름의 노드 이름을 따른다
![](https://i.imgur.com/EagoOoe.png)

### View Certificates
where to view at the right info

#### kubeadm
![](https://i.imgur.com/aDyFEhv.png)

### Certificates
- manage certificate request and rotate
- certificates api -> send certificatesigningrequest object
- instead of logging onto the master node and signing the certificate by himself,
- **he creates a Kubernetes API object called CertificateSigningRequest.**

![](https://i.imgur.com/7xz1lRD.png)


Reviews
ApproveRequests
Share certs to users

```
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

![](https://i.imgur.com/n7FwNtd.png)
- REQUEST에는 cat jane.csr | base64한 값을 넣는다

- kubectl get csr 
- kubectl certificate approve jane을 통해서 인증한다

![](https://i.imgur.com/z0FQXmB.png)

Controller Manager - CSR-Approving, CSR_Signing

![](https://i.imgur.com/avRcebl.png)

k3s는 여기서 확인할 수 있고 
k8s /etc/kubernetes/pki에 존재한다고합니다

apiserver의 client들 (controller manager. kubelet등)이 사용하는 Client 인증서는 /etc/kubernetes/ 에 Kubeconfig 형태로 저장된다.

Kubeconfig에는 api server의 주소, CA정보 및 Client Certificate와 Key와 같은 필요한 모든 정보가 포함된다.

아래는 control node에 설치된 client들이 사용하는 kubeconfig 파일이다.
![](https://i.imgur.com/tnACvg5.png)

**kubectl이 클러스터와 통신할 때 필요한 정보**를 담고 있다.

이 파일에는 **클러스터 정보, 사용자 인증 정보, 컨텍스트(context) 설정** 등이 포함

```
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    server: https://192.168.1.100:6443
    certificate-authority: /etc/kubernetes/pki/ca.crt

users:
- name: my-user
  user:
    client-certificate: /etc/kubernetes/pki/admin.crt
    client-key: /etc/kubernetes/pki/admin.key

contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: my-user
    namespace: default

current-context: my-context
```

~/.kube/config 파일은 kubectl이 Kubernetes API 서버와 안전하게 통신하기 위한 설정 파일이다.
1.	clusters: API 서버 주소 및 인증서 정보
2.	users: 사용자 인증 정보
3.	contexts: 클러스터, 사용자, 네임스페이스 조합
4.	current-context: 현재 활성 컨텍스트

https://devocean.sk.com/blog/techBoardDetail.do?ID=165722

Kubernetes의 CA를 어떻게 믿을 수 있는가?
- 자체 CA가 사인한 인증서를 어떻게 믿을 수 있을까?

### Kubeconfig
![](https://i.imgur.com/8mIegd4.png)

보통 - .kube/config폴더 안에 존재한다
export KUBECONFIG를 하면 된다

