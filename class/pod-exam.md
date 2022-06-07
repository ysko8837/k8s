create a static pod on node01 called mydb with image redis
create this pod on node01 and make sure that is recreated/restarted automatically in case of a failure.
 - use /ect/kubernetes/manifests as the static pod path for example
 - kubelet configured for static pods
 - pod mydb-node01 is Up and running

