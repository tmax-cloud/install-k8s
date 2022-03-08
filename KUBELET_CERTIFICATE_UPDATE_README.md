# kubelet 인증서 갱신

## Prerequisites
  * kubelet
 
## Step0. kubelet 인증서 갱신
* 목적 : `k8s와는 별개로 kubelet 인증서 또한 갱신이 필요하다. kubelet 인증서 관련 에러가 난 경우 인증서를 수동으로 갱신한다.`
* 순서 :
  * ERROR : part of the existing bootstrap client certificate is expired: 2023-01-05 05:10:08 +0000 UTC" 만료
  * kubelet 인증서는 기본적으로 1년 기준이다. 설정에 따라 기간을 설정할 수 있다. 
  
  * 기존 kubelet.crt 파일 백업
  ```bash
  cd /var/lib/kubelet/pki
  mv kubelet.crt kubelet.crt_backup
  ```
  
  * crt 파일 발급받기 위해 csr 수동 생성 후 아래 맞는 설정 입력
  ```bash
  openssl req -new -days 3650 -key kubelet.key -out kubelet.csr
  
  Country Name (2 letter code) [XX]:KR
  State or Province Name (full name) []:SEOUL
  Locality Name (eg, city) [Default City]:GANGNAMEGU
  Organization Name (eg, company) [Default Company Ltd]:TMAX
  Organizational Unit Name (eg, section) []:CK
  Common Name (eg, your name or your server's hostname) []:jinho
  Email Address []:jinho_choi@tmax.co.kr

  ```

  * alt_names에 IP 값을 포함시킨 openssl.cnf 파일 생성
  ```bash
  tee kubelet-openssl.cnf << EOF
  [ req ]
  x509_extensions = v3_req
  [ v3_req ]
  keyUsage = critical, digitalSignature, keyEncipherment
  extendedKeyUsage = serverAuth
  basicConstraints = critical, CA:FALSE
  subjectAltName = @alt_names
  [ alt_names ]
  DNS.1 = <호스트 이름>
  IP.1 = <호스트 IP>
  EOF
  ```
  
  * kubelet.crt 수동 발급 (10년)
  ```bash
  openssl x509 -req -days 3650 -in kubelet.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -extfile ./kubelet-openssl.cnf  -extensions v3_req -out kubelet.crt 
  ```
  
  * kubelet 재기동 이후 정상 기동 확인
  ```bash
  systemctl restart kubelet
  
  systemctl status kubelet
  Certificate expiration is 2023-03-03 05:38:24 +0000 UTC, rotation deadline is 2023-01-19 22:14:04.839547816 +0000 UTC 갱신 완료
  ```
