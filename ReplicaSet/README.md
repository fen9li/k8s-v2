
## clean up local docker images
```
feng@ubuntu:~/k8s-v2/ReplicaSet$ docker images | grep kubia
fen9li/kubia           v4.0.0              d6328a5c3c3c        8 hours ago         660MB
fen9li/kubia           v3.0.0              fac4ec6ae2f9        24 hours ago        660MB
fen9li/kubia           v2.0.0              771370886e94        27 hours ago        660MB
fen9li/kubia           v1.0.0              670e99ccfc86        27 hours ago        660MB
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

feng@ubuntu:~/k8s-v2/ReplicaSet$ docker images | grep kubia | awk '{print $3}'
d6328a5c3c3c
fac4ec6ae2f9
771370886e94
670e99ccfc86
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

docker rmi `docker images | grep kubia | awk '{print $3}'`

feng@ubuntu:~/k8s-v2/ReplicaSet$ docker images | grep kubia
feng@ubuntu:~/k8s-v2/ReplicaSet$ 
```

## prepare docker image
```
feng@ubuntu:~/k8s-v2/ReplicaSet$ ll
total 20
drwxr-xr-x 2 feng feng 4096 Jul 20 19:52 ./
drwxr-xr-x 5 feng feng 4096 Jul 20 19:27 ../
-rw-r--r-- 1 feng feng  356 Jul 20 19:52 app-v1.js
-rw-r--r-- 1 feng feng   64 Jul 20 19:52 Dockerfile
-rw-r--r-- 1 feng feng  813 Jul 20 19:51 README.md
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

docker build --tag fen9li/koala:v1 .
docker push fen9li/koala:v1

feng@ubuntu:~/k8s-v2/ReplicaSet$
```

## create and expose service (with nodeport)
```
feng@ubuntu:~/k8s-v2/ReplicaSet$ k create -f service.yaml --save-config
replicaset.apps/koala created
service/koala-nodeport created
feng@ubuntu:~/k8s-v2/ReplicaSet$

feng@ubuntu:~/k8s-v2/ReplicaSet$ k get all
NAME              READY   STATUS    RESTARTS   AGE
pod/koala-ddr9f   1/1     Running   0          8m48s
pod/koala-mwzkm   1/1     Running   0          8m48s
pod/koala-n8l49   1/1     Running   0          8m48s

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/koala-nodeport   NodePort    10.104.178.219   <none>        8080:30123/TCP   8m48s
service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          4d9h

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/koala   3         3         3       8m48s

feng@ubuntu:~/k8s-v2/ReplicaSet$ 

feng@ubuntu:~/k8s-v2/ReplicaSet$ minikube service koala-nodeport --url
http://192.168.99.101:30123
feng@ubuntu:~/k8s-v2/ReplicaSet$  

feng@ubuntu:~/k8s-v2/ReplicaSet$ curl http://192.168.99.101:30123
You've hit koala-ddr9f
feng@ubuntu:~/k8s-v2/ReplicaSet$ curl http://192.168.99.101:30123
You've hit koala-n8l49
feng@ubuntu:~/k8s-v2/ReplicaSet$ curl http://192.168.99.101:30123
You've hit koala-mwzkm
feng@ubuntu:~/k8s-v2/ReplicaSet$ 
```

![dashboard-01](images/dashboard-01.png)

## test app by using port froward
* configure port forward on pods
```
feng@ubuntu:~/k8s-v2/ReplicaSet$ k port-forward koala-ddr9f 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
... ...
```

* test app in a new window or in a browser
```
feng@ubuntu:~/k8s-v2$ curl http://localhost:8080
You've hit koala-ddr9f
feng@ubuntu:~/k8s-v2$ 
```

## expose service by adding ingress 

```
feng@ubuntu:~/k8s-v2$ minikube addons enable ingress
✅  ingress was successfully enabled
feng@ubuntu:~/k8s-v2$ minikube addons list | grep ingress
- ingress: enabled
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2/ReplicaSet$ k apply -f service.yaml 
replicaset.apps/koala unchanged
service/koala-nodeport unchanged
ingress.networking.k8s.io/koala created
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

feng@ubuntu:~/k8s-v2/ReplicaSet$ minikube ip
192.168.99.101
feng@ubuntu:~/k8s-v2/ReplicaSet$ sudo -i
[sudo] password for feng: 
root@ubuntu:~# vim /etc/hosts
root@ubuntu:~# tail -n 1 /etc/hosts
192.168.99.101 koala.example.com koala
root@ubuntu:~# exit
logout
feng@ubuntu:~/k8s-v2/ReplicaSet$ curl http://koala.example.com
You've hit koala-ddr9f
feng@ubuntu:~/k8s-v2/ReplicaSet$ curl http://koala.example.com
You've hit koala-mwzkm
feng@ubuntu:~/k8s-v2/ReplicaSet$ curl http://koala.example.com
You've hit koala-n8l49
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

```
![ingress_explain](images/ingress_explain.png)

