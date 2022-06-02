# namespace
 작업 공간을 세분화하여 독립적으로 사용 가능(prod/dev, homepage/admin 등으로 구분)
 
      kubectl get namespaces
      kubectl get pods -n default
      kubectl get pods -n kube-system
      kubectl create -f nginx_run.yaml -n blue #bule에 nginx 생성
      
## namespace 생성/삭제      
      kubectl create namespace blue  # cli로 생성
      kubectl create namespace orange --dry-run -o yaml > orange-ns.yaml # yaml로 생성
      kubectl create -f orange-ns.yaml
      kubectl delete namespace blue  #namespace 삭제 시, 하위 모든 요소가 삭제됨을 유의!
      kubectl delete pods mypod -n default                  #default namespace의 pod 삭제
      
## defalut namespace 변경
 주 작업을 default namespace로 변경하면 -n [namespace name] 없이 작업 가능
 
      kubectl config --help
      kubectl config view # context 확인용
      kubectl config set-context blue@kubernetes --cluster=kubernetes --user=kubernetes-admin --namespace=bule #같은 형식으로 context 생성
      kubectl config view 
      kubectl config current-context                        #default 사용중인 context 확인
      kubectl config use-context blue@kubernetes            #context 변경(namespace도 변경됨)
      kubectl delete namespaces blue                        #blue namespace 삭제
      
