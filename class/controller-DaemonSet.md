# DaemonSet
  - 전체 노드에서 pod가 하나씩 실행되도록 보장
  - 특정 노드에서 문제 생길경우, 완전 삭제된 후 생성 됨(롤링 업데이트 됨)
  - 로그수집기, 모니터링 에이전트 같은 프로그램 실행 시 적용(scouter 등)
  - replicaset과는 replica수 지정이 없고, kind만 다름
  - https://github.com/237summit/Getting-Start-Kubernetes/blob/main/6/daemonset-exam.yaml 참고

## 실행
```
cat > daemonset-exam.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
spec:
  selector:
    matchLabels:
      app: webui
  template:
    metadata:
      name: nginx-pod
      labels:
        app: webui
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```

```
kubectl create -f daemonset-exam.yaml
kubectl get pod
kubectl edit daemonsets.apps daemonset-nginx # 버전 변경 시, 롤링 업데이트 됨
kubectl rollout undo daemonset daemonset-nginx # 롤백
kubectl describe pod daemonset-nginx-4292j # 롤백 확인
```

## token 관리
  - kubeadm token list
  - kubeadm token delete
  - kubeadm token create --ttl 1h (1시간짜리)
  - kubeadm join ..... 