![ingress_in_dashboard](images/ingress_in_dashboard.png)

## setup tls for koala.example.com in ingress

* create tls certificate
```
feng@ubuntu:~/k8s-v2/ReplicaSet$ mkdir ssl
feng@ubuntu:~/k8s-v2/ReplicaSet$ cd ssl
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ openssl genrsa -out tls.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.......+++++
...................................................+++++
e is 65537 (0x010001)
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ openssl req -new -x509 -key tls.key -out tls.cert -days 3650 -subj /CN=koala.example.com
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ ls -l
total 8
-rw-r--r-- 1 feng feng 1135 Jul 20 21:36 tls.cert
-rw------- 1 feng feng 1679 Jul 20 21:35 tls.key
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ 
```

* create tls-secret
```
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key
secret/tls-secret created
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ k get secret
NAME                  TYPE                                  DATA   AGE
default-token-8dlmg   kubernetes.io/service-account-token   3      4d10h
tls-secret            kubernetes.io/tls                     2      10s
feng@ubuntu:~/k8s-v2/ReplicaSet/ssl$ 
```

* update service.yaml and apply

```
feng@ubuntu:~/k8s-v2/ReplicaSet$ sed -n '32,50p' service.yaml 
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: koala
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - koala.example.com
    secretName: tls-secret
  rules:
  - host: koala.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: koala-nodeport
          servicePort: 8080
feng@ubuntu:~/k8s-v2/ReplicaSet$ 
feng@ubuntu:~/k8s-v2/ReplicaSet$ k apply -f service.yaml 
replicaset.apps/koala unchanged
service/koala-nodeport unchanged
ingress.networking.k8s.io/koala configured
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

feng@ubuntu:~/k8s-v2/ReplicaSet$ k describe ingress koala
Name:             koala
Namespace:        default
Address:          10.0.2.15
Default backend:  default-http-backend:80 (172.17.0.4:8080)
TLS:
  tls-secret terminates koala.example.com
Rules:
  Host               Path  Backends
  ----               ----  --------
  koala.example.com  
                     /   koala-nodeport:8080 (172.17.0.5:8080,172.17.0.8:8080,172.17.0.9:8080)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{"nginx.ingress.kubernetes.io/rewrite-target":"/"},"name":"koala","namespace":"default"},"spec":{"rules":[{"host":"koala.example.com","http":{"paths":[{"backend":{"serviceName":"koala-nodeport","servicePort":8080},"path":"/"}]}}],"tls":[{"hosts":["koala.example.com"],"secretName":"tls-secret"}]}}

  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type    Reason  Age                  From                      Message
  ----    ------  ----                 ----                      -------
  Normal  UPDATE  4m35s (x2 over 84m)  nginx-ingress-controller  Ingress default/koala
feng@ubuntu:~/k8s-v2/ReplicaSet$ 

feng@ubuntu:~/k8s-v2/ReplicaSet$ curl -k -v https://koala.example.com
* Expire in 0 ms for 6 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 1 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
* Expire in 0 ms for 1 (transfer 0x55791ae754f0)
*   Trying 192.168.99.101...
* TCP_NODELAY set
* Expire in 200 ms for 4 (transfer 0x55791ae754f0)
* Connected to koala.example.com (192.168.99.101) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: none
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=koala.example.com
*  start date: Jul 20 11:36:24 2019 GMT
*  expire date: Jul 17 11:36:24 2029 GMT
*  issuer: CN=koala.example.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55791ae754f0)
> GET / HTTP/2
> Host: koala.example.com
> User-Agent: curl/7.64.0
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 200 
< server: nginx/1.15.9
< date: Sat, 20 Jul 2019 11:44:38 GMT
< 
You've hit koala-ddr9f
* Connection #0 to host koala.example.com left intact
feng@ubuntu:~/k8s-v2/ReplicaSet$ 
```

![test_ingress_with_tls](images/test_ingress_with_tls.png)
