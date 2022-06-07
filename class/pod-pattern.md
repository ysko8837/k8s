#pod 실행 패턴
  - pod를 구성하고 실행하는 패턴
  - multi-container Pod(sidecar, adapter, ambassador)
  - sidecar : 내부에서 생성된 정보를 토대로 동작(웹서버가 로그를 생성하고 그 로그를 가공하여 분석)
  - adapter : 외부 정보를 가져오는 형태(adapter가 연동하여 가져옴)
  - ambassador : 생성된 정보를 받아서 분기하여 보내주는 형태(loadbalancer하여 보내줌) 

https://seongjin.me/content/images/2022/02/pod-design-patterns.png
