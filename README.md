## 구성 요소 및 버전
* cri-o (v1.17.4) or docker-ce(v18.09.7)
* kubeadm, kubelet, kubectl (v1.17.6)
* k8s.gcr.io/kube-apiserver:v1.17.6
* k8s.gcr.io/kube-proxy:v1.17.6
* k8s.gcr.io/kube-scheduler:v1.17.6
* k8s.gcr.io/kube-controller-manager:v1.17.6
* k8s.gcr.io/etcd:3.4.3-0
* k8s.gcr.io/pause:3.1
* k8s.gcr.io/coredns:1.6.5

## Prerequisites
* 클러스터 구성전 master, worker node 최소 스팩
  * master node (controll plane node) - CPU : 2Core 이상
  * master/worker node - RAM : 2GiB 이상

## 폐쇄망 구축 가이드 
1. **폐쇄망에서 설치하는 경우** 아래 가이드를 참고 하여 image registry를 먼저 구축한다.
    * https://github.com/tmax-cloud/hypercloud-install-guide/tree/master/Image_Registry   
2. 사용하는 image repository에 k8s 설치 시 필요한 이미지를 push한다. 
    * 작업 디렉토리 생성 및 환경 설정
    ```bash
    $ mkdir -p ~/k8s-install
    $ cd ~/k8s-install
    $ export REGISTRY={ImageRegistryIP:Port}
      ex) $ export REGISTRY=172.22.5.2:5000
    ```
    * 외부 네트워크 통신이 가능한 환경에서 필요한 이미지를 다운받는다.
    ```bash
    $ sudo docker pull k8s.gcr.io/kube-proxy:v1.17.6
    $ sudo docker pull k8s.gcr.io/kube-apiserver:v1.17.6
    $ sudo docker pull k8s.gcr.io/kube-controller-manager:v1.17.6
    $ sudo docker pull k8s.gcr.io/kube-scheduler:v1.17.6
    $ sudo docker pull k8s.gcr.io/etcd:3.4.3-0
    $ sudo docker pull k8s.gcr.io/coredns:1.6.5
    $ sudo docker pull k8s.gcr.io/pause:3.1
    ```
    ![image](figure/dockerimages.PNG)
    * docker image를 tar로 저장한다.
    ```bash
    $ sudo docker save -o kube-proxy.tar k8s.gcr.io/kube-proxy:v1.17.6
    $ sudo docker save -o kube-controller-manager.tar k8s.gcr.io/kube-controller-manager:v1.17.6
    $ sudo docker save -o etcd.tar k8s.gcr.io/etcd:3.4.3-0
    $ sudo docker save -o coredns.tar k8s.gcr.io/coredns:1.6.5
    $ sudo docker save -o kube-scheduler.tar k8s.gcr.io/kube-scheduler:v1.17.6
    $ sudo docker save -o kube-apiserver.tar k8s.gcr.io/kube-apiserver:v1.17.6
    $ sudo docker save -o pause.tar k8s.gcr.io/pause:3.1
    ```
    ![image](figure/dockersave.PNG)
