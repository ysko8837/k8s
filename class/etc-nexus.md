# local(virtualbox) nexus
  -- 17080(private-image-registry docker:group), 18001(module-www-image docker:hosted) 은 repository에서 저장한 값

```

[성공 사례]

  127.0.0.1 => nexus로 변경 가능

#login  
sudo docker login \
  --username image-reader \
  --password imgcon11 \
  https://127.0.0.1:17080

#실제 파일은 다운로드 안됨(0 byte)
curl -X 'GET' -G -k \
  -u "file-uploader:filecon11" \
  -d repository="file-storage" \
  -d maven.groupId="apm-scouter" \
  -d maven.artifactId="agent-java-6-7" \
  -d maven.baseVersion="2.12.0.1" \
  -d maven.extension="jar" \
  -d sort=version \
  -d direction=desc \
  -o "./scouter.agent.jar" \
  -L "http://127.0.0.1:12081/service/rest/v1/search/assets/download"  

# docker build(nexus web에서 module-www-production에 https 체크, 18001 저장 후 성공)
# base image등 저장 성공
sudo docker build \
  --no-cache \
  --rm \
  --force-rm \
  --file ./Dockerfile-www-production \
  --tag "127.0.0.1:18001/module-www-production:12345" \
  --build-arg JAR_FILE_PATH=./ \
  --build-arg JAR_FILE_NAME=module-www-0.0.1-SNAPSHOT.jar \
  --build-arg EXPOSE_PORT=8080 \
  .

  sudo docker login \
  --username image-uploader \
  --password imgupcon11 \
  https://127.0.0.1:18001

# push
sudo docker push 127.0.0.1:18001/module-www-production:12345

# pull
docker pull nexus:17080/module-www-production:12345
  
```
