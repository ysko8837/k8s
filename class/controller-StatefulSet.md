# StatefulSet
  - pod의 상태(이름, 저장소)를 보존해주는 controller
  - 지금까지의 controller(rc,rs,deploy,daemonset)는 pod의 상태를 보존하지 않음
  - replicaset에 spec.serviceName이 반드시 필요함
  - https://github.com/237summit/Getting-Start-Kubernetes/blob/main/6/statefulset-exam.yaml 참고
  - 일반적인 controller는 spec.podManagementPolicy: OrderedReady가 default 
  - statefulset 은 spec.podManagementPolicy: Parallel 
  - 롤링 업데이트 및 롤백 됨

```
cat > statefulset-exam.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx
spec:
  replicas: 3
  serviceName: sf-service
#  podManagementPolicy: OrderedReady
  podManagementPolicy: Parallel
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
## pod의 이름을 보존(저장소는 별도 처리 필요하며, 현재는 자유롭게 저장소 사용중)
```
kubectl create -f statefulset-exam.yaml
kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
sf-nginx-0   1/1     Running   0          116s
sf-nginx-1   1/1     Running   0          116s
sf-nginx-2   1/1     Running   0          116s

kubectl scale statefulset sf-nginx --replicas=4
kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
sf-nginx-0   1/1     Running   0          3m10s
sf-nginx-1   1/1     Running   0          6s
sf-nginx-2   1/1     Running   0          3m10s
sf-nginx-3   1/1     Running   0          80s

kubectl edit statefulset sf-nginx # 버전업하면 롤링 업데이트
kubectl rollout undo statefulset sf-nginx # 롤백
kubectl delete statefulset sf-nginx # 삭제
```
