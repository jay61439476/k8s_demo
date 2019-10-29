# k8s_demo
k8s_demo with minikube

## MAC操作记录

* 启动
```bash
minikube start --image-mirror-country=cn
😄  minikube v1.4.0 on Darwin 10.15
⚠️  None of the known repositories in your location are accessible. Using registry.cn-hangzhou.aliyuncs.com/google_containers as fallback.
✅  Using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.16.0 on Docker 18.09.9 ...
🚜  Pulling images ...
🚀  Launching Kubernetes ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```

* 验证
```
minikube status

host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

* Demo
```
eval $(minikube docker-env)

# cd /Users/jay/code/dev/k8s/k8s_demo

➜  docker build -t k8s-demo:0.1 .

Sending build context to Docker daemon  5.632kB
Step 1/2 : FROM nginx
latest: Pulling from library/nginx
8d691f585fa8: Pull complete
5b07f4e08ad0: Pull complete
abc291867bca: Pull complete
Digest: sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
Status: Downloaded newer image for nginx:latest
 ---> 540a289bab6c
Step 2/2 : COPY html/* /usr/share/nginx/html
 ---> 279d420cbc27
Successfully built 279d420cbc27
Successfully tagged k8s-demo:0.1

➜  k8s_demo docker images|grep demo
k8s-demo                                                                          0.1                 279d420cbc27        5 seconds ago       126MB

➜  k8s_demo k get pods
No resources found.

➜  k8s_demo kubectl create -f pod.yml
pod/k8s-demo created

➜  k8s_demo k get pods
NAME       READY   STATUS    RESTARTS   AGE
k8s-demo   1/1     Running   0          4s

➜  k8s_demo kubectl describe pods | grep Labels
Labels:       app=k8s-demo

➜  k8s_demo kubectl create -f svc.yml
service/k8s-demo-svc created

➜  k8s_demo k get svc
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
k8s-demo-svc   NodePort    10.108.119.235   <none>        80:30050/TCP   8s
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        6m3s

➜  k8s_demo minikube service k8s-demo-svc --url
http://192.168.99.100:30050

➜  k8s_demo curl http://192.168.99.100:30050
<h1>Hello Docker!</h1>

➜  k create -f deployment.yml
deployment.apps/k8s-demo-deployment created

➜  k8s_demo k get rs
NAME                             DESIRED   CURRENT   READY   AGE
k8s-demo-deployment-7c4cf5fbbf   3         3         3       16s

```

* 横向扩展、滚动更新、版本回滚
```
# update app
echo '<h1>Hello Kubernetes!</h1>' > html/index.html
docker build -t k8s-demo:0.2 .

# 更新 deployment.yml  
k apply -f deployment.yml --record=true # --record=true 让 Kubernetes 把这行命令记到发布历史中备查
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/k8s-demo-deployment configured

# 查看日志
rollout status deployment k8s-demo-deployment

# 查看历史日志
➜  k8s_demo git:(master) ✗ k rollout history deployment k8s-demo-deployment
deployment.apps/k8s-demo-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl apply --filename=deployment.yml --record=true

# 回滚
k rollout undo deployment k8s-demo-deployment --to-revision=1
deployment.apps/k8s-demo-deployment rolled back
➜  k8s_demo git:(master) ✗ k  get pods
NAME                                   READY   STATUS        RESTARTS   AGE
k8s-demo-deployment-545967bfcb-2scqh   1/1     Running       0          112s
k8s-demo-deployment-545967bfcb-ppp6v   0/1     Terminating   0          103s
k8s-demo-deployment-545967bfcb-zckhk   1/1     Running       0          112s
k8s-demo-deployment-7c4cf5fbbf-h4hc8   1/1     Running       0          3s
k8s-demo-deployment-7c4cf5fbbf-qdn27   1/1     Running       0          3s

➜  k8s_demo git:(master) ✗ k  get pods
NAME                                   READY   STATUS        RESTARTS   AGE
k8s-demo-deployment-545967bfcb-2scqh   0/1     Terminating   0          119s
k8s-demo-deployment-7c4cf5fbbf-8zgkb   1/1     Running       0          3s
k8s-demo-deployment-7c4cf5fbbf-h4hc8   1/1     Running       0          10s
k8s-demo-deployment-7c4cf5fbbf-qdn27   1/1     Running       0          10s

➜  k8s_demo git:(master) ✗ k  get pods
NAME                                   READY   STATUS    RESTARTS   AGE
k8s-demo-deployment-7c4cf5fbbf-8zgkb   1/1     Running   0          72s
k8s-demo-deployment-7c4cf5fbbf-h4hc8   1/1     Running   0          79s
k8s-demo-deployment-7c4cf5fbbf-qdn27   1/1     Running   0          79s

```


## 参考
* [Docker 和 Kubernetes 从听过到略懂：给程序员的旋风教程](https://juejin.im/post/5b62d0356fb9a04fb87767f5)
