
## prepare kubectl-proxy docker image
```
feng@ubuntu:~/k8s-v2/KubectlProxy$ ll
total 24
drwxr-xr-x 2 feng feng 4096 Jul 21 17:11 ./
drwxr-xr-x 6 feng feng 4096 Jul 21 12:24 ../
-rw-r--r-- 1 feng feng  387 Jul 21 18:07 Dockerfile
-rw-r--r-- 1 feng feng  321 Jul 21 18:05 kubectl-proxy.sh
-rw-r--r-- 1 feng feng  199 Jul 21 18:10 pod.yaml
-rw-r--r-- 1 feng feng 1538 Jul 21 12:28 README.md
feng@ubuntu:~/k8s-v2/KubectlProxy$  

feng@ubuntu:~/k8s-v2/KubectlProxy$ docker build -t fen9li/kubectl-proxy:v1.0.0 .
Sending build context to Docker daemon  6.656kB
Step 1/4 : FROM alpine
 ---> b7b28af77ffe
Step 2/4 : RUN apk update && apk add curl && curl -L -O https://dl.k8s.io/v1.8.0/kubernetes-client-linux-amd64.tar.gz && tar zvxf kubernetes-client-linux-amd64.tar.gz kubernetes/client/bin/kubectl && mv kubernetes/client/bin/kubectl /usr/bin/ && rm -rf kubernetes && rm -f kubernetes-client-linux-amd64.tar.gz
 ---> Using cache
 ---> 9824e11f52a1
Step 3/4 : ADD kubectl-proxy.sh /kubectl-proxy.sh
 ---> d92a88897a37
Step 4/4 : ENTRYPOINT ["sh","kubectl-proxy.sh"]
 ---> Running in 3a91f28d5f49
Removing intermediate container 3a91f28d5f49
 ---> b501a565eb6e
Successfully built b501a565eb6e
Successfully tagged fen9li/kubectl-proxy:v1.0.0
feng@ubuntu:~/k8s-v2/KubectlProxy$

feng@ubuntu:~/k8s-v2/KubectlProxy$ docker push fen9li/kubectl-proxy:v1.0.0
The push refers to repository [docker.io/fen9li/kubectl-proxy]
b3759b25a654: Pushed 
3dba883acef1: Layer already exists 
1bfeebd65323: Layer already exists 
v1.0.0: digest: sha256:08b1b9c716ac644de2c276f07121235b61b2a585492ade1cf5618214becbab30 size: 947
feng@ubuntu:~/k8s-v2/KubectlProxy$  

```

## test kubectl-proxy

```
feng@ubuntu:~/k8s-v2/KubectlProxy$ k create -f pod.yaml 
pod/curl created
feng@ubuntu:~/k8s-v2/KubectlProxy$

feng@ubuntu:~/k8s-v2/KubectlProxy$ k get all
NAME       READY   STATUS    RESTARTS   AGE
pod/curl   2/2     Running   0          13s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d7h

feng@ubuntu:~/k8s-v2/KubectlProxy$ k exec -it curl --container curl -- bash
root@curl:/# curl http://localhost:8001
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/apps/v1beta1",
    "/apis/apps/v1beta2",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authentication.k8s.io/v1beta1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/authorization.k8s.io/v1beta1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1beta1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/coordination.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1beta1",
    "/apis/extensions",
    "/apis/extensions/v1beta1",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/networking.k8s.io/v1beta1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1beta1",
    "/apis/policy",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/rbac.authorization.k8s.io/v1beta1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/scheduling.k8s.io/v1beta1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/ca-registration",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/version"
  ]
}root@curl:/# exit
exit
feng@ubuntu:~/k8s-v2/KubectlProxy$
```