# 1. Deployment
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

# 2. Rolling update
  - kubectl set --help
  - kubectl set image deployment `<deploy_name> <container_name>=<new_version_image>`
  - 신규 1개 생성 => 기존 1개 제거 (반복)
 
## 실행
```
kubectl delete -f deployment-exam1.yaml
kubectl create -f deployment-exam1.yaml --record
kubectl set image deployment app-deploy web=nginx:1.15 --record
kubectl describe pod app-deploy-85766b486d-rn64j | grep -i 'nginx' #nginx 1.15로 변경확인
kubectl rollout status deployment app-deploy

Waiting for deployment "app-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "app-deploy" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "app-deploy" rollout to finish: 1 old replicas are pending termination...
deployment "app-deploy" successfully rolled out

```

## record 
  - --record 옵션으로 기록
  - status / history / pause / resume 가능
  - spec.revisionHistoryLimit: 10 으로 # history 보관 수 결정 가능
  - spec.progressDeadlineSeconds: 600 # 600초 이상인경우 롤백 됨
  - spec.strategy.rollingUpdate.maxSurge: 25% 
  - kubectl rollout status deployment `<name>` 
  - kubectl rollout history deployment `<name>` 
  - kubectl rollout pause/resume deployment `<name>`

## pod 삭제
  - 지워도 replica에 의해 계속 재생성됨
  - ReplicationController, ReplicaSet, Deployment, static(/var/lib/kubelet/config.yaml 에서 확인)를 삭제해야 함
  - 상위만 삭제하고 pod를 놔둘경우 --cascade=false 옵션 추가

# 3. Rolling back
  - kubectl rollout --help
  - kubectl rollout restart deploy app-deploy # 롤링된 결과를 재시작
  - kubectl rollout undo deploy app-deploy # 이전 revision으로 롤백
  - kubectl rollout undo deploy app-deploy --to-revision=3 # history 3번으로 롤백
  - history는 가장 마지막 번호가 현재 상태

```
kubectl rollout history deploy app-deploy
1         kubectl create --filename=deploymen-exam1.yaml --record=true
2         kubectl set image deployment app-deploy web=nginx:1.15 --record=true
3         kubectl set image deployment app-deploy web=nginx:1.16 --record=true
4         kubectl set image deployment app-deploy web=nginx:1.17 --record=true
5         kubectl set image deployment app-deploy web=nginx:1.18 --record=true
kubectl rollout undo deploy app-deploy
kubectl rollout history deploy app-deploy
1         kubectl create --filename=deploymen-exam1.yaml --record=true
2         kubectl set image deployment app-deploy web=nginx:1.15 --record=true
3         kubectl set image deployment app-deploy web=nginx:1.16 --record=true
5         kubectl set image deployment app-deploy web=nginx:1.18 --record=true
6         kubectl set image deployment app-deploy web=nginx:1.17 --record=true
kubectl rollout undo deploy app-deploy --to-revision=3
kubectl rollout history deploy app-deploy
1         kubectl create --filename=deploymen-exam1.yaml --record=true
2         kubectl set image deployment app-deploy web=nginx:1.15 --record=true
5         kubectl set image deployment app-deploy web=nginx:1.18 --record=true
6         kubectl set image deployment app-deploy web=nginx:1.17 --record=true
7         kubectl set image deployment app-deploy web=nginx:1.16 --record=true
kubectl rollout undo deploy app-deploy
kubectl rollout history deploy app-deploy
REVISION  CHANGE-CAUSE
1         kubectl create --filename=deploymen-exam1.yaml --record=true
2         kubectl set image deployment app-deploy web=nginx:1.15 --record=true
5         kubectl set image deployment app-deploy web=nginx:1.18 --record=true
7         kubectl set image deployment app-deploy web=nginx:1.16 --record=true
8         kubectl set image deployment app-deploy web=nginx:1.17 --record=true
```
=> undo는 바로 직전 번호로 바뀜, 한번더 한다면? 7번이 9번으로 변경되고 맨 밑으로 내려와 현재 상태가 됨
=> undo를 계속 반복하면 1.16, 1.17만 반복적으로 바뀜
