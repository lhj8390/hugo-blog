+++
author = "lhj8390"
title = "nginx에서 ssl 설정"
date = "2022-07-31"
description = "nginx에서 ssl 보안 인증서를 설정하는 법을 설명한다."
tags = ["DevOps", "server", "linux", "nginx"]
categories = [
    "DevOps"
]
subcategories = ["server"]
+++
## HTTPS 키 발급

> 도메인 회사에서 인증서 발급받을 수 있다.\
> 인증 파일들은 보통 아래와 같이 구성
>- root.crt : root 파일
>- chain.pem : chain 파일
>- domain.crt : 서버 인증서
>- domain.key.pem : 비공개 키
    

## 발급된 키 공개키 만들기

순서대로 서버 인증서, chain, root파일을 cert.pem 파일로 병합한다.

```bash
$ cat domain.crt chain.pem root.crt > cert.pem
```

💡 <span class="red">순서가 어긋나면 인증서 사용 불가!</span>

## Nginx에 HTTPS 적용

합쳐진 cert.pem 파일과 비공개 키를 nginx 에 적용하면 된다.

```
server {
       listen 443 ssl default_server;
       ssl on // deprecated from Nginx v1.15
       server_name example.com
       ssl_certificate /directory/to/cert.pem // 병합한 파일
       ssl_certificate_key /directory/to/domain.key.pem  // 비공개 키
       ssl_protocols TLSv1.1 TLSv1.2;

       location / {
           root /home/ubuntu/web/dist/;
           index index.html index.htm index.nginx-debian.html;
           try_files $uri $uri/ /index.html;
       }
}
```

- `ssl_certificate` 에는 합쳐진 cert.pem 명시
- `ssl_certificate_key` 에는 비공개 키 명시

## HTTP를 HTTPS로 리다이렉션

```
server {
       listen 80;
       server_name example.com;
       return 301 HTTPS://$server_name$request_uri;
}
```

80포트로 접속 시 301(영구 리다이렉션)하도록 설정한다.