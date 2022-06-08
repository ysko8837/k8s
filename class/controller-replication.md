# Replication controller 
  - https://github.com/237summit/Getting-Start-Kubernetes.git 의 6강좌 참고
  - pod개수를 보장
  - 부족할 경우 template을 이용해 pod를 추가, 많으면 `최근 생성된 Pod`를 삭제
  - 기본구성 : `selector`, `replicas`, `template`
  - selector의 key:value는 template.metadata.labels로 반드시 포함하고 있어야 함(key:value)
  - selector의 app: mainui를 replicas수만큼 운영, 없으면 template을 이용해서 pod를 만들어서 
  
## deploy를 통한 replica  
```
# kubectl edit deploy xxx
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

## ReplicationController를 통한 replica 
### rc로 생성
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
```
cat > rc-nginx.yaml    # 위 내용으로 생성
kubectl create -f rc-nginx.yaml
kubectl describe rc rc-nginx  # kind가 ReplicationController(rc) 이므로
```

### controller 특징
```
kubectl run redis --image=redis --labels=app=webui --dry-run=client -o yaml > redis.yaml
vi redis.yaml
kubectl create -f redis.yaml # webui라는 redis 생성 명령
kubectl get pods --show-labels  # 라벨이 webui로 같으므로 rc는 redis를 생성하지 못함(rc는 selector만 확인하고 replica수가 맞아서 생성안됨)
```

### scale out/down
```
kubectl edit rc rc-nginx                # 방법 1
kubectl scale rc rc-nginx --replicas=2  # 방법 2  
```


### 롤링 업데이트
```
kubectl edit rc rc-nginx  # template의 nginx 버전을 1.15로 변경
kubectl get pods -o wide  # pod name 확인
kubectl edit pod rc-nginx-spk2s # nginx 버전 확인(1.14로 유지되며 변경안됨)
kubectl delete pod rc-nginx-spk2s # 해당 pod 삭제
kubectl get pods -o wide  # 재생성된 pod name 확인
kubectl edit pod rc-nginx-7zmz6 # 재생성된 pod의 nginx 버전 확인(1.15로 변경됨)
```
