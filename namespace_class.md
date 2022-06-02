# defalut namespace 변경(namespace 삭제 시, 하위 모든 요소가 삭제됨을 유의!)
주 작업을 default namespace로 변경하면 -n [namespace name] 없이 작업 가능

      kubectl config --help
      kubectl config view
      kubectl config set-context blue@kubernetes --cluster=kubernetes --user=kubernetes-admin --namespace=bule #같은 형식으로 context 생성
      kubectl config view 
      kubectl config current-context
      kubectl config use-context blue@kubernetes #변경
      kubectl delete pods mypod -n default #default namespace의 pod 삭제
