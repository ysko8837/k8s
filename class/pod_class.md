# pod
```
kubectl run web1 --image=nginx:1.14 --port=80 -n orange  #cli 를 통한 pod 생성
kubectl config use-context orange@kubernertes            # default 변경
kubectl config current-context 
kubectl get pods
cat nginx_run.yaml                                       #yaml을 통한 pod 생성
kubectl create -f nginx_run.yaml 
kubectl get pods web1 -o yaml                            #생성된 정보를 통한 template관리(안에 불필요 요소 삭제 하고 사용)
kubectl get pods web1 -o json | grep -i podip            #특정 부분만 확인 가능

```

