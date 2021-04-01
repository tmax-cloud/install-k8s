# k8s-master installer 사용법

## Prerequisites
  * 해당 installer는 폐쇄망 기준 가이드입니다.
  * OS 설치 및 package repo를 아래 가이드에 맞춰 설치합니다.
    * https://github.com/tmax-cloud/install-pkg-repo
  * image registry를 아래 가이드에 맞춰 구축합니다.
    * https://github.com/tmax-cloud/install-registry/tree/5.0
  * image registry에 이미지를 push 합니다.  
    * https://github.com/tmax-cloud/install-k8s/blob/5.0/README.md

## Step0. k8s.config 설정
* 목적 : `k8s 설치 진행을 위한 k8s config 설정`
* 순서 : 
  * 환경에 맞는 config 내용을 작성합니다.
     * imageRegistry={IP}:{PORT}
       * ex : imageRegistry=172.22.5.2:5000
     * crioVersion={crio version}
       * ex : crioVersion=1.17
     * k8sVersion={kubernetes version}
       * ex : k8sVersion=1.17.8
     * apiServer={kubernetes API server ip}
       * ex : apiServer=172.21.7.2
     * podSubnet={POD_IP_POOL}/{CIDR}
       * ex : podSubnet=10.244.0.0/16

## Step1. installer 실행
* 목적 : `k8s 설치 진행을 위한 shell script 실행`
* 순서 : 
	```bash
  sudo ./k8s_infra_installer.sh up
	```
* 비고 :
    * k8s.config, k8s_infra_installer.sh파일과 yaml 디렉토리는 같은 디렉토리 내에에 있어야 합니다.
    * runtime docker 이용시 아래의 명령어를 실행합니다. 
    ```bash
    sudo ./k8s_infra_installer.sh up_docker
    ```    