3. 위의 과정에서 생성한 tar 파일들을 폐쇄망 환경으로 이동시킨 뒤 사용하려는 registry에 이미지를 push한다.
    ```bash
    $ sudo docker load -i kube-apiserver.tar
    $ sudo docker load -i kube-scheduler.tar
    $ sudo docker load -i kube-controller-manager.tar 
    $ sudo docker load -i kube-proxy.tar
    $ sudo docker load -i etcd.tar
    $ sudo docker load -i coredns.tar
    $ sudo docker load -i pause.tar
    ```
    ![image](figure/dockerload.PNG)
    ```bash
    $ sudo docker tag k8s.gcr.io/kube-apiserver:v1.17.6 ${REGISTRY}/k8s.gcr.io/kube-apiserver:v1.17.6
    $ sudo docker tag k8s.gcr.io/kube-proxy:v1.17.6 ${REGISTRY}/k8s.gcr.io/kube-proxy:v1.17.6
    $ sudo docker tag k8s.gcr.io/kube-controller-manager:v1.17.6 ${REGISTRY}/k8s.gcr.io/kube-controller-manager:v1.17.6
    $ sudo docker tag k8s.gcr.io/etcd:3.4.3-0 ${REGISTRY}/k8s.gcr.io/etcd:3.4.3-0
    $ sudo docker tag k8s.gcr.io/coredns:1.6.5 ${REGISTRY}/k8s.gcr.io/coredns:1.6.5
    $ sudo docker tag k8s.gcr.io/kube-scheduler:v1.17.6 ${REGISTRY}/k8s.gcr.io/kube-scheduler:v1.17.6
    $ sudo docker tag k8s.gcr.io/pause:3.1 ${REGISTRY}/k8s.gcr.io/pause:3.1
    ```
    ![image](figure/tag.PNG)
    ```bash
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/kube-apiserver:v1.17.6
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/kube-proxy:v1.17.6
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/kube-controller-manager:v1.17.6
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/etcd:3.4.3-0
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/coredns:1.6.5
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/kube-scheduler:v1.17.6
    $ sudo docker push ${REGISTRY}/k8s.gcr.io/pause:3.1
    ```
    ![image](figure/push.PNG)
    ```bash
    $ curl ${REGISTRY}/v2/_catalog
    ```    
    ![image](figure/check.PNG)
* 비고 :
    * 위 내용은 2개이상의 마스터 구축시 마스터 1개에서만 진행한다.

