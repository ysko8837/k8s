  # pod 관련 총정리 (5-7 강좌)
  
  - create a static pod on node01 called mydb with image redis
    create this pod on node01 and make sure that is recreated/restarted automatically in case of a failure.
    - use /ect/kubernetes/manifests as the static pod path for example
    - kubelet configured for static pods
    - pod mydb-node01 is Up and running
  - 다음과 같은 조건에 맞는 Pod를 생성하시오
    - Pod name : myweb, image: nginx:1.14
    - CPU 200m, Memory 500Mi를 요구하고, CPU 1 core, Memory 1Gi 제한 받는다
    - application동작에 필요한 환경변수 DB=mydb를 포함한다. 
    - namespace product에서 동작되어야 한다. 

