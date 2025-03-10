## HTTP 커넥션

#### 브라우저에서 URL 행동

![[Pasted image 20250223213728.png]]

```
http://www.joes-hardware.com:80/power-tools.html
```

1. 브라우저가 www.joes-hardware.com 호스트를 추출한다
2. 이 호스트에 대해서 IP 주소를 찾는다
3. 브라우저가 80번 포트를 얻는다
4. 브라우저가 IP+포트 번호로 TCP 커넥션을 생성한다
5. 브라우저가 HTTP GET 요청 메세지를 보낸다
6. 응답 메세지를 읽는다
7. 커넥션을 끊는다
---
#### HTTP 지연

각 TCP 세그먼트는 하나의 IP 주소에서 다른 IP주소로 IP패킷에 담겨 전달된다
- IP 패킷 헤더(보통 20바이트)
- TCP 세그먼트 헤더(보통 20바이트)
- TCP 데이터 조각(0 혹은 그 이상의 바이트)

![](https://i.imgur.com/aYCwxDl.png)


#### 1. TCP 네트워크 지연
대부분의 HTTP 지연은 TCP 네트워크 지연 때문에 발생한다
![](https://i.imgur.com/zq7u3wh.png)

- 클라이언트는 URI 에서 웹 서버의 IP 주소와 포트번호를 알아
	- **DNS look up**을 해야한다
- **TCP 커넥션 요청을 보내고 회신**
- 커넥션이 맺어지면 **TCP 파이프를 통해 전송**.
	- 읽고 처리되는데까지 시간 소요
- **응답을 보냄**

#### 2. 지연된 ACK
- **확인응답은 크기가 작기 때문에 같은 방향으로 송출되는 패킷에 편승(Piggyback)** 시킨다 
- => **네트워크 효율적으로 사용**
- 특정 시간동안 송출 패킷을 찾는데 없다면 별도의 패킷을 만들어 전송
- 하지만 **요청/응답 두가지 형식 밖에 없어 편승할 기회가 별로 없어 지연이 자주 발생**한다
#### 3. TCP Slow Start
- **처음에는 커넥션의 최대 속도를 제한하고 데이터가 성공적으로 전송됨에 따라 속도 제한을 높여나가는것**
- **급작스러운 부하와 혼잡**을 **방지**한다

####  3-1 네이글 알고리즘
- 세그먼트가 최대크기가 되지 않으면 전송을 하지 않는다
- 하지만 다른 모든 패킷이 확인응답을 받은 경우에는 최대 크기보다 작은 패킷의 전송 허락한다
- 1. 크기가 작은 메세지는 패킷을 채우지 못해 지연
- 2. 확인응답지연과 함께 쓰이면 형편없다. 
- TCP_NODELAY 파라미터 값 설정하여 비활성화하기도 한다

###  Connection 헤더란?
`Connection` 헤더는 현재 맺고 있는 커넥션에만 적용될 옵션을 정의하는 역할을 한다. 이 헤더에 지정된 값들은 다음 요청에서 유지되지 않고, 특정 커넥션에만 한정된다.

Connection 헤더에 전달될 수 있는 토큰
- 커넥션에만 해당되는 헤더
- 임시적인 토큰 값은, 커넥션에 대한 비표준옵션을 의미
- close 값, 커넥션이 작업이 완료되면 종료

> 커넥션 토큰이 HTTP 헤더 필드명을 가지고 있으면 다음 커넥션에는 전달하면 안된다
- Hop-by-Hop 헤더명 기술

### HTTP 커넥션 성능 향상 기술

|       | 병렬 커넥션                                                                        | 지속 커넥션                                               |
| ----- | ----------------------------------------------------------------------------- | ---------------------------------------------------- |
| 설명    | 여러 개의 TCP 커넥션을 동시에 맺어 병렬 처리하는 방식 병렬 처리를 통해 각 커넥션의 지연 시간을 겹쳐 총 요청 시간을 단축할 수 있음 | HTTP 요청이 완료된 후에도 TCP 연결을 유지하여 추가 요청을 처리할 수 있도록 하는 방식 |
| 장점    | 여러 개의 요청을 동시에 처리할 수 있음                                                        | 커넥션을 유지하여 연결 비용을 줄일 수 있음                             |
| 단점    | 새로운 커넥션을 계속 생성해야 함 (Slow Start)                                               | 잘못 관리하면 커넥션이 과도하게 유지됨                                |
| 사용 사례 | 빠르게 여러 개의 독립적인 요청을 보낼 때                                                       | 같은 서버에 여러 개의 요청을 보낼 때                                |

#### 병렬 커넥션의 장점
- **여러 개의 리소스를 동시에 요청**할 수 있어 성능이 향상될 가능성이 있음
#### 병렬 커넥션의 단점
- 항상 더 빠른 것은 아님. **네트워크 대역폭이 좁다면 병렬 커넥션의 효과가 크지 않음**
- 새로운 TCP 커넥션이 생성될 때 초기 속도가 느려지는 **Slow Start 문제** 발생
- 많은 커넥션을 유지하면 **메모리와 CPU 사용량 증가**
- 일반적으로 4개 이하의 병렬 커넥션만 허용됨
#### 지속 커넥션의 장점
- 새로운 커넥션을 맺기 위한 지연 시간을 줄일 수 있음
- 이미 **튜닝된 TCP 커넥션을 유지하므로 성능이 향상됨**
- 커넥션 수를 줄여 서버와 클라이언트의 **리소스 사용량 절감**
#### 지속 커넥션의 단점
- 관리가 제대로 되지 않으면 **불필요한 커넥션이 계속 유지**될 수 있음
- 너무 많은 지속 커넥션이 유지되면, 서버의 연결 수가 제한되어 새로운 요청을 처리하지 못하는 문제 발생

> **Persistent Connection이 필요한 이유**
> "Site Locality": 웹에서 특정 페이지를 보여주기 위해서 서버에 연속적으로 이미지 request를 보내는 것처럼 서버에 연속적으로 동일한 클라이언트가 여러 요청을 보낼 가능성이 높은 경우를 의미
> 즉 TCP Connection을 요청마다 close 하지 않고 재사용할 수 있는 방법 -> persistent connection
#### HTTP Keep Alive란?
- persistent connection을 맺는 기법중 하나로 HTTP/1.0+부터 지원한다
- HTTP 1.1은 Persistent Connection을 지원하지만 HTTP/1.0은 하나의 request에 응답할때마다 connection을 close 하도록 설정되어있다

##### HTTP/1.0+의 Keep-Alive 커넥션
- keep-alive 옵션을 통해 persistent connection을 맺기 위해서는 HTTP header에 아래와 같이 입력해주어야 한다. 만약 **서버에서 keep-alive connection을 지원**하는 경우에는 **동일한 헤더를 response에 담아** 보내주고, **지원하지 않으면 헤더에 담아 보내주지 않는다**. 만약 서버의 응답에 해당 헤더가 없을 경우 client는 지원하지 않는다고 가정하고 connection을 재사용하지 않는다.

```
НТТР/1.1 200 0K
Connection: Keep-Alive
Keep-Alive: timeout=5, max=1000
```

1. max (MaxkeepAliveRequests) : keep-alive connection을 통해서 주고받을 수 있는 request의 최대 갯수. 이 수보다 더 많은 요청을 주고 받을 경 우에는 connection은 close된다.
2. timeout (KeepAlivetimeout) : 커넥션이 idle한 채로 얼마동안 유지될 것인가를 의미한다. 이 시간이 지날 동안 request가 없을 경우에 connection은 close된다.

링크(https://etloveguitar.tistory.com/137)

> 프락시와 게이트웨이는 Connection 헤더의 규칙을 철저히 지켜야 한다. 
> 프락시와 게이트웨이는 메시지를 전달하거나 캐시에 넣기 전에 **Connection 헤더에 명시된 모든 헤더 필드와 Connection 헤더를 제거**해야 한다.

![](https://i.imgur.com/NGix6xv.png)

이를 해결하기 위해 **Proxy Connection이라는 헤더**를 사용
![](https://i.imgur.com/KnnSnv2.png)

Connection 헤더 대신에 비표준인 Proxy-Connection 확장 헤더를 프락시에게 전달한다. 
프락시가 Proxy-Connection 헤더를 무조건 전달하더라도 웹 서버는 그것을 무시하기 때문에 문제 X
하지만 영리한 프락시(지속 커넥션 핸드셰이킹을 이해할 수 있는)라면, 의미 없는 Proxy-Connection 헤더를 Connection 헤더로 바꿈으로써 원하던 효과를 얻음

하지만 아래와 같이 두개이상의 프락시를 사용하게 되면 이러한 문제가 발생할 수가 있다

![[Pasted image 20250223231225.png]]

#### HTTP 1.1에서의 지속 커넥션 개선

**고로 HTTP 1.1에서는 이러한 커넥션을 지원하지 않는다**
**대신 설계가 더 개선된 지속 커넥션을 지원한다**

**HTTP 1.1의 지속 커넥션은 기본으로 활성화**
**고로 Connection: close헤더가 응답에 없으면 응답후에도 커넥션을 유지하는것으로 추정**
하지만 언제든 끊을 수 있다
HTTP/1.1 프록시 서버는 **클라이언트**가 커넥션 관련 기능에 대한 **클라이언트의 지원 범위를 알고 있지 않는 한 지속 커넥션을 맺으면 안됨**

#### HTTP 1.1의 지속 커넥션을 통한 파이프라인 커넥션
 ![](https://i.imgur.com/iVck8lq.png)
조건들
1. HTTP 클라이언트는 커넥션이 지속 커넥션인지 확인하기 전까지 파이프라인을 이어서는 안됩니다.
2. HTTP 응답은 요청 순서와 같게 와야 합니다. HTTP 메시지는 순번이 매겨저 있지 않아서 응답이 순서 없이 오면 순서에 맞게 정렬시킬 방법이 없습니다.
3. HTTP 클라이언트는 커넥션이 언제 끊어지더라도, 완료되지 않은 요청이 파이프라인에 있으면 언제든 다시 요청을 보낼 준비가 되어 있어야 합니다.
4. HTTP 클라이언트는 POST 요청(비멱등 요청)같이 반복해서 보낼 경우 문제가 생기는 요청은 파이프라인을 통해 보내면 안됩니다. 에러가 발생하면 파이프라인을 통한 요청 중에 어떤 것들이 서버에서 처리되었는지 클라이언트가 알 방법이 없기 때문입니다.

#### 커넥션 끊기

지속 커넥션이 일정 시간 동안 요청을 전송하지 않고 유휴 상태에 있으면 서버는 그 커넥션을 끊을 수 있다.
하지만 서버가 그 유휴상태에 있는 커넥션을 끊는 시점에, 서버는 클라이언트가 데이터를 전송하지 않을 것이라고 확신하지 못한다. 만약 이 일이 벌어지면 클라이언트는 그 요청 메시지를 보내는 도중에 문제가 생김

정적인 HTML 페이지를 GET 하는 부류의 요청들은 반복적으로 요청하더라도 결과적으로 아무런 영향을 끼치지 않는다. 
반면 온라인 서점에 주문을 POST 하는 부류의 요청들은 반복될 경우 주문이 여러 번 중복될 것이기 때문에 반복은 피해야 한다.

> 한 번 혹은 여러 번 실행됐는지에 상관없이 같은 결과를 반환한다면 그 트랜잭션은 멱등(idempotent)하다고 한다. GET, HEAD, PUT, DELETE, TRACE 그리고 OPTIONS 메서드들은 멱등하다고 이해하면 된다. 

**클라이언트는 POST와 같이 멱등인 아닌 요청은 파이프라인을 통해 요청하면 안 된다**. 그렇지 않으면 전송 커넥션이 예상치 못하게 끊어져 버렸을 때, 알 수 없는 결과를 초래할 수 있다. **비멱등인 요청을 다시 보내야 한다면, 이전 요청에 대한 응답을 받을 때까지 기다려야 한다. 비멱등인 메서드나 순서에 대해 에이전트가 요청을 다시 보낼 수 있도록 기능을 제공한다 하더라도, 자동으로 재시도하면 안 된다.** 예를 들어 대부분 브라우저는 캐시된 POST 요청 페이지를 다시 로드하려고 할 때, 요청을 다시 보내기를 원하는지 묻는 대화상자를 보여준다

#### 전체 끊기와 절반 끊기
![](https://i.imgur.com/EFFYlwu.png)

- TCP 입력 채널, 출력 채널 중 한개만 끄거나 둘다 끊을 수 있따
- close() -> 전체끊기
- shutdown() -> 절반 끊기

보통 커넥션의 출력 채널을 끊는것이 안전하다
반대편의 기기는 모든 데이터를 버퍼로부터 읽고 나서 데이터 전송이 끊남과 동시에 당신이 커넥션을 끊을 것을 알게됨
입력채널을 끊는것은 위험 -> 끊긴 입력 채널에 데이터 전송 -> 서버의 운영체제는 TCP "connection reset by peer" 메세지를 보낸다. 심각한 에러로 판단 -> 버퍼에 저장된 아직 읽히지 않은 데이터 모두 삭제

#### 우아하게 커넥션 끊기
- 자신의 출력 -> 다른쪽의 출력 끊기기를 기다리기 

#### 그렇다면 무중단 배포에서 이러한 지속 커넥션 문제는 어떠한 문제를 일으킬 수 있는가?
1. 배포 중인 서버가 종료되는데, 기존 클라이언트의 지속 커넥션이 아직 살아있음
- 클라이언트가 Connection: keep-alive를 사용하여 연결을 유지하려 하지만, 서버가 내려가면서 요청이 실패함.
2.	로드 밸런서가 새로운 서버로 트래픽을 보내지 못하고, 기존 서버로 계속 요청을 전달하는 문제
- 무중단 배포를 위해 기존 서버를 정상적으로 종료하기 전에 로드 밸런서에서 트래픽을 차단해야 하지만, 기존 연결이 살아 있으면 문제가 됨.
3. 클라이언트가 새로운 서버로 자동으로 연결되지 않음
- HTTP/1.1의 지속 커넥션을 사용하면 클라이언트는 기존 서버와 연결을 유지하려고 하므로, 새로운 서버로의 연결이 즉시 이루어지지 않을 수 있음.


1. 로드 밸런서의 Connection Draining or Deregistration delay 옵션을 사용
- 기존 요청이 완료될 때까지 일정 시간 동안 **새로운 연결을 차단하고, 기존 연결이 종료되도록 대기**하는 방식
2. Graceful Shutdown 
- 서버가 SIGTERM을 받으면 새로운 연결을 받지 않는다
- 기존 연결이 완료될때까지 대기
- 모든 요청이 처리된후 서버를 안전하게 종료

과 같은 것이 있다

Kubernetes에서
```
terminationGracePeriodSeconds => 기존 요청 처리할 시간 확보

spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # 한번에 다운되는 Pod 개수 제한
      maxSurge: 1        # 한번에 추가되는 Pod 개수 제한
  와 같이 설정

애플리케이션의 SIGTERM 설정이 필요

서비스의 상태를 바로 업데이트해야한다!
```
