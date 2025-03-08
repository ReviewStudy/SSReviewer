# KAKAO - Kubernetes를 이용한 서비스 무중단 배포

## kubernetes pod, service, ingress 관계

kubernetes클러스터 내부에 있는 pod까지 트래픽이 도달하는 경로
![](https://i.imgur.com/gnhOpqV.png)
Ingress-controller: 트래픽을 클러스터 내부까지 전달해줄 LB역할을 해줄 매개체
service: kubernetes 내부에서 pod간 통신을 위해서 존재

#### Kubernetes Pod 배포시 기본 구조
- 기본적인 교체 구조는 새로운 Podv2이 생성되고 헬스체크가 성공한 후 트래픽이 Podv2로 흘러간다 
- 그러고 나서 podV1이 제거된다

고려해야할 부분
1. graceful shutdown 구현이 필요하다!
	1. 새로운 Pod이 실행되고 이전 pod을 종료할때 쿠버네테스에서 노드의 컨테이너를 관리하는 프로세스인 kubelet이 먼저 pod에 sigterm신호를 보내게 된다 
	2. 컨테이너에서 sigterm을 받았을때 기존에 처리중이던 요청에 대한 처리를 완료하고 새로운 요청을 받지 않도록 개발되어 있어야한다!
	3. 그렇지 않으면 트래픽은 아직 podv1으로가는데 podv1이 종료되어 ingress-controller설정이 갱신되기전에 podv1으로 가는 요청들은 에러를 낸다
	4. sigterm을 보낸뒤 일정 시간동안 graceful shutdown 이 되지 않는다면 강제로 SIGKILL을 보내서 pod을 종료하고

> 정확히 언제 서비스 iptables을 수정할까요?

	5. terminationGracePeriodSeconds 으로 설정 가능 기본 30초이다.
![](https://i.imgur.com/JNClk4O.png)

1. rolling update를 위한 MaxSurge, MaxUnavailable 설정
 - maxSurge: deployment에 설정되어 있는 기본 pod개수보다 여분의 pod가 몇개가 더 추가될 수 있는지를 설정
 - maxUnavailable는 업데이트하는 동안 몇 개의 pod가 이용 불가능하게 되어도 되는지
 - => 두가지 옵션을 잘 조절해야한다
 - 둘다 한꺼번에 0으로 설정이 되면 pod이 존재하지 않는다

2. pod의 readinessProbe설정
- kubernetes에서는 pod의 헬스체크를 확인하기 위해 2가지 상태체크 옵션을 주는데
- livenessProbe: 컨테이너가 살아있는지 확인하는 역할, 헬스체크가 실패하면 kubelet이 컨테이너를 죽이게 된다. 그리고 restartPolicy에 따라 컨테이너 재시작
- readinessProbe: 실제로 컨테이너가 서비스 요청을 처리할 준비가 되어있는지 확인
	- 이 설정이 ok상태여야지 **pod와 연결된 service에 pod의 ip가 추가되고 트래픽을 받을 수 있게 된다**
	- 만약 이설정을 하기 힘들다면 .spec.minReadySeconds옵션: pod의 status 가 ready가 될때까지의 최소 대기시간 설정을 통해 -> 이 시간 동안은 트래픽을 받지 않는다
	- 즉 준비될때까지 트래픽을 받지 않는다! 초기화가 오래걸리면 필요!

3. 만약 graceful shutdown을 구현하지 못하는 경우 
- prestop hook: pod 라이프사이클 중에 hook을 설정한다 
- pod이 실행되고난 직후에 실행하는 poststart hook과 pod이 종료되기 직전 실행하는 prestop hook
- prestop훅은 sigterm을 보내기 전에 실행이 되므로 prestop을 이용하면 앱과 별개로 gracefulshutdown의 효과를 낼 수 있다!
- prestop훅의 실행이 완료되기 전까지는 컨테이너에 sigterm을 보내지 않기 때문에 앱의 구현과는 별개로 종료되기 전에 대기시간을 줄 수 도 있다!
- 하지만 이러한 prestop훅도 terminationGracePeriodSeconds시간을 초과하면 프로세스 종료가 일어난다

### 나의 싱송에서 한번 해볼까?

현재는 재배포를 하게 된다면 이렇게 되고 
![](https://i.imgur.com/UWrfbb4.png)


![](https://i.imgur.com/uq0DWkn.png)

![](https://i.imgur.com/Wfwzrmn.png)


다음과 같이 Revision이 하나 더 생기는 것을 확인할 수 있다!

자 그러면 이제 graceful shutdown이 잘 되고 있는지 알고싶다

```
kubectl get events --sort-by=.metadata.creationTimestamp | grep Terminating
```

현재는 pod이 성공적으로 삭제가 되었다는 그러한 어떤 이벤트도 발생하지 않는다 아무래도 그냥 삭제가 되는것 같다. 하지만 이제는 sigterm이 들어오면 gracefully 하게 shutdown 되어야한다. 

Application-> GraceFul Shutdown

```
//if err := r.Run(); err != nil {
	//	log.Fatalf("서버 실행 실패: %v", err)
	//}

quit := make(chan os.Signal, 1)
// kill (no param) default send syscall.SIGTERM
// kill -2 is syscall.SIGINT
// kill -9 is syscall. SIGKILL but can"t be catch, so don't need add it
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
// <-quit → SIGTERM이 발생할 때까지 대기(Blocking).
<-quit
log.Println("Shutdown Server ...")

//5초 동안 서버 종료를 기다릴 수 있는 컨텍스트 생성.
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
if err := srv.Shutdown(ctx); err != nil {
	log.Fatal("Server Shutdown:", err)
}
// catching ctx.Done(). timeout of 5 seconds.
select {
case <-ctx.Done():
	log.Println("timeout of 5 seconds.")
}
log.Println("Server exiting")
```

현재는 pod이 성공적으로 삭제가 되었다는 그러한 어떤 이벤트도 발생하지 않는다 아무래도 그냥 삭제가 되는것 같다. 하지만 이제는 sigterm이 들어오면 gracefully 하게 shutdown 되어야한다. 

일반적인 잘 운영이 되는 상황에서 k6부하테스트를 진행하면

![](https://i.imgur.com/Ay5qAta.png)

다음과 같이 failed 0.00%가 나오는것을 확인할 수 있습니다 그런데 이제 

```
kubectl rollout restart deployment singsong-golang-deployment -n default
```

명령어가 실행되는 와중에 k6를 쏴보겠습니다 

![](https://i.imgur.com/KLvuFx6.png)

라고 합니다. 

위에서 있었던 것처럼 graceful shutdown을 넣긴했는데 이것만으로는 부족했나 봅니다. 

먼저 Readiness Probe를 추가해봅시다!

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: singsong-golang-deployment
  labels:
    app: singsong-golang
spec:
  replicas: 1
  selector:
    matchLabels:
      app: singsong-golang
  template:
    metadata:
      labels:
        app: singsong-golang
    spec:
      containers:
      - name: singsong-golang-container
        image: sunwupark/singsong-golang-private
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: singsong-config
        readinessProbe:
          httpGet:
            path: /   # Go 애플리케이션이 제공하는 헬스 체크 엔드포인트
            port: 8080
          initialDelaySeconds: 5  # Pod 시작 후 첫 체크까지 대기 시간
          periodSeconds: 2        # 2초마다 체크
      imagePullSecrets:
      - name: regcred
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

그러고 나서 kubectl replace -f 로 새로운 버전을 배포했을떄

![](https://i.imgur.com/2eC5lC3.png)

바로 100%가 나오게 되었습니다

event를보니 

![](https://i.imgur.com/CPyNBCs.png)

Unhealthy라고 하니 기존의 deployment pod을 내리지 않고 있다가 30초가 지나자 scaled down 하고 삭제하는 것을 볼 수 있다 