chapter4 참조

# pod 생성
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

# multi-container pod 생성
```
kubectl create -f pod-multi.yaml  #chapter4의 pod-multi.yaml 참고
kubectl describe pod multipod  #상세 실행 내역 확인
kubectl get pods -o wide   #ip확인
curl 10.244.2.15  #동작 확인
kubectl exec multipod -c nginx-container -it -- /bin/bash  #해당 pod의 nginx-container라는 container에 bash명령 전달
=> cd /usr/share/nginx/html/  
=> echo "TEST web" > index.html  #welcome page 수정
=> exit
curl 10.244.2.15  #수정내역 확인
kubectl exec multipod -c centos-container -it -- /bin/bash  #해당 pod의 centos로 접근
=> ps -ef #os의 process확인
=> curl localhost #결과는 성공(같은 ip를 같고 있으므로 같은 pod내의 nginx 호출됨)
=> exit
kubectl logs multipod -c nginx-container  #해당 pod의 nginx-container라는 container의 로그를 확인
```

