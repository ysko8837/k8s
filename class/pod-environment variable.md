#pod 환경변수
  - pod내의 컨테이너가 실행될 때 필요로 하는 변수
  - pod 실행 시 미리 정의된 컨테이너 환경변수를 변경할 수 있음


```
# yaml에 환경변수 지정
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - image: nginx:1.14
    name: nginx-container
    ports:
    - containerPort: 80
      protocol: TCP
    env:
    - name: MYVAR
      value: "testvalue"
    - name: MYVAR2
      value: "testvalue2"
```

```
# env 확인(위에 정의한 환경변수 확인 가능)
kubectl exec nginx-pod -it -- /bin/bash
=> env
```
