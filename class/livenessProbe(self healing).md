#livenessProbe (self healing)
  - 3회 시도 후, container를 새로 받고 container를 재시작(pod재시작이 아님, 따라서 ip유지)
  - 3개(httpGet, tcpSocket, exec)의 Probe 지원
 
 
