# Kubernetes 실습환경구성

* 장애발생 시, 아래 블로그 참고
* https://yooloo.tistory.com/203?category=0

**Single Master, Multi node Kubernets** 환경을 구성합니다.

|HOST|IP address  | arch | CPU | Memory | OS |
|--|--|--|--|--|--|
|master.example.com|10.100.0.104|X86_64|2core|4GiB |CentOS 7.6|
|node1.example.com|10.100.0.101|X86_64|2core|2GiB |CentOS 7.6|
|node2.example.com|10.100.0.102|X86_64|2core|2GiB |CentOS 7.6|
|node3.example.com|10.100.0.103|X86_64|2core|2GiB |CentOS 7.6|


## 1. Docker Install
master, node1,node2, node3 시스템에 도커 설치

[docs.docker.com](https://docs.docker.com/engine/install/centos/)

    # yum install -y yum-utils
    # yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    # yum install docker-ce docker-ce-cli containerd.io
    # systemctl start docker && systemctl enable docker
    # docker version
	    Client: Docker Engine - Community
		 Version:           19.03.8
		 API version:       1.40
		 Go version:        go1.12.17
		 Git commit:        afacb8b
		 Built:             Wed Mar 11 01:27:04 2020
		 OS/Arch:           linux/amd64
		 Experimental:      false

		Server: Docker Engine - Community
		 Engine:
		  Version:          19.03.8
		  API version:      1.40 (minimum version 1.12)
		  Go version:       go1.12.17
		  Git commit:       afacb8b
		  Built:            Wed Mar 11 01:25:42 2020
		  OS/Arch:          linux/amd64
		  Experimental:     false
		 containerd:
		  Version:          1.2.13
		  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
		 runc:
		  Version:          1.0.0-rc10
		  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
		 docker-init:
		  Version:          0.18.0
		  GitCommit:        fec3683

    
## 2. kubeadm, kubectl, kubelet 설치 동작
master, node1,node2, node3 시스템에 kubeadm, kubectl, kubelet 설치 및 동작

[kubernetes.io](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

	1) Swap disabled
	# swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab
	
	2) Letting iptables see bridged traffic
	# cat <<EOF > /etc/sysctl.d/k8s.conf
	net.bridge.bridge-nf-call-ip6tables = 1
	net.bridge.bridge-nf-call-iptables = 1
	EOF
	# sysctl --system

	3) Disable firewall
	# systemctl stop firewalld 
	# systemctl disable firewalld
	
	4) Set SELinux in permissive mode (effectively disabling it)
	# setenforce 0
	# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
	
	5) kubeadm, kubelet, kubectl 설치
	# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=Kubernetes
	baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	exclude=kubelet kubeadm kubectl
	EOF
	
	# yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
	# systemctl start kubelet && systemctl enable kubelet

## add. 도커 대몬 교체(쿠버네티스와 도커의 cgroup 드라이버가 동일해야 함)
	...
	cat > /etc/docker/daemon.json <<EOF
	{
	  "exec-opts": ["native.cgroupdriver=systemd"],
	  "log-driver": "json-file",
	  "log-opts": {
	    "max-size": "100m"
	  },
	  "storage-driver": "overlay2"
	}
	EOF

	$ sudo mkdir -p /etc/systemd/system/docker.service.d
	$ sudo systemctl daemon-reload
	$ sudo systemctl restart docker
	...
## add. vitualbox의 node들 ip 세팅 필요(NAT 네트워크)
https://coffeewhale.com/kubernetes/cluster/virtualbox/2020/08/31/k8s-virtualbox/

## add. 초기화
	$ systemctl stop kubelet
	$ docker rm -f $(docker ps -aq)
	$ docker rmi -f $(docker images -q)
	$ systemctl restart docker
	$ systemctl start kubelet
	$ rm -f  ~/.kube/config
	$ kubeadm reset

## 3. Master 컴포넌트 구성및 네트워크 환경구성
master 시스템에서만 구성 
### Creating a single control-plane cluster with kubeadm

	# kubeadm init
	...
	To start using your cluster, you need to run the following as a regular user:

	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config
	...
	kubeadm join 10.100.0.104:6443 --token 1ou05o.kkist9u6fbc2uhp3 --discovery-token-ca-cert-hash sha256:8d9a7308ea6ff73.........576c112f326690

**kubeadm init** 명령실행시 master에 kubernetes 컴포넌트들이 생성된다. 
출력 결과에서 위의 내용중 mkdir, cp, chown 명령을 kubernetes 관리자 계정에서 실행한다.

	$ mkdir -p $HOME/.kube
	$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

