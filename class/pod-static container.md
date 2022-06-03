# static pod
- 동작 명령 시, scheduler에 의해 node가 선택되고 api에 의해 동작되는게 일반적
- static pod는 실행할 노드에서 api없이 kubelet에 의해 동작
- staticPodPath 디렉토리에 yaml파일을 넣으면, 해당 노드에 pod가 생성됨(yaml파일을 지우면 pod가 삭제됨)


```
# kubernetes가 설치된 모든(master포함) 노드에 아래 파일 존재
 cat /var/lib/kubelet/config.yaml
  ...
  staticPodPath: /etc/kubernetes/manifests
  ...
 
```

```
# 해당 위치에 yaml 파일이 생성되면, 즉시 pod가 생성됨(use-context로 지정한 namespace가 아닌 default에 생성됨)
 cd /etc/kubernetes/manifests
 cat > nginx.yaml
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
kubectl get pods
```
