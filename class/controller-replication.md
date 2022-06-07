# Replication controller 
  - https://github.com/237summit/Getting-Start-Kubernetes.git의 6강좌 참고
  - pod개수를 보장
  - 부족할 경우 template을 이용해 pod를 추가, 많으면 `최근 생성된 Pod`를 삭제
  - 기본구성 : selector, replicas, template
  - selector의 key:value는 template.metadata.labels로 반드시 포함하고 있어야 함(key:value)
  
```
# kubectl edit deploy xxx (deploy를 통한 replica)
apiVersion: apps/v1
kind: Deployment
spec:
  progressDeadlineSeconds: 600
  replicas: 2  
  selector:
    matchLabels:
      app: mainui
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mainui
```

```
# rc-nginx.yaml replicationController를 통한 replica
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
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
