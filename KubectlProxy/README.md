
## prepare kubectl-proxy docker image
```
feng@ubuntu:~/k8s-v2/ApiServerAmbassador$ ll
total 16
drwxr-xr-x 2 feng feng 4096 Jul 21 12:16 ./
drwxr-xr-x 6 feng feng 4096 Jul 21 12:12 ../
-rw-r--r-- 1 feng feng  371 Jul 21 12:14 Dockerfile
-rw-r--r-- 1 feng feng  322 Jul 21 12:13 kubectl-proxy.sh
-rw-r--r-- 1 feng feng    0 Jul 21 12:16 README.md
feng@ubuntu:~/k8s-v2/ApiServerAmbassador$ 
feng@ubuntu:~/k8s-v2/ApiServerAmbassador$ docker build --tag fen9li/kubectl-proxy .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM alpine
 ---> b7b28af77ffe
Step 2/4 : RUN apk update && apk add curl && curl -L -O https://dl.k8s.io/v1.8.0/kubernetes-client-linux-amd64.tar.gz && tar zvxf kubernetes-client-linux-amd64.tar.gz kubernetes/client/bin/kubectl && mv kubernetes/client/bin/kubectl / && rm -rf kubernetes && rm -f kubernetes-client-linux-amd64.tar.gz
 ---> Using cache
 ---> 94c6dd3ff12a
Step 3/4 : ADD kubectl-proxy.sh /kubectl-proxy.sh
 ---> Using cache
 ---> bc7cceeee01e
Step 4/4 : ENTRYPOINT /kubectl-proxy.sh
 ---> Using cache
 ---> 050575f42b8a
Successfully built 050575f42b8a
Successfully tagged fen9li/kubectl-proxy:latest
feng@ubuntu:~/k8s-v2/ApiServerAmbassador$ docker push fen9li/kubectl-proxy
The push refers to repository [docker.io/fen9li/kubectl-proxy]
2b6379a8b567: Layer already exists 
1e46ec28da8a: Layer already exists 
1bfeebd65323: Layer already exists 
latest: digest: sha256:dadc669e4cbf235a8fe092e4d9f7726c7750a0ef663b4db0555e08ed300abc6a size: 947
feng@ubuntu:~/k8s-v2/ApiServerAmbassador$ 

```