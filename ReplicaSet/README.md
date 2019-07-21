
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

## test app by using pods shell command
```
feng@ubuntu:~/k8s-v2$ k exec koala-ddr9f -- curl -s http://10.104.178.219:8080
You've hit koala-mwzkm
feng@ubuntu:~/k8s-v2$ k exec koala-ddr9f -- curl -s http://10.104.178.219:8080
You've hit koala-n8l49
feng@ubuntu:~/k8s-v2$
```

## accessing apiserver from inside pods
* explort pods service account by using its shell command
```
feng@ubuntu:~/k8s-v2$ k exec -it koala-ddr9f -- bash
root@koala-ddr9f:/# ls -l /var/run/secrets/kubernetes.io/serviceaccount/
total 0
lrwxrwxrwx 1 root root 13 Jul 21 01:39 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root 16 Jul 21 01:39 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 12 Jul 21 01:39 token -> ..data/token
root@koala-ddr9f:/# cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt 
-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
a3ViZUNBMB4XDTE5MDcxMTA2MzEzOFoXDTI5MDcwOTA2MzEzOFowFTETMBEGA1UE
AxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKO7
TYY0MIhUr4sZN7kmFDVn6g6PEF9+0/Llq4TzbUPlCKxSO/CgeHqxGpz5LqTv4v5Z
0FuPzUtoq4Ic3I6AT8IveFsCN1k5FIiq29kQvRG29yo0VwcEhtbN4th6QbmjBBhR
8DWZPlp/VXcCADItevDHRSe1NRI2BeqiDQ4Zl9RoN0MDVd6vy+PBn2ZK7irL/laQ
w5o5S9uYXjVMauAwP3O3qrks50LqjdmpiSRK5d5ZB8BNncPWX1Hur0XoObckpBEC
ovwk1PSWoCSjd853CHzSTjSQHgI+Oq8B7zWUPuWZ1HsB3BoRim2qZvTgtIrxt1o+
fI4WajmcikEbR0/0HCkCAwEAAaNCMEAwDgYDVR0PAQH/BAQDAgKkMB0GA1UdJQQW
MBQGCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3
DQEBCwUAA4IBAQChny1MhU/QMj7brgYPLOJUbNLmPyGRbZVAqgpSjwTXHAeVrgNE
+hkevGWHIphqryOVlGamyS9PuqhsrBEHYJlIgRP1p5w5JZ/3zFAZjFFdm9/ujinc
b9Mx4cjeBbOFucDp5QVwZsfuToR94Wj10o4T4fzPdMIAYIOoHAkEAPkLf2q3VRg4
4p+TaSx8V/gZQaKUC+KgrXFfh3nS7/AZEYGmhBecBUtC0CEBWhlteuR40HD3P0vc
M/h8r+rojp078tLT/uUr6zIWn32lloQk+QWIwnGoS37A2du0EacaAwDExd7Lx6Bo
CYE3/fPMkgQEADDJn7paVY5QshaVUJqOoaEN
-----END CERTIFICATE-----
root@koala-ddr9f:/# cat /var/run/secrets/kubernetes.io/serviceaccount/namespace 
default
root@koala-ddr9f:/# cat /var/run/secrets/kubernetes.io/serviceaccount/token     
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOGRsbWciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImRhYWMxMTI5LTcyZGUtNGJlYy1hM2FhLTYwZDBjZDRkZTEwZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.oRMqFmr3ugNmoEtX8szId2yv-HvoD45xet1v2hQQ3Luy6RbyH5JPQLyx5tjRiAon6YVZJIgVgOHcuWjGX87Kj1iGzK_hkuuibnvd-XcZ55kQE3iSJkVqY_6dkwDb1-2gS1lTA73cg_ZJjMzp4_mC8LNi_T3pk-0DhXqvd7_KyuGFxctBJoN0Hzz4g8-StmOkw8sCrYig04WddkBrwFQS-W1iqKCT7jdi4JegH2XV3ydyefMYIuKjO65iFPBE4hn8WwbLvQjG9e13EJxNmkKr3uVhAd0A9p9avHDUudvGG1IaJW1a9opCZsg4UloBQYkaI-0hQM23s48eDa3ra4lOFQ
root@koala-ddr9f:/# 
root@koala-ddr9f:/# exit
exit
feng@ubuntu:~/k8s-v2$ 
```

