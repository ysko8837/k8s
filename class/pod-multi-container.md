chapter4 참조
# 요약
1. multi-container는 동일한 pod내에 두개 이상의 컨테이너가 존재(yaml에 이름 등 두개이상 생성)
2. multi인 경우, -c옵션을 통해 특정 컨테이너를 지정하여 명령 수행
3. ip역시 동일함
4. 리눅스의 watch를 통해 2초마다 확인하거나, --watch 옵션을 통해 변경사항 라인별로 확인가능
5. grep -i xxx 옵션으로 특정 부분만 확인 가능

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


# pod flow 이해
- 사전작업
``` 
watch kubuctl get pods -o wide #2초마다 확인
kubectl delete pod --all #기존 pod 모두 삭제
kubectl create -f nginx_run.yaml #nginx pod 실행
```

- pod 
``` 
kubectl get pods -o wide --watch  #동작중인 상태를 자세하게 보여줌
kubectl delete pod nginx-pod #삭제를 다른 터미널에서 실행하면, 위의 watch에 단계별로 정보 표시
```
- 5-1-2강좌 exam
```
  464  kubectl get pods
  465  kubectl get pods --all-namespaces
  466  kubectl run nginx-pod --image=nginx:1.14 --port=80
  467  kubectl get pods -o wide
  473  kubectl get pods nginx-pod -o yaml | grep -i image
  474  kubectl get pods -o wide
  475  kubectl describe pod nginx-pod
  476  kubectl get pods
  477  kubectl delete pod nginx-pod  
  479  kubectl run redis123 --image=redis --dry-run=client -o yaml > redis.yaml
  480  cat redis.yaml
  481  vi redis.yaml
  482  kubectl create -f redis.yaml
  483  kubectl get pods  
  485  kubectl edit pod redis123  
```
