# kubeadm을 이용한 인증서 갱신

## Prerequisites
  * kubeadm

## Step0. 인증서 만료 확인
* 목적 : `인증서가 만료되는 시기를 확인한다.`
* 순서 : 
  ```bash
  - kubeadm v1.20 이상 버전인 경우
  kubeadm certs check-expiration
  
  - kubeadm v1.19 이하 버전인 경우
  kubeadm alpha certs check-expiration
  ```
  ```bash
  CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
  admin.conf                 Nov 30, 2021 07:23 UTC   364d                                    no
  apiserver                  Nov 30, 2021 07:23 UTC   364d            ca                      no
  apiserver-etcd-client      Nov 30, 2021 07:23 UTC   364d            etcd-ca                 no
  apiserver-kubelet-client   Nov 30, 2021 07:23 UTC   364d            ca                      no
  controller-manager.conf    Nov 30, 2021 07:23 UTC   364d                                    no
  etcd-healthcheck-client    Nov 30, 2021 07:23 UTC   364d            etcd-ca                 no
  etcd-peer                  Nov 30, 2021 07:23 UTC   364d            etcd-ca                 no
  etcd-server                Nov 30, 2021 07:23 UTC   364d            etcd-ca                 no
  front-proxy-client         Nov 30, 2021 07:23 UTC   364d            front-proxy-ca          no
  scheduler.conf             Nov 30, 2021 07:23 UTC   364d                                    no
  
  CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
  ca                      Nov 21, 2030 06:29 UTC   9y              no
  etcd-ca                 Nov 21, 2030 06:29 UTC   9y              no
  front-proxy-ca          Nov 21, 2030 06:29 UTC   9y              no
  ```

## Step1. 기존 인증서 백업
* 목적 : `기존 인증서 백업을 한다.`
* 순서 : 
  * 기존 인증서 및 config 백업 (필수사항)
  ```bash
  mkdir ~/cert_temp            
  cp /etc/kubernetes/pki ~/cert_temp
  cd /etc/kubernetes/
  cp {admin.conf,controller-manager.conf,kubelet.conf,scheduler.conf} ~/cert_temp
  ```

## Step2. 인증서 갱신
* 목적 : `인증서를 수동으로 갱신한다.`
* 순서 :
  * 아래 명령어는 /etc/kubernetes/pki 에 저장된 CA(또는 프론트 프록시 CA) 인증서와 키를 사용하여 갱신을 수행한다.
  * kubeadm으로 생성된 클라이언트 인증서는 1년 기준이다.
  * warning : 다중화 클러스터 구성의 경우, 모든 컨트롤 플레인 노드에서 이 명령을 실행해야 한다.  
  ```bash
  - kubeadm v1.20 이상 버전인 경우
  kubeadm certs renew all
  
  - kubeadm v1.19 이하 버전인 경우
  kubeadm alpha certs renew all  
  ```  
  ```bash  
  [renew] Reading configuration from the cluster...
  [renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
  
  certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
  certificate for serving the Kubernetes API renewed
  certificate the apiserver uses to access etcd renewed
  certificate for the API server to connect to kubelet renewed
  certificate embedded in the kubeconfig file for the controller manager to use renewed
  certificate for liveness probes to healthcheck etcd renewed
  certificate for etcd nodes to communicate with each other renewed
  certificate for serving etcd renewed
  certificate for the front proxy client renewed
  certificate embedded in the kubeconfig file for the scheduler manager to use renewed   
  ```
  * kube-system pod (kube scheduler, api server, controller, etcd) 재기동
  ```bash
  kubectl delete pod <pod_name> -n kube-system
  
  ex)
  kubectl delete pod -l 'component=kube-apiserver' -n kube-system
  kubectl delete pod -l 'component=kube-scheduler' -n kube-system
  kubectl delete pod -l 'component=kube-controller-manager' -n kube-system
  kubectl delete pod -l 'component=etcd' -n kube-system
  ```
  * config 복사
  ```bash
  cp -i /etc/kubernetes/admin.conf /root/.kube/config  
  ```  
* 비고 :
    * 이미 인증서가 만료된 경우 아래 가이드를 참조하여 인증서를 갱신한다. 
      * 새 인증서 생성 및 config 변경 적용
      ```bash
      kubeadm init phase certs all --config=<설치할때 사용했던 kubeadm-config.yaml 파일 경로>
      
      kubeadm init phase kubeconfig all --config=<설치할때 사용했던 kubeadm-config.yaml 파일 경로>
      ```
      * etcd -> api server -> scheduler -> controller 순으로 재기동
      * kube config 복사
      ```bash
      cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      ```
