---
layout: single
title:  "[니어니어 프로젝트] nginx 에 https 적용(Let's Encrypt)"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

## 1. 도메인 발급받기

- 가비아에 가서 도메인 발급 받고, EC2 서버와 연결하기

## 2. Let's Encrypt 클라이언트 다운로드

Ubuntu 18.04 버전 이상에서는 python3 버전으로 대체

```shell
sudo apt-get update
sudo apt-get install certbot
sudo apt-get install python3-certbot-nginx
# sudo apt-get install python-certbot-nginx
```

## 3. nginx ssl 설정

1. nginx 설치 후 텍스트 편집기로 아래 경로에 가서 파일 만들고, default 에 include 하기

```shell
sudo vim /etc/nginx/conf.d/domain-name.conf
```

2. server_name 지시문을 사용해서 도메인 이름 지정

```shell
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name example.com www.example.com;
}
```

3. 파일 저장하고 nginx 다시시작

```shell
sudo systemctl restart nginx
```

## 4. SSL/TLS 인증서 받기

1. NGINX 플러그인으로 인증서 생성

```shell
sudo certbot --nginx -d example.com -d www.example.com
```

2. certbot 이메일 주소 입력하고 Let's Encrypt 서비스 약관에 동의하는 것을 포함하여 HTTPS 설정 구성하라는 메시지에 응답
3. 인증서 생성 완료되면 NGINX 가 새 설정 다시 로드
4. https 로 접속 잘 되는지 확인하기!



---

출처 : https://nginxstore.com/blog/nginx/lets-encrypt-%EC%9D%B8%EC%A6%9D%EC%84%9C%EB%A1%9C-nginx-ssl-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/