* access apiserver
```
feng@ubuntu:~/k8s-v2$ k exec -it koala-ddr9f -- bash
root@koala-ddr9f:/# export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
root@koala-ddr9f:/# TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
root@koala-ddr9f:/# curl -v -H "Authorization: Bearer $TOKEN" https://kubernetes
* Rebuilt URL to: https://kubernetes/
* Hostname was NOT found in DNS cache
*   Trying 10.96.0.1...
* Connected to kubernetes (10.96.0.1) port 443 (#0)
* successfully set certificate verify locations:
*   CAfile: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  CApath: /etc/ssl/certs
* SSLv3, TLS handshake, Client hello (1):
* SSLv3, TLS handshake, Server hello (2):
* SSLv3, TLS handshake, CERT (11):
* SSLv3, TLS handshake, Server key exchange (12):
* SSLv3, TLS handshake, Request CERT (13):
* SSLv3, TLS handshake, Server finished (14):
* SSLv3, TLS handshake, CERT (11):
* SSLv3, TLS handshake, Client key exchange (16):
* SSLv3, TLS change cipher, Client hello (1):
* SSLv3, TLS handshake, Finished (20):
* SSLv3, TLS change cipher, Client hello (1):
* SSLv3, TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* Server certificate:
*        subject: O=system:masters; CN=minikube
*        start date: 2019-07-20 01:39:15 GMT
*        expire date: 2020-07-20 01:39:15 GMT
*        subjectAltName: kubernetes matched
*        issuer: CN=minikubeCA
*        SSL certificate verify ok.
> GET / HTTP/1.1
> User-Agent: curl/7.38.0
> Host: kubernetes
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOGRsbWciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImRhYWMxMTI5LTcyZGUtNGJlYy1hM2FhLTYwZDBjZDRkZTEwZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.oRMqFmr3ugNmoEtX8szId2yv-HvoD45xet1v2hQQ3Luy6RbyH5JPQLyx5tjRiAon6YVZJIgVgOHcuWjGX87Kj1iGzK_hkuuibnvd-XcZ55kQE3iSJkVqY_6dkwDb1-2gS1lTA73cg_ZJjMzp4_mC8LNi_T3pk-0DhXqvd7_KyuGFxctBJoN0Hzz4g8-StmOkw8sCrYig04WddkBrwFQS-W1iqKCT7jdi4JegH2XV3ydyefMYIuKjO65iFPBE4hn8WwbLvQjG9e13EJxNmkKr3uVhAd0A9p9avHDUudvGG1IaJW1a9opCZsg4UloBQYkaI-0hQM23s48eDa3ra4lOFQ
> 
< HTTP/1.1 200 OK
< Content-Type: application/json
< Date: Sun, 21 Jul 2019 02:07:49 GMT
< Transfer-Encoding: chunked
< 
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
* Connection #0 to host kubernetes left intact
root@koala-ddr9f:/# exit
exit
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
feng@ubuntu:~/k8s-v2$ k describe ingress koala
Name:             koala
Namespace:        default
Address:          10.0.2.15
Default backend:  default-http-backend:80 (172.17.0.5:8080)
TLS:
  tls-secret terminates koala.example.com
Rules:
  Host               Path  Backends
  ----               ----  --------
  koala.example.com  
                     /   koala-nodeport:8080 (172.17.0.7:8080,172.17.0.8:8080,172.17.0.9:8080)
Annotations:
  koala.example.com/creator:                         feng li
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{"koala.example.com/creator":"feng li","nginx.ingress.kubernetes.io/rewrite-target":"/"},"name":"koala","namespace":"default"},"spec":{"rules":[{"host":"koala.example.com","http":{"paths":[{"backend":{"serviceName":"koala-nodeport","servicePort":8080},"path":"/"}]}}],"tls":[{"hosts":["koala.example.com"],"secretName":"tls-secret"}]}}

  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  UPDATE  11h (x3 over 15h)  nginx-ingress-controller  Ingress default/koala
  Normal  CREATE  11m                nginx-ingress-controller  Ingress default/koala
feng@ubuntu:~/k8s-v2$ k describe service koala-nodeport
Name:                     koala-nodeport
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"koala-nodeport","namespace":"default"},"spec":{"ports":[{"nodePor...
Selector:                 app=koala
Type:                     NodePort
IP:                       10.104.178.219
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30123/TCP
Endpoints:                172.17.0.7:8080,172.17.0.8:8080,172.17.0.9:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ k get endpoints
NAME             ENDPOINTS                                         AGE
koala-nodeport   172.17.0.7:8080,172.17.0.8:8080,172.17.0.9:8080   15h
kubernetes       192.168.99.101:8443                               5d
feng@ubuntu:~/k8s-v2$
feng@ubuntu:~/k8s-v2$ k describe endpoints koala-nodeport
Name:         koala-nodeport
Namespace:    default
Labels:       <none>
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2019-07-20T10:06:31Z
Subsets:
  Addresses:          172.17.0.7,172.17.0.8,172.17.0.9
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  8080  TCP

Events:  <none>
feng@ubuntu:~/k8s-v2$ 
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

## make koala container access apiserver via kubectl-proxy container
```
feng@ubuntu:~/k8s-v2/ReplicaSet$ k create -f service.yaml --save-config
replicaset.apps/koala created
service/koala-nodeport created
ingress.networking.k8s.io/koala created
feng@ubuntu:~/k8s-v2/ReplicaSet$ k get all
NAME              READY   STATUS    RESTARTS   AGE
pod/koala-ggxdx   2/2     Running   0          7s
pod/koala-wqhph   2/2     Running   0          7s
pod/koala-x4m68   2/2     Running   0          7s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/koala-nodeport   NodePort    10.98.71.235   <none>        8080:31399/TCP   7s
service/kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP          5d7h

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/koala   3         3         3       7s

feng@ubuntu:~/k8s-v2/ReplicaSet$ 
feng@ubuntu:~/k8s-v2/ReplicaSet$ k exec -it koala-ggxdx --container koala -- bash
root@koala-ggxdx:/# curl http://localhost:8001
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
}root@koala-ggxdx:/# exit
exit
feng@ubuntu:~/k8s-v2/ReplicaSet$ 
```