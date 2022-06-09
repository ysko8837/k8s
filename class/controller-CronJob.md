# CronJob
  - job controller로 실행할 pod를 주기적으로 반복실행
  - data backup, send mail,cleaning tasks 등에 사용
  - https://github.com/237summit/Getting-Start-Kubernetes/blob/main/6/cronjob-exam.yaml
  
## cron expression
  - minutes (from 0 to 59)
  - hours (from 0 to 23)
  - day of th month ( from 1 to 31)
  - month ( from 1 to 12)
  - day of the week ( from 0 to 6) # 0은 일요일, 1-5 는 월~금요일

```
0 9 1 * *     #매월 1일 아침 9시에 job 실행
0 3 * * 0     #매주 일요일 3시에 job 실행
0 3 * * 1-5   #주중 새벽 3시에 job 실행
0 3 * * 0,6   #주말(토요일,일요일) 3시에 job 실행
* * * * *     #매분마다 job 실행
*/5 * * *     #5분마다 job 실행
0 */2 * * *   #2시간마다 매시 정각에 job 실행
0 */2 1,15 * * #매월 1일과 15일에 매 2시간마다 정각에 job 실행
```

## yaml 형태
  - kind: CronJob
  - 기존 job의 spec이하 전체를 spec.jobTemplate: 아래에 둬야 함
  - spec.schedule: "0 3 1 * *" 이 추가되어야 함
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-exam
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo Hello; sleep 10; echo Bye
          restartPolicy: Never
```

## concurrencyPolicy(Allow / Forbid)
  - Allow(default) : cron schedule마다 계속 실행됨, 동시에 여러개가 실행될 수 있음
  - Forbid : 한번에 하나씩만 실행됨, 이전 작업이 완료되어야 다음 스케줄이 동작
  - startingDeadlineSeconds : 제한시간안에 완료되지 않으면 취소시킴
  - successfulJobsHistoryLimit: 반복 실행된 히스토리를 남길 수 지정 가능(default 3)

```
# schedule: "* * * * *" , concurrencyPolicy: Forbid 실행결과
vi cronjob-exam.yaml
kubectl create -f cronjob-exam.yaml
watch .... 

NAME                          READY   STATUS      RESTARTS   AGE   IP             NODE    NOMINATED NODE   READINESS GATES
cronjob-exam-27579407-gkdcj   0/1     Completed   0          69s   10.244.1.116   node1   <none>           <none>
cronjob-exam-27579408-rdv2k   1/1     Running     0          9s    10.244.2.116   node2   <none>           <none>

NAME                          READY   STATUS      RESTARTS   AGE   IP             NODE    NOMINATED NODE   READINESS GATES
cronjob-exam-27579407-gkdcj   0/1     Completed   0          84s   10.244.1.116   node1   <none>           <none>
cronjob-exam-27579408-rdv2k   0/1     Completed   0          24s   10.244.2.116   node2   <none>           <none>

NAME                          READY   STATUS      RESTARTS   AGE    IP             NODE    NOMINATED NODE   READINESS GATES
cronjob-exam-27579407-gkdcj   0/1     Completed   0          2m6s   10.244.1.116   node1   <none>           <none>
cronjob-exam-27579408-rdv2k   0/1     Completed   0          66s    10.244.2.116   node2   <none>           <none>
cronjob-exam-27579409-67dnw   1/1     Running     0          6s     10.244.1.117   node1   <none>           <none>
```

```
# 현 cronjob을 yaml형태로 확인(successfulJobsHistoryLimit 위치 확인가능)
kubectl get cronjobs.batch -o yaml 
```
