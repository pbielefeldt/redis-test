## start redis ##
```bash
kubectl create -f redis.yml
kubectl get all
kubectl exec -it pod/redis-XYZ -- redis-cli --pass p4ssw0rd
```

## check if persistency is enabled ##
```bash
CONFIG GET save
CONFIG GET appendonly
```

## fill something into database ## 
```bash
set "hi" 42420
get hi
# repeat 9+ times with different keys if "save ... 10" is set
exit
```

## verify append-file is written ##
```bash
kubectl exec -it pod/redis-XYZ -- /usr/bin/env sh
cat appendonly.aof
exit
```

## delete pod (but not pvc) ##
```bash
kubectl delete pod/redis-XYZ
kubectl get pods
```

## check if entry was persisted ##
```bash
kubectl exec -it pod/redis-ABC -- redis-cli --pass p4ssw0rd
get hi
exit
```

## clean up ##
```bash
kubectl delete -f redis.yml
```

<!--
## -> ##
```bash
$ kubectl create -f redis.yml
configmap/example-redis-config created
persistentvolumeclaim/redis-data-pvc created
deployment.apps/redis created
$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/redis-6c79db55f6-nhwgr   1/1     Running   0          10s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1/1     1            1           10s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-6c79db55f6   1         1         1       10s
$
$ kubectl exec -it pod/redis-6c79db55f6-nhwgr -- redis-cli
127.0.0.1:6379> set "hi" 42420
OK
127.0.0.1:6379> set "hello_world" "hello world!"
OK
127.0.0.1:6379> get hi
"42420"
127.0.0.1:6379> get hello_world
"hello world!"
127.0.0.1:6379> exit
$ kubectl exec -it pod/redis-6c79db55f6-nhwgr -- /usr/bin/env sh
# cat appendonly.aof
*2
$6
SELECT
$1
0
*3
$3
set
$2
hi
$5
42420
*3
$3
set
$11
hello_world
$12
hello world!
# exit
$
$ kubectl delete pod/redis-6c79db55f6-nhwgr
pod "redis-6c79db55f6-nhwgr" deleted
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
redis-6c79db55f6-lqg78   1/1     Running   0          24s
$
$ kubectl exec -it pod/redis-6c79db55f6-lqg78 -- redis-cli
127.0.0.1:6379> get hi
"42420"
127.0.0.1:6379> get hello_world
"hello world!"
127.0.0.1:6379> exit
$
$ kubectl delete -f redis.yml
configmap "example-redis-config" deleted
persistentvolumeclaim "redis-data-pvc" deleted
deployment.apps "redis" deleted
```
-->