**kubeadm init** 명령 실행 결과 나오는 kubeadm join <토크> 은  worker node가 master에 연결될때 필요하다. 별도로 저장해두는것이 좋다.

	# cat > token.tx
	kubeadm join 10.100.0.104:6443 --token 1ou05o.kk...3 --discovery-token-ca-cert-hash sha256:8d9a7308ea6ff73.........576c112f326690
	<Ctrl>+<d>

### Installing a Pod network add-on
Weave Net works
	# kubectl delete -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

	# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
	
	# kubectl get nodes
=> 안될경우, # kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml (추천)
## 4. Worker Node 구성
node1, node2, node3에서 실행
master의 **kubeadm init** 명령 실행시 출력된 토큰을 가지고 마스터와 연결

	# kubeadm join 10.100.0.104:6443 --token 1ou05o.kk...3 --discovery-token-ca-cert-hash sha256:8d9a7308ea6ff73.........576c112f326690

마스터 노드에서 kubectl 명령으로 쿠버네티스 상태 확인

	# kubectl get nodes
	NAME                 STATUS   ROLES    AGE   VERSION
	master.example.com   Ready    master   10m   v1.18.0
	node1.example.com    Ready    <none>   17m   v1.18.0
	node2.example.com    Ready    <none>   17m   v1.18.0
	node3.example.com    Ready    <none>   17m   v1.18.0	
	
## 5. bash shell에서 [TAB]키를 이용해 kubernetes command 자동완성 구성하기
	
	# source <(kubectl completion bash)
	# echo "source <(kubectl completion bash)" >> ~/.bashrc
	# source <(kubeadm completion bash)
	# echo "source <(kubeadm completion bash)" >> ~/.bashrc

## add. 성공사례
	1. kubeadm init --apiserver-advertise-address=10.0.1.4 --pod-network-cidr=10.244.0.0/16 
	2. kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
	3. cat > example.yaml 
	4. kubectl create -f example.yaml
	5. kubectl get pods -o wide (node ip 확인하고)
	6. curl 10.244.1.2 (nginx 응답 확인 ok!!) 
	7. 앞선 실패 원인이 mac주소 중복일 수 있음(virtualbox 네트워크에서 변경 가능)
	 => MAC 주소 및 product_uuid가 모든 노드에 대해 고유해야 하며
            (ifconfig -a MAC 주소를 확인, product_uuid는 sudo cat /sys/class/dmi/id/product_uuid 확인)
	    
## add. deployment
   1. run은 1개 실행
   2. create deployment를 활용하여 관리 가능
   
      	
	# kubectl create deployment mainui --image=httpd --replicas=3 deployment로 생성
	# kubectl get deployments.apps deploy로 축약가능
	# kubectl get pods -o wide 자세히보기 
	# kubectl get pod mainui-77fc86948f-pctm8 -o json json형식으로 보기
	# kubectl get pod mainui-77fc86948f-pctm8 -o yaml yaml형식으로 보기
	# kubectl exec deploy-exam-8f458dc5b-khx7h -it -- /bin/bash 안에 들어가서 수정하기 
	# kubectl port-forward deploy-exam-8f458dc5b-khx7h 8081:80 port 변경
	# kubectl logs deploy-exam-8f458dc5b-khx7h log확인
	# kubectl edit deployments.apps mainui 실행중인 오브젝트 수정
	# kubectl run webserver --image=nginx:1.14 --port 80 --dry-run=client -o yaml > webserver-pod.yaml 
	  (dry-run은 동작확인만, -o yaml 형태로, > 를 이용하여 저장)
	# kubectl delete pod webserver 실행중인 pod 삭제
	# kubectl create -f webserver-pod.yaml

## 6. 실습 : 간단한 yaml 파일을 생성해서 nginx를 배포해보자.
	1. 아래와 같은 example.yaml을 생성한다.
	$ cat > example.yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: deploy-exam
	spec:
	  replicas: 1
	  selector:
	    matchLabels:
	      app: nginx
	  template:
	    metadata:
	      labels:
	        app: nginx
	    spec:
	      containers:
	      - name: nginx
	        image: nginx

	2. 앞에서 생성한 yaml을 이용해 웹서버 nginx를 실행한다.
	# kubectl create -f example.yaml

	3. 동작중인 deploy를 확인
	# kubectl get deploy

	4. depoy의 child로 동작되는 replicaset과 pod를 확인한다.
	# kubectl get rs
	# kubectl get pod

	Pod 확장하기
	5. 다음 명령을 사용하여 Pod 수를 5개로 확장하자:
	$ kubectl scale deploy deploy-exam --replicas=5
	$ kubectl get pod
	$ kubectl get pod -o wide

	4. 생성된 deploy를 제거한다.
	$ kubectl delete deploy deploy-exam 
