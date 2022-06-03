# livenessProbe (self healing)
  - 3회 시도 후, container를 새로 받고 container를 재시작(pod재시작이 아님, 따라서 ip유지)
  - 3개(httpGet, tcpSocket, exec)의 Probe 지원
  - chapter4 yaml 참고
 
 
 ```
 # default option
 initialDelaySeconds: 15
 periodSeconds: 20
 timeoutSeconds: 1
 successThreshold: 1
 failureThresHold: 3
 ```
 
```
# create yaml에 liveness 지정 (spec.containers.image 위치에 아래 추가)
    livenessProbe:      
      httpGet:
        path: /
        port: 80
```

```
# liveness example(5회까지는 200, 이후 500 리턴하는 서비스)
watch kubectl get pods -o wide
kubectl create -f pod-liveness.yaml
kubectl describe pod liveness-pod
```
