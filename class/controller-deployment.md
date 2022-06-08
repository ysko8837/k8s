# Deployment
  - rolling update 목적
  - replicaSet을 컨트롤해서 pod수를 조절
  - kind가 Deployment인 것만 빼고 ReplicaSet과 100동일하게 사용가능

## deploy-nginx.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 3
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

## 실행
```
# deployment가 replicaset을 통해 pod를 생성
# deploy-nginx : deployment 이름, deploy-nginx-6d75c5dd9b : replicaset 이름, deploy-nginx-6d75c5dd9b-p56f2 : pod 이름
kubectl create -f deploy-nginx.yaml
kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS        AGE     IP            NODE    NOMINATED NODE   READINESS GATES
deploy-nginx-6d75c5dd9b-7f4lm   1/1     Running   0               4m52s   10.244.1.65   node1   <none>           <none>
deploy-nginx-6d75c5dd9b-cp6lh   1/1     Running   0               4m52s   10.244.2.64   node2   <none>           <none>
deploy-nginx-6d75c5dd9b-p56f2   1/1     Running   0               4m52s   10.244.2.65   node2   <none>           <none>

kubectl delete pod deploy-nginx-6d75c5dd9b-p56f2 # pod 삭제 (replicaset을 통해 자동 재생성됨)
kubectl delete rs deploy-nginx-6d75c5dd9b # replicaset 삭제 (deployment를 통해 자동 재생성됨)
```

# record 
  - --record 옵션으로 기록
  - kubectl rollout history deployment <name> 으로 확인

# Rolling update
  - kubectl set --help
  - kubectl set image deployment <deploy_name> <container_name>=<new_version_image>
 
## 
```

kubectl set image deployment <deploy_name> <container_name>=<new_version_image>
```

# pod 삭제
  - 지워도 replica에 의해 계속 재생성됨
  - ReplicaController, ReplicaSet, Deployment, static(/var/lib/kubelet/config.yaml 에서 확인)를 삭제해야 함
  - 상위만 삭제하고 pod를 놔둘경우 --cascade=false 옵션 추가

