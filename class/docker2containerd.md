# kubernetes Docker 에서 containerd로 마이그레이션
 - 참조 : https://ikcoo.tistory.com/190
 - containerd 명령어 참조 : https://trylhc.tistory.com/entry/ctr-containerd-CLI-tools
 - nerdctl : https://github.com/containerd/nerdctl (docker와 유사한 명령어 가능)

```
# ctr 명령어
ctr namespaces list
ctr -n moby containers list
ctr -n k8s.io containers list
```
