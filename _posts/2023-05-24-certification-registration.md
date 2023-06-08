---
title: 인증서 OS 등록
categories: [etc, openssl]
tag: [인증서, openssl]
toc: true
toc_sticky: true
search: true
---

## MacOS

원격 서버에서 인증서를 사용하여 Web 서비스를 제공할 경우, Browser에서 해당 서비스의 인증서를 승인하기 위해 CA 인증서를 등록하게 된다.

1. `ca.crt` 더블 클릭
2. 키체인 접근
3. 사이드바 `시스템` -> 상단 `인증서` 탭 -> `CN` 값으로 지정한 이름의 인증서 더블클릭 ![Screenshot-0589715](https://hognod.synology.me:5543/2023/05/08/Screenshot-0589715.png)
4. `신뢰` 항목 -> `이 인증서 사용 시`를 `항상 신뢰`로 변경 ![Screenshot-0589900](https://hognod.synology.me:5543/2023/05/08/Screenshot-0589900.png)

## CentOS

서비스용 인증서를 등록하여 서비스 이용에 보안을 제공하기 위해 주로 사용된다.

1. 인증서 관련 패키지가 없는 경우 패키지 설치 및 활성화

   ```bash
   sudo yum install ca-certificates
   ```

   ```bash
   sudo update-ca-trust force-enable
   ```



2. 인증서 파일 아래 위치에 복사

   ```bash
   sudo cp ./cert/service/serivce.crt /etc/pki/ca-trust/source/anchors/
   ```



3. 인증서 등록

   ```bash
   sudo update-ca-trust
   ```

   
