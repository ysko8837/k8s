# infra container(pause)
  - 하나의 container를 만들어도 숨겨진 pause container가 함께 생성되고 함께 삭제됨
  - pause는 infra환경만 만들어주고 작업을 수행하지는 않음
  - pod의 환경을 만들어주는 컨테이너 (ip, hostname 등 관리)


```
# 실행중인 노드로 들어가서 pause 확인
ssh ysko@node2 -p 2222
docker ps
```
