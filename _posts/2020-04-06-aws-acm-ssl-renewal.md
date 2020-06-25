---
layout: post
title: AWS 에서 외부 SSL 인증서 갱신하기
subtitle: 외부에서 발급한 SSL 인증서를 AWS에서 갱신하기
tags: [aws]
comments: true
---

외부에서 발급한 SSL 인증서를 AWS에서 구동중인 웹 사이트에서 갱신 하려면,
 
AWS 의 ACM (AWS Certificate Manager)에서 갱신된 인증서 정보를 입력해야만 갱신이 가능하며,

이때 필요한 정보들과 갱신하는 방법에 대해 알아보겠습니다.

***
### #인증서 파일 정보 입력
ACM에서 인증서 갱신을 위해 필요한 파일은 다음의 3개 파일입니다.

**{DOMAIN_NAME}_key.nopass.pem 파일**

**{DOMAIN_NAME}_cert.pem 파일**

**Chain_RootCA_Bundle.crt 파일**


만약 인증서 vendor사에서 전달받은 인증서 파일 중 **{DOMAIN_NAME}_key.nopass.pem 파일** 파일이 없다면, 

추가 요청해서 발급을 받도록 합니다.



위 3개 파일을 준비 후, 

ACM > 인증서 다시 가져오기 메뉴에서,

![acm](/assets/img/20200406/acm.png)


아래와 같이 파일 내용을 입력 해줍니다.


![acm_ssl_info](/assets/img/20200406/acm_ssl_info.png)




***
### #확인

브라우저에서 인증서 유효 기간이 올바르게 갱신 되었는지 확인 합니다.


![ssl_check](/assets/img/20200406/ssl_check.png)




