---
layout: single
title:  "[니어니어 프로젝트] https 적용 trouble shooting"
published : true
categories: niornear_project
toc: true
author_profile: false
sidebar:
    nav: "docs"
---

https 스프링 부트 서버에 적용하고, https://domain.com/~ 로 웹 브라우저에서 요청을 보내는데, 인증되지 않았다고 뜸

### 해결

1. nginx 설치

   ```
   sudo apt install nginx
   ```

   

2. Let's encrypt 설치

   ```
   sudo apt install certbot
   ```

   

3. 도메인으로 인증서 발급 받기

   - SSL 인증서 발급

   ```shell
   sudo certbot certonly --standalone -d sub.domain.com -d www.sub.domain.com
   ```

   - 스프링부트 서버에 적용할 Keystore 파일 생성

   ```shell
   sudo openssl pkcs12 -export -in /etc/letsencrypt/live/sub.domain.com/fullchain.pem -inkey /etc/letsencrypt/live/sub.domain.com/privkey.pem -out keystore.p12 -name tomcat
   ```

4. mynginx.conf 생성

   ```nginx
   server {
           listen 80;
           listen [::]:80;
           server_name sub.domain.com www.sub.domain.com;
           root /usr/share/nginx/html;
   
           # Load configuration files for the default server block
           include /etc/nginx/default.d/*.conf;
   
           location / {
                   proxy_pass https://sub.domain.com;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Fowarded-For $proxy_add_x_forwarded_for;
                   proxy_set_header Host $http_host;
           }
   }
   ```

   

5. /etc/nginx/nginx.conf 에 http 블럭에 include 문 추가

   ```nginx
   include /etc/nginx/sites-available/mynginx.conf;
   ```

   

6. 다시 nginx 다시 시작하고 접속하면 https 정상 접속됨

   ```shell
   sudo systemctl reload nginx
   sudo systemctl restart nginx
   ```

   