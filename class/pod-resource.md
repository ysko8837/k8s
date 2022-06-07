# resource(cpu, memory) 할당
  - resource requests : 파드 실행을 위한 최소 리소스 양을 요청
  - resource limits : 파드가 사용할 수 있는 최대 리소스 양을 제한(초과되면 파드는 종료되며 다시 스케줄링 됨)
  - node에 자원이 부족하면 pending상태로 대기(여유 자원이 생기면 자동 생성)
  - memory unit : `1Gib = 1024Mib = 1024*1024Kib` => Gi, Mi, Ki 사용 (참고 1Mb = 1000kb)
  - cpu unit : `1 = 1000m` => 코어수 또는 밀리코어(m)으로 사용
  - `spec.containers`에 작성
  
```
# limit과 requests 할당
spec:
  containers:
  - image: nginx:1.14    
    ...
    resources:
      limits:
        cpu: 1
        memory: 1Gi
      requests:
        cpu: 200m
        memory: 500Mi
    ...
```

