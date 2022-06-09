# job controller
  - 쿠버네티스는 pod를 running상태로 유지하려고 함
  - batch 처리는 작업이 완료되면 종료되어야 하고, 실패하면 재시도 해야함
  - job controller는 이름 지원하여 성공적인 완료를 보장
  - `spec.restartPolicy`에 OnFailure(container를 재시작), Never(리스타트를 안하므로 pod를 재시작 시도함) 가 가능
  - Onfailure는 `spec.backoffLimit`을 써줌(재시작 횟수, default는 6번)
  - 재시작 횟수만큼 실패하면 pod를 삭제하고 작업이 중단됨
  - https://github.com/237summit/Getting-Start-Kubernetes/blob/main/6/job-exam.yaml 참고


## Never 상태
```
vi job-example.yaml
kubectl create -f job-example.yaml 
kubectl delete pod centos-job-c86mk # complete이전에는 pod가 재생성됨(50s)
kubectl get jobs.batch
kubectl describe job centos-job
kubectl delete jobs.batch centos-job
```

## OnFailure 상태
  - bash명령어를 일부러 틀리게 처리, backoffLimit: 3
```
vi job-example.yaml 
kubectl delete -f job-example.yaml
kubectl create -f job-example.yaml 
watch kubectl get pods -o wide 
# 3회 실패 후, pod 삭제됨
NAME               READY   STATUS              RESTARTS     AGE   IP             NODE    NOMINATED NODE   READINESS GATES
centos-job-n7ctm   0/1     RunContainerError   3 (1s ago)   43s   10.244.2.108   node2   <none>           <none>
```

## 쿠버네티스 동작 원리 
```
kubectl run tespod --image=centos:7 --command sleep 5 
watch kubectl get pods -o wide
NAME      READY   STATUS      RESTARTS      AGE   IP             NODE    NOMINATED NODE   READINESS GATES
testpod   0/1     Completed   2 (22s ago)   34s   10.244.2.103   node2   <none>           <none>
# 컨테이너 내부에서 5초간 슬립되고 종료됨
# 쿠버네티스는 다시 pod를 실행함
```

## completions, parallelism, activeDeadlineSeconds
  - replica가 없고, spec.completions: 3 로 3번 실행 선언 가능(completed되면 다음 running으로 순차적으로 실행됨)
  - spec.parallelism: 2 동시에 2개씩 pod가 실행됨
  - spec.activeDeadlineSeconds: 5 5초가 되도록 완료되지 않으면, 강제로 종료로 처리됨

```
# spec.completions: 5 로 수정 후 job 실행결과
NAME               READY   STATUS      RESTARTS   AGE    IP             NODE    NOMINATED NODE   READINESS GATES
centos-job-4fgbb   0/1     Completed   0          63s    10.244.2.110   node2   <none>           <none>
centos-job-5nxjv   0/1     Completed   0          92s    10.244.1.110   node1   <none>           <none>
centos-job-7mv24   0/1     Completed   0          73s    10.244.1.111   node1   <none>           <none>
centos-job-rbf7s   0/1     Completed   0          100s   10.244.1.109   node1   <none>           <none>
centos-job-zs6f4   0/1     Completed   0          82s    10.244.2.109   node2   <none>           <none>
```

```
# spec.completions: 5, spec.parallelism: 2 로 수정 후 job 실행결과(2개, 2개, 1개 실행됨)
NAME               READY   STATUS      RESTARTS   AGE   IP             NODE    NOMINATED NODE   READINESS GATES
centos-job-2wrp4   0/1     Completed   0          84s   10.244.2.111   node2   <none>           <none>
centos-job-9lcsq   0/1     Completed   0          66s   10.244.2.113   node2   <none>           <none>
centos-job-9pbst   0/1     Completed   0          75s   10.244.1.113   node1   <none>           <none>
centos-job-gcvhj   0/1     Completed   0          84s   10.244.1.112   node1   <none>           <none>
centos-job-pkcxs   0/1     Completed   0          75s   10.244.2.112   node2   <none>           <none>
```

```
#spec.activeDeadlineSeconds: 5 5초후 자동 종료
```
