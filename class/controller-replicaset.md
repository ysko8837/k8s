# ReplicaSet
 - replicationController와 같은 역할을 하는 컨트롤러
 - 보다 풍부한 label selector 지원
 - expression-operator : In, NotIn, Exists, DoseNotExists
 - apiVersion: apps/v1
 - kind: ReplicaSet

## replication controller
```
# app: webui 이고 version: "2.1"인 container를 3개 보장 해줘
spec:
  replicas: 3
  selector:
    app: webui
    version: "2.1"
  temp...
```

## ReplicaSet
```
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui      
    matchExpressions:
    - {key: version, operator: In, value:["2.1"]} # 위의 replication controller와 동일
    - {key: version, operator: In, value:["2.1","2.2"]} # app: webui이고 version이 2.1이거나 2.2인 경우
    - {key: version, operator: NotIn, value:["2.1","2.2"]} # app: webui이고 version이 2.1이거나 2.2이 아닌 경우
    - {key: version, operator: Exists} # app: webui이고 version이 존재한 경우
    - {key: version, operator: DoseNotExists} # app: webui이고 version이 존재하지 않는 경우
  temp...
```

## history
  - controller 삭제/생성에 다른 어플리케이션이 묶일 수 있으니, label과 selector 이름은 신중히 결정
  - label을 여러개 묶어서 쓰는게 좋음
```
kubectl delete rc --all # rc삭제하면 pod까지 같이 삭제됨
kubectl get pods
cat > rs-nginx.yaml
kubectl create -f rs-nginx.yaml
kubectl get pod --show-labels
kubectl get rs
kubectl delete pod rs-nginx-c6rmw
kubectl get pod  #자동 복구되어 3개
kubectl scale rs rs-nginx --replicas=2 #2개로 변경
kubectl delete rs rs-nginx --cascade=false # pod삭제없이 controller만 삭제 ( --cascade=orphan 으로 변경됨)
kubectl get pods # rs-nginx2개 확인됨(controller없는 단독 pod가 됨)
kubectl get pod --show-labels #2개 확인
kubectl create -f rs-nginx.yaml # 기존 2개에 하나가 추가 생성되고, 3개의 pod가 모두 controller에 의해 다시 관리됨
kubectl get pods --show-labels #3개 확인
```
