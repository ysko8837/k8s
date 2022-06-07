# Replication controller 
  - pod개수를 보장
  - 부족할 경우 template을 이용해 pod를 추가, 많으면 `최근 생성된 Pod`를 삭제
  - 기본구성 : selector, replicas, template
  
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