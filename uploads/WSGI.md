> WSGI(Web Server Gateway Interface)는 **Python 웹 프레임워크와 웹 서버 간의 표준 인터페이스**

> 대표적으로 Django와 Flask에서 사용함
#### Background
- 과거에는 **CGI(Common Gateway Interface)**를 사용하여 웹 서버가 Python 코드를 실행
- **요청마다 새로운 프로세스를 생성하는 방식**이었기 때문에 속도가 느리고 리소스 사용량이 많았음
- WSGI는 웹 애플리케이션과 웹 서버 간의 표준을 제공하여 **효율적으로 요청을 처리**할 수 있도록 함.


![](https://i.imgur.com/WkLWfid.png)

#### 처리 흐름
- **클라이언트가 요청**을 보냄 (ex: 브라우저에서 `http://example.com` 접속).
- **웹 서버(Apache, Nginx, Gunicorn 등)가 요청을 수신**.
- **웹 서버는 WSGI 애플리케이션 객체를 호출**하고, 요청 정보를 전달함.
- **WSGI 애플리케이션(Flask, Django 등)이 요청을 처리**하고 HTTP 응답을 생성.
- **웹 서버가 응답을 클라이언트에게 반환**.

#### 사용
WSGI는 **프로토콜(표준)**일 뿐이며, 실제 요청을 처리하려면 **WSGI 서버**가 필요.

✅ 대표적인 WSGI 서버:

- **Gunicorn** (Green Unicorn) → Flask, Django에서 가장 많이 사용됨.
- **uWSGI** → 고성능, 멀티스레드 지원, 대규모 서비스에서 사용됨.
- **mod_wsgi** → Apache HTTP 서버에서 WSGI 애플리케이션을 실행할 때 사용됨.


#### Limitation
WSGI는 **동기적(Synchronous)으로 요청을 처리**하기 때문에 다음과 같은 한계

- 요청 하나를 처리하는 동안 다른 요청을 기다려야 함.
- WebSocket과 같은 실시간 통신을 지원하지 않음.
- 비동기 I/O 작업(예: 데이터베이스 쿼리, 외부 API 요청)이 많으면 성능이 떨어짐.

![](https://i.imgur.com/sB3SXPl.png)
[A Detailed Study of WSGI - Web Server Gateway Interface of Python](https://www.cabotsolutions.com/blog/a-detailed-study-of-wsgi-web-server-gateway-interface-of-python)