## 설치 가이드
0. [(Master/Worker 공통) 환경 설정](/README.md#step0-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95)
1. [(Master/Worker 공통) runtime 설치 - cri-o 설치](/README.md#step-1-cri-o-%EC%84%A4%EC%B9%98)
1-1. [docker-ce 설치](/README.md#step-1-cri-o-%EC%84%A4%EC%B9%98)
2. [(Master/Worker 공통) kubeadm, kubelet, kubectl 설치](/README.md#step-2-kubeadm-kubelet-kubectl-%EC%84%A4%EC%B9%98)
3. [(Master) kubernetes cluster 구성](/README.md#step-3-kubernetes-cluster-%EA%B5%AC%EC%84%B1)
3-1. [kubernetes cluster 구성(master 다중화)](/README.md#step-3-1-kubernetes-cluster-%EB%8B%A4%EC%A4%91%ED%99%94-%EA%B5%AC%EC%84%B1%EC%9D%84-%EC%9C%84%ED%95%9C-keepalived-%EC%84%A4%EC%B9%98)
4. [(Worker) kubernetes cluster join](/README.md#step-4-cluster-join-worker)

## 삭제 가이드
0. [(Master/Worker 공통) kubernetes cluster reset](https://github.com/jinho0928/install-k8s#step-1-cluster-reset-masterworker-%EA%B3%B5%ED%86%B5)
1. [(Master/Worker 공통) 패키지 삭제](https://github.com/jinho0928/install-k8s/blob/main/README.md#step-2-%EC%84%A4%EC%B9%98-%ED%8C%A8%ED%82%A4%EC%A7%80-%EC%82%AD%EC%A0%9C-masterworker-%EA%B3%B5%ED%86%B5)

## 설치 가이드
## Step0. 환경 설정 (Master/Worker 공통)
* 목적 : `k8s 설치 진행을 위한 os 환경 설정`
* 순서 : 
    * os hostname을 설정한다.
	```bash
	sudo hostnamectl set-hostname k8s-master
	```
    * hostname과 ip를 등록한다.
      * sudo vi /etc/hosts
	```bash
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

	172.22.5.2 k8s-master
	```
    * 방화벽(firewall)을 해제한다. 
	```bash
	sudo systemctl stop firewalld
	sudo systemctl disable firewalld
	```	
    * 스왑 메모리를 비활성화 한다. 
	```bash
	sudo swapoff -a
	```
    * 스왑 메모리 비활성화 영구설정
      * sudo vi /etc/fstab 
	```bash
	swap 관련 부분 주석처리
	# /dev/mapper/centos-swap swap                    swap    defaults        0
	```
    ![image](figure/fstab.PNG)
    * SELinux 설정을 해제한다. 
	```bash
	sudo setenforce 0
	sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
	```
	* (cri-o) 사용 전 환경 설정
	```bash
	sudo modprobe overlay
	sudo modprobe br_netfilter
	
	sudo cat << "EOF" | sudo tee -a /etc/sysctl.d/99-kubernetes-cri.conf
	net.bridge.bridge-nf-call-iptables  = 1
	net.ipv4.ip_forward                 = 1
	net.bridge.bridge-nf-call-ip6tables = 1
	EOF
	
	sudo sysctl --system
	```
	* (docker) 사용 전 환경 설정
	```bash
	sudo cat << "EOF" | sudo tee -a /etc/sysctl.d/k8s.conf
	net.bridge.bridge-nf-call-iptables  = 1
	net.ipv4.ip_forward                 = 1
	net.bridge.bridge-nf-call-ip6tables = 1
	EOF
	
	sudo sysctl --system
	```
    * xfs filesystem을 사용하는 경우, 'xfs_info /' 커맨드가 ftype=1이 출력되는지 확인한다.
    
## Step 1. cri-o 설치 및 설정 (Master/Worker 공통)
* 목적 : `k8s container cri-o runtime 설치`
* 순서 :
    * cri-o를 설치한다. 
     * (폐쇄망) 아래 주소를 참조하여 패키지 레포를 등록 후 crio를 설치한다.
        * https://github.com/tmax-cloud/hypercloud-install-guide/tree/master/Package#step-1-local-repository-%EA%B5%AC%EC%B6%95
	```bash
	sudo yum -y install cri-o
	sudo systemctl enable crio
	sudo systemctl start crio
	```
     * (외부망) crio 버전 지정 및 레포를 등록 후 crio를 설치한다.
	```bash
	VERSION=1.17
	sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/CentOS_7/devel:kubic:libcontainers:stable.repo
	sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:${VERSION}.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:${VERSION}/CentOS_7/devel:kubic:libcontainers:stable:cri-o:${VERSION}.repo
  
	sudo yum -y install cri-o
	sudo systemctl enable crio
	sudo systemctl start crio
	```	
    * cri-o 설치를 확인한다.
	```bash
	sudo systemctl status crio
	rpm -qi cri-o
	```
    ![image](figure/rpm.PNG)
* 비고 :
    * 추후 설치예정인 network plugin과 crio의 가상 인터페이스 충돌을 막기위해 cri-o의 default 인터페이스 설정을 제거한다.
	```bash
	sudo rm -rf  /etc/cni/net.d/100-crio-bridge.conf
 	sudo rm -rf  /etc/cni/net.d/200-loopback.conf
	``` 
    * crio.conf 내용을 수정한다. ( sudo vi /etc/crio/crio.conf )
      * plugin_dirs : "/opt/cni/bin" 추가
      * (폐쇄망) insecure_registries = ["{registry}:{port}"]
      * (폐쇄망) pause_image : "k8s.gcr.io/pause:3.1" 을 "{registry}:{port}/k8s.gcr.io/pause:3.1" 로 변경
      ![image](figure/crio_config.PNG)
    * pid cgroup의 max pid limit 설정이 필요한 경우 pids_limit 개수를 수정한다.
      * default : pids_limit = 1024
      * 시스템의 제한값인 `/proc/sys/kernel/pid_max`의 값 이하로 설정한다.
	```bash
	pids_limit = 32768
	```     
    * private image registry를 사용할 경우 registries.conf 내용을 수정한다.
      * sudo vi /etc/containers/registries.conf
	```bash
	unqualified-search-registries = ['registry.fedoraproject.org', 'registry.access.redhat.com', 'registry.centos.org', 'docker.io', '{registry}:{port}']
	ex) unqualified-search-registries = ['registry.fedoraproject.org', 'registry.access.redhat.com', 'registry.centos.org', 'docker.io', '172.22.5.2:5000']
	``` 	      
    * crio를 재시작 한다.
	```bash
	sudo systemctl restart crio
	```
	
## Step 1-1. docker-ce 설치 및 설정 (Master/Worker 공통)
* 목적 : `k8s container docker runtime 설치`
* 생성 순서 :
    * docker-ce를 설치한다. (image registry 구축 시에 docker를 이미 설치하였고 daemon.json 내용이 같다면 step1-1의 과정은 생략한다.)
    ```bash
    sudo yum install -y docker-ce
    sudo systemctl start docker
    sudo systemctl enable docker
    ```  
    * 폐쇄망 환경에서 private registry 접근을 위해 daemon.json 내용을 수정한다.
      * sudo vi /etc/docker/daemon.json
      ```bash
      {
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": { "max-size": "100m" },
        "storage-driver": "overlay2",
        "insecure-registries": ["{registry}:{port}"]
      }
      ```
    * docker를 재실행하고 status를 확인한다.
    ```bash
      sudo systemctl restart docker
      sudo systemctl status docker
    ```
* 비고 : 
    * private image registry를 사용할 경우 registries.conf 내용을 수정한다.
      * sudo vi /etc/containers/registries.conf
	```bash
	unqualified-search-registries = ['registry.fedoraproject.org', 'registry.access.redhat.com', 'registry.centos.org', 'docker.io', '{registry}:{port}']
	ex) unqualified-search-registries = ['registry.fedoraproject.org', 'registry.access.redhat.com', 'registry.centos.org', 'docker.io', '172.22.5.2:5000']
	``` 	      
    * docker를 재시작 한다.
	```bash
	sudo systemctl restart docker
	```
	
## Step 2. kubeadm, kubelet, kubectl 설치 (Master/Worker 공통)
* 목적 : `Kubernetes 구성을 위한 kubeadm, kubelet, kubectl 설치한다.`
* 순서:
    * CRI-O 메이저와 마이너 버전은 쿠버네티스 메이저와 마이너 버전이 일치해야 한다.
    * (폐쇄망) kubeadm, kubectl, kubelet 설치 (v1.17.6)
	```bash
	sudo yum install -y kubeadm-1.17.6-0 kubelet-1.17.6-0 kubectl-1.17.6-0
	
	sudo systemctl enable kubelet
	```  	
    * (외부망) 레포 등록 후 kubeadm, kubectl, kubelet 설치 (v1.17.6)
	```bash
	sudo cat << "EOF" | sudo tee -a /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=Kubernetes
	baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	EOF

	sudo yum install -y kubeadm-1.17.6-0 kubelet-1.17.6-0 kubectl-1.17.6-0
	
	sudo systemctl enable kubelet
	```  

## Step 3. Control Plane 구성 (Master)
* 목적 : `kubernetes master를 구축한다.`
* 순서 :
    * 쿠버네티스 설치시 필요한 kubeadm-config를 작성한다.
        * vi kubeadm-config.yaml
	```bash
	apiVersion: kubeadm.k8s.io/v1beta2
	kind: InitConfiguration
	localAPIEndpoint:
  		advertiseAddress: {api server IP}
  		bindPort: 6443
	nodeRegistration:
  		criSocket: /var/run/crio/crio.sock
	---
	apiVersion: kubeadm.k8s.io/v1beta2
	kind: ClusterConfiguration
	kubernetesVersion: {k8s version}
	controlPlaneEndpoint: {endpoint IP}:6443
	imageRepository: {registry}/k8s.gcr.io
	networking:
 		serviceSubnet: {SERVICE_IP_POOL}/{CIDR}
  		podSubnet: {POD_IP_POOL}/{CIDR}
	---
	apiVersion: kubelet.config.k8s.io/v1beta1
	kind: KubeletConfiguration
	cgroupDriver: systemd
	```
      * kubernetesVersion : kubernetes version
      * advertiseAddress : API server IP ( master IP )
        * 해당 master 노드의 IP
      * controlPlaneEndpoint : endpoint IP ( master IP or virtual IP) , port는 반드시 6443으로 설정
        * 1개의 마스터 : master IP , 2개 이상의 마스터 구축시 : virtual IP
      * serviceSubnet : ${SERVICE_IP_POOL}/${CIDR}
      * podSubnet : ${POD_IP_POOL}/${CIDR}
      * imageRepository : ${registry}/docker_hub_name
      * cgroupDriver: cgroup driver systemd 변경
      
     * kubeadm init (2개 이상 마스터 구축시에는 아래 가이드 참조)
	```bash
	sudo kubeadm init --config=kubeadm-config.yaml
	```
	![image](figure/kubeinit.PNG)
     * 마스터 다중화 진행시 아래 마스터 다중화 가이드로 설치를 진행한다. 
       * [마스터 다중화 가이드](https://github.com/tmax-cloud/hypercloud-install-guide/tree/master/K8S_Master#step-3-1-kubernetes-cluster-%EB%8B%A4%EC%A4%91%ED%99%94-%EA%B5%AC%EC%84%B1%EC%9D%84-%EC%9C%84%ED%95%9C-keepalived-%EC%84%A4%EC%B9%98)
     * 듀얼 스택 클러스터 구축 시에는 아래의 [비고] yaml을 참조하여 진행한다.                
    * kubernetes config 
	```bash
	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config
	```
	![image](figure/success.PNG)
    * 확인
	```bash
	kubectl get nodes
	```
	![image](figure/nodes.PNG)
	```bash
	kubectl get pods -A -o wide
	```
	![image](figure/pods.PNG)	
* 비고 : 
    * master에도 pod 스케줄을 가능하게 하려면 master taint를 제거한다
	```bash
	kubectl taint node [master hostname] node-role.kubernetes.io/master:NoSchedule-
	ex) kubectl taint node k8s- node-role.kubernetes.io/master:NoSchedule- 
	```
    * 듀얼 스택 클러스터 구축 시에는 아래의 kubeadm-config.yaml을 참조한다.
      * vi kubeadm-config.yaml
      ```bash
      apiVersion: kubeadm.k8s.io/v1beta2
      kind: InitConfiguration
      localAPIEndpoint:
      advertiseAddress: {api server IP}
  		bindPort: 6443
      nodeRegistration:
  		criSocket: /var/run/crio/crio.sock
      ---
      apiVersion: kubeadm.k8s.io/v1beta2
      kind: ClusterConfiguration
      kubernetesVersion: {k8s version}
      controlPlaneEndpoint: {endpoint IP}:6443
      imageRepository: {registry}/k8s.gcr.io
      networking:
 		serviceSubnet: ${SERVICE_IPV4_POOL}/${CIDR},${SERVICE_IPV6_POOL}/${CIDR}
  		podSubnet: ${POD_IPV4_POOL}/${CIDR},${POD_IPV6_POOL}/${CIDR}
      featureGates:
    		IPv6DualStack: true
      ---
      apiVersion: kubelet.config.k8s.io/v1beta1
      kind: KubeletConfiguration
      cgroupDriver: systemd
      ---
      apiVersion: kubeproxy.config.k8s.io/v1alpha1
      kind: KubeProxyConfiguration
      mode: ipvs
	```
     * kubernetesVersion : kubernetes version
     * advertiseAddress : API server IP ( master IP )
        * 해당 master 노드의 IP
     * controlPlaneEndpoint : endpoint IP ( master IP or virtual IP) , port는 반드시 6443으로 설정
        * 1개의 마스터 : master IP , 2개 이상의 마스터 구축시 : virtual IP
     * serviceSubnet : ${SERVICE_IPV4_POOL}/${CIDR},${SERVICE_IPV6_POOL}/${CIDR}
     * podSubnet : ${POD_IPV4_POOL}/${CIDR},${POD_IPV6_POOL}/${CIDR}
     * imageRepository : ${registry}/docker_hub_name
     * IPv6DualStack: true, dual stack 기능이 베타 버전이라서 기본값으로는 비활성화이기 때문에 활성화 시키는 세팅(추후 기본으로 활성화 되면 빠져야 함)
     * cgroupDriver: cgroup driver systemd 변경
     * mode: ipvs, dual stack 기능은 kube-proxy ipvs 모드에서만 동작
      
## Step 3-1. kubernetes cluster 다중화 구성을 위한 Keepalived 설치 (Master)
* 목적 : `K8S cluster의 Master 다중화 구성을 위한 Keepalived를 설치 및 설정한다`
* 순서 : 
    * Keepalived 설치
    ```bash
    sudo yum install -y keepalived
    ```
    * Keepalived 설정
    ```bash
	sudo vi /etc/keepalived/keepalived.conf
	
	vrrp_instance VI_1 {    
	state {MASTER or BACKUP}   
	interface {network interface}    
	virtual_router_id {virtual router id}    
	priority {priority}    
	advert_int 1    
	nopreempt    
	authentication {        
		auth_type PASS        
		auth_pass $ place secure password here.   
		}   
	virtual_ipaddress {        
		{VIP}  
		} 
	}
    ```	
    ![image](figure/keepalived.PNG)
	* interface : network interface 이름 확인 (ip a 명령어로 확인) ex) enp0s8
	* state : master or backup으로 설정, 하나의 master에만 master를 설정하고 나머지 master에는 backup으로 설정
	* priority : Master 우선순위  
	    * priority 값이 높으면 최우선적으로 Master 역할 수행
	    * 각 Master마다 다른 priority 값으로 수정
	    * ex) master1 priority 100, master2 priority 99, master3 priority 98 
	* virtual_ipaddress : virtual ip(VIP) 설정
	* virtual_router_id : vritual router id ex) 50
	
    * keepalived 재시작 및 상태 확인
    ```bash
    sudo systemctl restart keepalived
    sudo systemctl enable keepalived
    sudo systemctl status keepalived
    ```
	
    * network interface 확인
    ```bash
    ip a
    ```
    * 설정한 VIP 확인 가능, 여러 마스터 중 하나만 보임.
    * inet {VIP}/32 scope global eno1
    ![image](figure/ipa.PNG)
	
## Step 3-2. kubernetes cluster 다중화 구성 설정 
* 목적 : `K8S cluster의 Master 다중화를 구성한다`
* 순서 : 
    * kubeadm-config.yaml 파일로 kubeadm 명령어 실행한다.
        * Master 다중구성시 --upload-certs 옵션은 반드시 필요.
        * join 시에 --cri-socket=/var/run/crio/crio.sock 옵션을 추가하여 실행한다.
	    ```bash
	    sudo kubeadm init --config=kubeadm-config.yaml --upload-certs 
	    sudo kubeadm join {IP}:{PORT} --token ~~ discovery-token-ca-cert-hash --control-plane --certificate-key ~~ --cri-socket=/var/run/crio/crio.sock (1)
	    sudo kubeadm join {IP}:{PORT} --token ~~ discovery-token-ca-cert-hash --cri-socket=/var/run/crio/crio.sock (2)
	    ```
	![image](figure/master2.PNG)    
	* 해당 옵션은 certificates를 control-plane으로 upload하는 옵션
	* 해당 옵션을 설정하지 않을 경우, 모든 Master 노드에서 key를 복사해야 함
	* Master 단일구성과는 다르게, --control-plane --certificate-key 옵션이 추가된 명령어가 출력됨
	* (1)처럼 Master 다중구성을 위한 hash 값을 포함한 kubeadm join 명령어가 출력되므로 해당 명령어를 복사하여 다중구성에 포함시킬 다른 Master에서 실행
	* (2)처럼 Worker의 join을 위한 명령어도 출력되므로 Worker 노드 join시 사용, crio 사용시 --cri-socket 옵션 추가
	   ```bash
	     kubeadm join 172.22.5.2:6443 --token 2cks7n.yvojnnnq1lyz1qud \ --discovery-token-ca-cert-hash sha256:efba18bb4862cbcb54fb643a1b7f91c25e08cfc1640e5a6fffa6de83e4c76f07 \ --control-plane --certificate-key f822617fcbfde09dff35c10e388bc881904b5b6c4da28f3ea8891db2d0bd3a62 --cri-socket=/var/run/crio/crio.sock
	   ```
	* kubernetes config 
	    ```bash
	    mkdir -p $HOME/.kube
	    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	    sudo chown $(id -u):$(id -g) $HOME/.kube/config
	    ```
* 비고 : 	
    * join시에 apiserver ip 변경이 필요할 경우 --apiserver-advertise-address 옵션을 추가한다.
	```bash
	kubeadm join 172.22.5.2:6443 --token 2cks7n.yvojnnnq1lyz1qud \ --discovery-token-ca-cert-hash sha256:efba18bb4862cbcb54fb643a1b7f91c25e08cfc1640e5a6fffa6de83e4c76f07 \ --control-plane --certificate-key f822617fcbfde09dff35c10e388bc881904b5b6c4da28f3ea8891db2d0bd3a62 --cri-socket=/var/run/crio/crio.sock --apiserver-advertise-address=172.22.4.3
	```    
	
## Step 4. Cluster join (Worker)
* 목적 : `kubernetes cluster에 join한다.`
* 순서 :
    * kubernetes master 구축시 생성된 join token을 worker node에서 실행한다.
    * kubeadm join
       * (cri-o) join token에 --cri-socket=/var/run/crio/crio.sock 옵션을 추가하여 실행한다.
       ```bash
       kubeadm join 172.22.5.2:6443 --token r5ks9p.q0ifuz5pcphqvc14 \ --discovery-token-ca-cert-hash sha256:90751da5966ad69a49f2454c20a7b97cdca7f125b8980cf25250a6ee6c804d88 --cri-socket=/var/run/crio/crio.sock
       ```
       * (docker) join token을 그대로 실행한다.
       ```bash
       kubeadm join 172.22.5.2:6443 --token r5ks9p.q0ifuz5pcphqvc14 \ --discovery-token-ca-cert-hash sha256:90751da5966ad69a49f2454c20a7b97cdca7f125b8980cf25250a6ee6c804d88
       ```	
    ![image](figure/noding.PNG)
* 비고 : 
    * kubeadm join command를 저장해놓지 못한 경우, master node에서 아래 명령어를 통해 token 재생성이 가능하다.
	```bash
	kubeadm token create --print-join-command
	```	
    
## 삭제 가이드
## Step 1. Cluster reset (Master/Worker 공통)
* 목적 : `kubernetes cluster를 reset한다.`
* 순서 :
    * kubeadm reset
       * (cri-o) --cri-socket=/var/run/crio/crio.sock 옵션을 추가하여 kubeadm reset을 실행한다.
       ```bash
       kubeadm reset --cri-socket=/var/run/crio/crio.sock
       ```
       * (docker) kubeadm reset을 실행한다.
       ```bash
       kubeadm reset
       ```

## Step 2. 설치 패키지 삭제 (Master/Worker 공통)
* 목적 : `kubernetes에 필요한 패키지들을 삭제한다`
* 순서 :
    * 설치했던 패키지들을 삭제 한다.
      ```bash
      sudo yum remove -y kubeadm-1.17.6-0 kubelet-1.17.6-0 kubectl-1.17.6-0
      
      sudo yum remove -y crio  or  sudo yum remove -y docker-ce
      
      sudo yum remove -y keepalived
      ```
