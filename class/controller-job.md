# job controller
  - 쿠버네티스는 pod를 running상태로 유지하려고 함
  - batch 처리는 작업이 완료되면 종료되어야 하고, 실패하면 재시도 해야함
  - job controller는 이름 지원하여 성공적인 완료를 보장
  - `spec.restartPolicy`에 OnFailure, Never 가 가능





















## 쿠버네티스 동작 원리 
```
kubectl run tespod --image=centos:7 --command sleep 5 
watch kubectl get pods -o wide
NAME      READY   STATUS      RESTARTS      AGE   IP             NODE    NOMINATED NODE   READINESS GATES
testpod   0/1     Completed   2 (22s ago)   34s   10.244.2.103   node2   <none>           <none>
# 컨테이너 내부에서 5초간 슬립되고 종료됨
# 쿠버네티스는 다시 pod를 실행함
```
