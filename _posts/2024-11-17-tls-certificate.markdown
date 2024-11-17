---
layout: post
title:  "SSL/TLS 인증서"
date:   2024-11-17 15:20:00 +0900
categories: backend
---

## SSL/TLS 인증서
웹사이트에서 HTTPS 를 사용한 통신은 거의 필수이다.
HTTPS 는 HTTP 위에 SSL/TLS 프로토콜을 사용하고 이 프로토콜의 핵심 부분중 하나는 인증서 (certificate) 이다.

인증서는 certificate authority (CA) 로부터 발급이되며 CA 로부터 서명이된 공개키이다. CA 는 공개키 소유자 (웹 서비스 제공자) 의 신분을 
검증하고 성공적인 검증의 표시로 CA 의 개인키 (private key) 로 소유자의 공개키를 서명해 인증서 형태로 발급을 해준다.

그러므로 인증서는 클라이언트와 서버 간 아이덴티티 검증을 도와주고 man-in-the-middle attack 같은 공격을 방지할 수 있다.

CA 는 하나만 있는게 아니라 여러개가 계층으로 나뉘어 존재한다. 이것 또한 보안과 효울성을 높인 디자인적인 요소이다. 
제일 위에 여러 root CA 가 있고 그리고 그 밑으로 여러 intermeidate CA 트리형태로 조직도를 그린다.
모든 CA 의 신뢰성 (Trust) 은 root 에서 아래 방향으로 intermediate CA 에게 계승이 된다. 
Root CA 는 바로 아래있는 intermediate CA 가 발급하는 인증서의 효력을 부여하고 이 관계는 제일 하단에 존재하는 leaf CA 까지 적용이된다. 

인증서의 유효성 검증 과정은 반대방향으로 진행이된다. 인증서를 발급한 CA 를 시작으로 root CA 에 도달할때까지 검증은 계속된다.

인증서를 검증을 시작으로 해당 인증서를 발급한 CA 그리고 그 CA 의 인증서를 서명한 상위 CA 로 root CA 에 도달할때까지 진행된다.

그리고 이 보안성을 완성하는건 바로 root CA 의 인증서는 우리가 사용하는 컴퓨터에 이미 존재하거나 또는 브라우저 설치시 딸려온다는 것이다.
결국 모든 검증은 최종적으로 root CA 를 통하기 때문에 로컬에 root CA 를 가지고 있으므로써 통신에서 생기는 보안 취약점을 걱정할필요가 없다.


## 인증서를 발급받는법?
인증서 발급 방법은 다양하다. DigiCert 같은 유료 서비스를 사용할수도 있고 Let's Encrypt 같은 무료 서비스도 존재한다.
Let's Encrypt 같은 경우 Certbot 같은 클라이언트 프로그램을 사용해 인증서 발급부터 재발급까지 자동화 할 수 있다.
여기서 자세히 다루지는 않겠다.

아래에서 간단한 예제를 위해 self-signed 인증서도 만들 수 있다는 보여주지만 실제 서비스에서 사용하기에는 적합하지 않다.
그 이유는 본인이 본인의 유효성을 검증하는식이니 당연히 말이되지 않는다.
이런 인증서는 아래 예제처럼 테스트용으로 만들어 사용하는 정도다.

## nginx 및 인증서를 사용해 HTTPS 로 서빙해보기

#### self-signed 인증서 만들기
1. 개인키 생성 
    ```
    openssl genrsa -out cert.key 2048
    ```
    RSA 를 사용하여 2048 비트 크기의 개인키를 만들어 cert.key 에 저장한다.

2. Certificate Signing Request (CSR) 만들기
    ```
    openssl req -new -key cert.key -out cert.csr
    ```
    CSR 은 인증서를 서명하기를 요망하는 CA 기관에다 전달할 서명 요청이다.

    커멘드를 실행하면 몇 단계에 거처 도메인과 요청하는 기관에 대한 정보를 입력한다.
    이 예시에서는 self-sign 을 진행하기 때문에 엔터를 눌러 디폴트 값으로 진행해도 무방하다.

    만들어진 CSR 파일에는 CLI 에서 입력한 메타데이터와 개인키에 짝이되는 공개키 정보가 들어가있다.

    여기까지는 self-sign 을 진행하든 CA 를 통해 발급을 진행하든 프로세스가 비슷하다.

