![[Pasted image 20221026174642.png]]
SSL
= 보안인증서
Secure Socket Layer
https 로 접속하는 was(443포트) 서버가 SSL 발급기관을 통해 돈을 내거나 무료로 발급받아 갖고있는 인증문서
client는 해당 서버가 발급받은 ssl 을 가져와서 등록해놓으면 암호화상태로 서버와 통신한다
TLS(Transport Layer Security) 는 SSL 의 업그레이드버전이지만, SSL과 일반적으로 구분해서 사용하진 않는다.

# openssl
https://gaesae.com/75