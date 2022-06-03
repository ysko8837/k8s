# init container pod
  - 앱 컨테이너 실행 전에 미리 동작시킬 컨테이너
  - pod안에 init과 main container가 있을때, init container가 성공해야 main container가 동작

## step1
```
# cat > init-container-exam.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```
```
# kubectl create -f init-container-exam.yaml
# kubectl get pods -o wide (myservice가 계속 반복 동작하여 mydb, myapp이 동작하지 않음)
NAME        READY   STATUS     RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
myapp-pod   0/1     Init:0/2   0          4m55s   10.244.2.33   node2   <none>           <none>
```

## step 2
```
# cat > init-container-exam-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```
```
# kubectl create -f init-container-exam-svc.yaml
# kubectl get pods -o wide (위 결과로 하나만 동작하여 대기 상태)
NAME        READY   STATUS     RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
myapp-pod   0/1     Init:1/2   0          4m55s   10.244.2.33   node2   <none>           <none>
```

## step3
```
# cat > init-container-exam-db.yaml
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```
```
# kubectl create -f init-container-exam-db.yaml
# kubectl get pods (위 결과로 두개 모두 동작하여 main도 동작)
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          10m
```
