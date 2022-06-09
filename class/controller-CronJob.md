# CronJob
  - job controller로 실행할 pod를 주기적으로 반복실행
  - data backup, send mail,cleaning tasks 등에 사용
  
## cron expression " 0 3 1 * *" => 요일상관없이, 매월 1일 3시에 반복
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
