# WSGI: Python 웹 서버와 프레임워크의 연결 고리
#WSGI #Python #웹서버 #Django #Flask

**Summary**: WSGI는 Python 웹 프레임워크와 웹 서버 간의 표준 인터페이스로, 효율적인 요청 처리를 가능하게 합니다. 이는 과거 CGI의 비효율성을 개선하여, 웹 애플리케이션과 서버 간의 원활한 통신을 지원합니다.

## Content

WSGI(Web Server Gateway Interface)는 Python 웹 프레임워크와 웹 서버 간의 표준 인터페이스로, 주로 Django와 Flask에서 사용됩니다.

### Background
과거에는 CGI(Common Gateway Interface)를 통해 웹 서버가 Python 코드를 실행했으나, 요청마다 새로운 프로세스를 생성하는 방식이어서 속도가 느리고 리소스 사용량이 많았습니다. WSGI는 이러한 문제를 해결하고, 웹 애플리케이션과 웹 서버 간의 표준을 제공하여 효율적으로 요청을 처리할 수 있도록 합니다.

![WSGI Flow](https://i.imgur.com/WkLWfid.png)

### 처리 흐름
1. 클라이언트가 요청을 보냅니다 (예: 브라우저에서 `http://example.com` 접속).
2. 웹 서버(Apache, Nginx, Gunicorn 등)가 요청을 수신합니다.
3. 웹 서버는 WSGI 애플리케이션 객체를 호출하고, 요청 정보를 전달합니다.
4. WSGI 애플리케이션(Flask, Django 등)이 요청을 처리하고 HTTP 응답을 생성합니다.
5. 웹 서버가 응답을 클라이언트에게 반환합니다.

### 사용
WSGI는 프로토콜(표준)일 뿐이며, 실제 요청을 처리하려면 WSGI 서버가 필요합니다.

**대표적인 WSGI 서버**:
- **Gunicorn** (Green Unicorn): Flask, Django에서 가장 많이 사용됩니다.
- **uWSGI**: 고성능, 멀티스레드 지원, 대규모 서비스에서 사용됩니다.
- **mod_wsgi**: Apache HTTP 서버에서 WSGI 애플리케이션을 실행할 때 사용됩니다.

### Limitation
WSGI는 동기적으로 요청을 처리하기 때문에 다음과 같은 한계가 있습니다:
- 요청 하나를 처리하는 동안 다른 요청을 기다려야 합니다.
- WebSocket과 같은 실시간 통신을 지원하지 않습니다.
- 비동기 I/O 작업(예: 데이터베이스 쿼리, 외부 API 요청)이 많으면 성능이 떨어집니다.

![WSGI Limitations](https://i.imgur.com/sB3SXPl.png)

[A Detailed Study of WSGI - Web Server Gateway Interface of Python](https://www.cabotsolutions.com/blog/a-detailed-study-of-wsgi-web-server-gateway-interface-of-python)

## Key Points
- WSGI는 Python 웹 프레임워크와 웹 서버 간의 표준 인터페이스입니다.
- 과거 CGI의 비효율성을 개선하여 효율적인 요청 처리를 지원합니다.
- 대표적인 WSGI 서버로 Gunicorn, uWSGI, mod_wsgi가 있습니다.
- WSGI는 동기적 처리 방식으로 인해 실시간 통신 및 비동기 작업에 한계가 있습니다.

## Follow-up Questions / Discussion Points
1. WSGI가 CGI에 비해 효율적인 이유는 무엇인가요?
2. WSGI의 동기적 처리 방식이 가지는 한계는 무엇이며, 이를 극복할 수 있는 방법은 무엇인가요?
3. Gunicorn과 uWSGI의 차이점은 무엇인가요?
4. WSGI가 지원하지 않는 WebSocket과 같은 실시간 통신을 구현하려면 어떤 대안이 있을까요?
5. WSGI 서버를 선택할 때 고려해야 할 요소는 무엇인가요?