---
title: 인증서 생성
categories: [etc, openssl]
tag: [인증서, openssl]
toc: true
toc_sticky: true
search: true
---

## 1. CA 인증서

### 1.1. Root CA Key 생성

```bash
openssl genrsa -out ca.key 2048
```

```bash
chmod 600 ca.key
```



### 1.2. Root CA CSR 생성을 위한 Conf 파일

```bash
vi ca.conf
```

```bash
[ req ]
default_bits            = 2048
default_md              = sha1
default_keyfile         = lesstif-rootca.key
distinguished_name      = req_distinguished_name
extensions             = v3_ca
req_extensions = v3_ca
  
[ v3_ca ]
basicConstraints       = critical, CA:TRUE, pathlen:0
subjectKeyIdentifier   = hash
##authorityKeyIdentifier = keyid:always, issuer:always
keyUsage               = keyCertSign, cRLSign
nsCertType             = sslCA, emailCA, objCA
[req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = KR
countryName_min                 = 2
countryName_max                 = 2
 
# 회사명 입력
organizationName              = Organization Name (eg, company)
organizationName_default      = lesstif Inc.
  
# 부서 입력
#organizationalUnitName          = Organizational Unit Name (eg, section)
#organizationalUnitName_default  = Condor Project
  
# SSL 서비스할 domain 명 입력
commonName                      = Common Name (eg, your name or your server's hostname)
commonName_default             = lesstif's Self Signed CA
commonName_max                  = 64
```

* CA 인증서의 경우 OS에 등록하여 종속되는 인증서들이 유효함을 인증만 해주면 되기 때문에 별도의 `CN`을 지정할 필요없다.



### 1.3. Root CA CSR 생성

```bash
openssl req -new -key ca.key -config ca.conf -out ca.csr
```

```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [KR]:
Organization Name (eg, company) [lesstif Inc.]:
Common Name (eg, your name or your servers hostname) [lesstifs Self Signed CA]:
```

* `ca.conf` 파일에 지정한 default 값으로 지정되게 `Enter` 입력



### 1.4. Root CA 인증서 생성

```bash
openssl x509 -req \
-days 3650 \
-extensions v3_ca \
-set_serial 1 \
-in ca.csr \
-signkey ca.key \
-extfile ca.conf \
-out ca.crt
```



## 2. 서비스 인증서

### 2.1. 서비스 Key 생성

```bash
openssl genrsa -out service.key 2048
```

```bash
chmod 600 service.key
```



### 2.2. 서비스 CSR 생성을 위한 Conf 파일

```bash
vi service.conf
```

```
[ req ]
default_bits            = 2048
default_md              = sha1
default_keyfile         = lesstif-rootca.key
distinguished_name      = req_distinguished_name
extensions             = v3_user
## 인증서 요청시에도 extension 이 들어가면 authorityKeyIdentifier 를 찾지 못해 에러가 나므로 막아둔다.
## req_extensions = v3_user
 
[ v3_user ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
authorityKeyIdentifier = keyid,issuer
subjectKeyIdentifier = hash
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
## SSL 용 확장키 필드
extendedKeyUsage = serverAuth,clientAuth
subjectAltName          = @alt_names
[ alt_names]
## Subject AltName의 DNSName field에 SSL Host 의 도메인 이름을 적어준다.
## 멀티 도메인일 경우 *.lesstif.com 처럼 쓸 수 있다.
DNS.1   = <서비스 Domain 명>
# DNS.2   = lesstif.com
# DNS.3   = *.lesstif.com
 
[req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = KR
countryName_min                 = 2
countryName_max                 = 2
 
# 회사명 입력
organizationName              = Organization Name (eg, company)
organizationName_default      = lesstif Inc.
  
# 부서 입력
organizationalUnitName          = Organizational Unit Name (eg, section)
organizationalUnitName_default  = lesstif SSL Project
  
# SSL 서비스할 domain 명 입력
commonName                      = Common Name (eg, your name or your server's hostname)
commonName_default             = <서비스 Domain 명>
commonName_max                  = 64
```

* `DNS.1` 항목과 `commonName_default` 항목에 서비스의 Domain Name을 입력
  * 인증서 1개로 여러 서비스에 사용하려면 `*.example.com`과 같이 지정
  * `a.example.com`, `b.example.com`과 같이 각 서비스 수 만큼 도메인을 지정하여 인증서를 여러개 만들어 사용하기도 가능
    * 서비스별 도메인이 다를 때 주로 사용 (ex. `a.test.com`, `b.example.net`)




### 2.3. 서비스 CSR 생성

```bash
openssl req -new -key service.key -config service.conf -out service.csr
```

```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [KR]:
Organization Name (eg, company) [lesstif Inc.]:
Organizational Unit Name (eg, section) [lesstif SSL Project]:
Common Name (eg, your name or your servers hostname) [*.example.com]:
```

* `service.conf` 파일에 지정한 default 값으로 지정되게 `Enter` 입력



### 2.4. 서비스 인증서 생성

```bash
openssl x509 -req \
-days 1825 \
-extensions v3_user \
-in service.csr \
-CA ca.crt \
-CAcreateserial \
-CAkey ca.key \
-extfile service.conf \
-out service.crt
```
