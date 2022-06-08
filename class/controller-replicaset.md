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