3. 개인키로 CSR 서명

    원래라면 CSR 을 CA 에 제출을 하지만 self-signed 인증서에 경우, 우리가 1번에서 만든 개인키로
    CSR 파일을 서명해 인증서를 만든다.

    ```
    openssl x509 -req -days 3650 -in cert.csr -signkey cert.key -out cert.crt
    ```

### 간단한 서버 어플리케이션 준비
예제에서는 간단한 서버 어플리케이션을 작성해 도커 컨테이너로 실행했다. 실제 서비스 환경에서는 이 어플리케이션
자리에 원하는 서비스를 끼워넣을수 있을것이다. 원하는 어플리케이션 코드로 대체가 가능하기 때문에 Go 에 관련된
코드나 도커파일 디테일은 여기서 크게 중요하지 않다. 중요한건 nginx 를 reverse-proxy 로 우리의 어플리케이션
앞단에 위치해줄것이라는 거다. 


1. root 경로에서 헬로월드 리턴하는 서버
```go
package main

import (
	"fmt"
	"net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	if r.URL.Path == "/" {
		w.Header().Set("Content-Type", "text/plain")
		w.WriteHeader(http.StatusOK)
		fmt.Fprintln(w, "Hello, World!")
	}
}

func main() {
	http.HandleFunc("/", helloHandler)

	fmt.Println("Starting server on port 8080...")
	http.ListenAndServe(":8080", nil)
}
```

2. 도커파일 생성: 서버 코드 도커 이미지로 빌드
```Dockerfile
FROM golang:alpine

WORKDIR /app

COPY go.mod ./

RUN go mod download && go mod verify

COPY . .
RUN go build -o server

CMD ["./server"]

```

### Nginx 컨테이너 준비
1. Nginx 설정 파일을 아래와 같이 생성해 주자.

```nginx
user  nginx;  # nginx 와 관련된 프로세스 유져/그룹 이름
worker_processes  1;  # nginx worker 프로세스 개수

# 로그 파일 경로
error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;

# Nginx worker 설정
events {
    worker_connections  1024;  # worker 당 최대 연결 개수
}

http {
    server {
        # SSL/TLS 설정
        listen              443 ssl;
        server_name         localhost;
        ssl_certificate     /etc/ssl/certs/cert.crt;
        ssl_certificate_key /etc/ssl/private/cert.key;

        # Reverse Proxy 설정
        location / {
            proxy_pass http://go-server:8080;

            # 클라이언트가 보낸 요청 헤더를 어플리케이션에게 바톤터치하는 설정
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```


2. 도커파일 생성. 여기서 주의 할점은 개인키는 항상 외부로 부터 보호해주어야 한다. 

    주의! 개인키를 공개 저장소에 올리지 말고 또한 빌드된 이미지도 공개적인 레지스트리에 올리면 안된다!

```Dockerfile
FROM nginx:latest

# 앞서 만든 인증서와 개인키를 잘 복사해 주자
COPY cert.crt /etc/ssl/certs/cert.crt
COPY cert.key /etc/ssl/private/cert.key

# 앞서 만든 nginx 설정 파일도 잘 복사해 주자
COPY nginx.conf /etc/nginx/nginx.conf
```

### 실행 및 접속
이제 docker compose 를 통해 Nginx 와 서버 어플리케이션을 실행하자

1. docker-compose.yml 작성
```yml
services:
  go-server:
    build: ./server

  nginx-proxy:
    build: ./nginx
    ports:
      - 8443:443
```

2. docker compose 실행
```sh
docker compose up --build
```

3. 이제 브라우저를 열어서 https://localhost:8443 접속
    잊지말자 우리가 사용한 인증서는 유효하지 않아서 경고창이 먼저 뜰것이다. 더보기 옵션을 펼처 사이트에 접속을 강행해보자.
    "Hello World" 메세지가 보이고 서버가 접속이 되는것을 확인할 수 있을 것이다.

간단하게 작성해보고 싶었는데 생각보다 이것저것 작성한게 많아서 따로 [깃헙](https://github.com/hojoungjang/nginx-reverse-proxy-example)에 사용한 파일들을 올려놓았다.
