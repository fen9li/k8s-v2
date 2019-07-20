
## setup
```
feng@ubuntu:~$ mkdir k8s-v2
feng@ubuntu:~$ cd k8s-v2
feng@ubuntu:~/k8s-v2$ curl -u 'fen9li' https://api.github.com/user/repos -d '{"name":"k8s-v2"}' | jq '.ssh_url'
Enter host password for user 'fen9li':
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5140  100  5123  100    17   3277     10  0:00:01  0:00:01 --:--:--  3286
"git@github.com:fen9li/k8s-v2.git"
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ git init
Initialized empty Git repository in /home/feng/k8s-v2/.git/
feng@ubuntu:~/k8s-v2$ git remote add origin git@github.com:fen9li/k8s-v2.git
feng@ubuntu:~/k8s-v2$ git remote -v
origin  git@github.com:fen9li/k8s-v2.git (fetch)
origin  git@github.com:fen9li/k8s-v2.git (push)
feng@ubuntu:~/k8s-v2$ git config --global --list
user.name=Feng Li
user.email=lifcn@yahoo.com
feng@ubuntu:~/k8s-v2$

feng@ubuntu:~/k8s-v2$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        README.md

nothing added to commit but untracked files present (use "git add" to track)
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ git add .
feng@ubuntu:~/k8s-v2$ git commit -am "initial commit"
[master (root-commit) afaaf98] initial commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ git push --set-upstream origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 210 bytes | 210.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:fen9li/k8s-v2.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
feng@ubuntu:~/k8s-v2$ 
```

## dashboard
* start minikube
```
feng@ubuntu:~$ minikube start
😄  minikube v1.2.0 on linux (amd64)
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Restarting existing virtualbox VM for "minikube" ...
⌛  Waiting for SSH access ...
🐳  Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
🔄  Relaunching Kubernetes v1.15.0 using kubeadm ... 
⌛  Verifying: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
feng@ubuntu:~$ 
```

* check if dashboard addons enable
```
feng@ubuntu:~$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.101
feng@ubuntu:~$ 

feng@ubuntu:~$ minikube addons list | grep dashboard
- dashboard: enabled
feng@ubuntu:~$ 

```

* start dashboard
```
feng@ubuntu:~$ minikube dashboard
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:36587/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
[7128:7148:0720/192655.452001:ERROR:browser_process_sub_thread.cc(221)] Waited 3 ms for network service
Opening in existing browser session.

```

![k8s-dashboard.png](images/k8s-dashboard.png)

## to list all suported API versions and resources
```
feng@ubuntu:~/k8s-v2$ k api-versions
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
apps/v1beta1
apps/v1beta2
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ k api-resources
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
leases                                         coordination.k8s.io            true         Lease
events                            ev           events.k8s.io                  true         Event
daemonsets                        ds           extensions                     true         DaemonSet
deployments                       deploy       extensions                     true         Deployment
ingresses                         ing          extensions                     true         Ingress
networkpolicies                   netpol       extensions                     true         NetworkPolicy
podsecuritypolicies               psp          extensions                     false        PodSecurityPolicy
replicasets                       rs           extensions                     true         ReplicaSet
ingresses                         ing          networking.k8s.io              true         Ingress
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
feng@ubuntu:~/k8s-v2$ 
```

## start apiserver proxy to explore k8s objects
* The apiserver paths can be explored about your cluster
```
feng@ubuntu:~/k8s-v2$ curl http://localhost:8001
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
}
feng@ubuntu:~/k8s-v2$ 
```

* get objects' name defined in path apis/extensions/v1beta1
```
feng@ubuntu:~/k8s-v2$ curl http://localhost:8001/apis/extensions/v1beta1 | jq '.resources[].name'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4273    0  4273    0     0  1390k      0 --:--:-- --:--:-- --:--:-- 2086k
"daemonsets"
"daemonsets/status"
"deployments"
"deployments/rollback"
"deployments/scale"
"deployments/status"
"ingresses"
"ingresses/status"
"networkpolicies"
"podsecuritypolicies"
"replicasets"
"replicasets/scale"
"replicasets/status"
"replicationcontrollers"
"replicationcontrollers/scale"
feng@ubuntu:~/k8s-v2$ 
```

* get deeper level object details by using 'jq'
```
feng@ubuntu:~/k8s-v2$ curl http://localhost:8001/apis/extensions/v1beta1/deployments | jq '.items[].metadata.selfLink'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 23491    0 23491    0     0  4588k      0 --:--:-- --:--:-- --:--:-- 4588k
"/apis/extensions/v1beta1/namespaces/kube-system/deployments/coredns"
"/apis/extensions/v1beta1/namespaces/kube-system/deployments/default-http-backend"
"/apis/extensions/v1beta1/namespaces/kube-system/deployments/kubernetes-dashboard"
"/apis/extensions/v1beta1/namespaces/kube-system/deployments/nginx-ingress-controller"
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ curl http://localhost:8001/apis/extensions/v1beta1/deployments | jq '.items[0]'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 23491    0 23491    0     0  1147k      0 --:--:-- --:--:-- --:--:-- 1147k
{
  "metadata": {
    "name": "coredns",
    "namespace": "kube-system",
    "selfLink": "/apis/extensions/v1beta1/namespaces/kube-system/deployments/coredns",
    "uid": "d541f8a1-c9fb-4f5b-b3ea-ddf01b34538d",
    "resourceVersion": "238542",
    "generation": 10,
    "creationTimestamp": "2019-07-16T00:59:28Z",
    "labels": {
      "k8s-app": "kube-dns"
    },
    "annotations": {
      "deployment.kubernetes.io/revision": "1"
    }
  },
  "spec": {
    "replicas": 2,
    "selector": {
      "matchLabels": {
        "k8s-app": "kube-dns"
      }
    },
    "template": {
      "metadata": {
        "creationTimestamp": null,
        "labels": {
          "k8s-app": "kube-dns"
        }
      },
      "spec": {
        "volumes": [
          {
            "name": "config-volume",
            "configMap": {
              "name": "coredns",
              "items": [
                {
                  "key": "Corefile",
                  "path": "Corefile"
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "coredns",
            "image": "k8s.gcr.io/coredns:1.3.1",
            "args": [
              "-conf",
              "/etc/coredns/Corefile"
            ],
            "ports": [
              {
                "name": "dns",
                "containerPort": 53,
                "protocol": "UDP"
              },
              {
                "name": "dns-tcp",
                "containerPort": 53,
                "protocol": "TCP"
              },
              {
                "name": "metrics",
                "containerPort": 9153,
                "protocol": "TCP"
              }
            ],
            "resources": {
              "limits": {
                "memory": "170Mi"
              },
              "requests": {
                "cpu": "100m",
                "memory": "70Mi"
              }
            },
            "volumeMounts": [
              {
                "name": "config-volume",
                "readOnly": true,
                "mountPath": "/etc/coredns"
              }
            ],
            "livenessProbe": {
              "httpGet": {
                "path": "/health",
                "port": 8080,
                "scheme": "HTTP"
              },
              "initialDelaySeconds": 60,
              "timeoutSeconds": 5,
              "periodSeconds": 10,
              "successThreshold": 1,
              "failureThreshold": 5
            },
            "readinessProbe": {
              "httpGet": {
                "path": "/health",
                "port": 8080,
                "scheme": "HTTP"
              },
              "timeoutSeconds": 1,
              "periodSeconds": 10,
              "successThreshold": 1,
              "failureThreshold": 3
            },
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "IfNotPresent",
            "securityContext": {
              "capabilities": {
                "add": [
                  "NET_BIND_SERVICE"
                ],
                "drop": [
                  "all"
                ]
              },
              "readOnlyRootFilesystem": true,
              "allowPrivilegeEscalation": false
            }
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "Default",
        "nodeSelector": {
          "beta.kubernetes.io/os": "linux"
        },
        "serviceAccountName": "coredns",
        "serviceAccount": "coredns",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "CriticalAddonsOnly",
            "operator": "Exists"
          },
          {
            "key": "node-role.kubernetes.io/master",
            "effect": "NoSchedule"
          }
        ],
        "priorityClassName": "system-cluster-critical"
      }
    },
    "strategy": {
      "type": "RollingUpdate",
      "rollingUpdate": {
        "maxUnavailable": 1,
        "maxSurge": "25%"
      }
    },
    "revisionHistoryLimit": 10,
    "progressDeadlineSeconds": 600
  },
  "status": {
    "observedGeneration": 10,
    "replicas": 2,
    "updatedReplicas": 2,
    "readyReplicas": 2,
    "availableReplicas": 2,
    "conditions": [
      {
        "type": "Progressing",
        "status": "True",
        "lastUpdateTime": "2019-07-16T00:59:47Z",
        "lastTransitionTime": "2019-07-16T00:59:34Z",
        "reason": "NewReplicaSetAvailable",
        "message": "ReplicaSet \"coredns-5c98db65d4\" has successfully progressed."
      },
      {
        "type": "Available",
        "status": "True",
        "lastUpdateTime": "2019-07-20T07:55:19Z",
        "lastTransitionTime": "2019-07-20T07:55:19Z",
        "reason": "MinimumReplicasAvailable",
        "message": "Deployment has minimum availability."
      }
    ]
  }
}
feng@ubuntu:~/k8s-v2$ 
```

## how to find k8s object 'configmap' details

* find in which VERSION configmap was defined
```
feng@ubuntu:~/k8s-v2$ k explain configmap
KIND:     ConfigMap
VERSION:  v1

DESCRIPTION:
     ConfigMap holds configuration data for pods to consume.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   binaryData   <map[string]string>
     BinaryData contains the binary data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. BinaryData can contain byte sequences that are
     not in the UTF-8 range. The keys stored in BinaryData must not overlap with
     the ones in the Data field, this is enforced during validation process.
     Using this field will require 1.10+ apiserver and kubelet.

   data <map[string]string>
     Data contains the configuration data. Each key must consist of alphanumeric
     characters, '-', '_' or '.'. Values with non-UTF-8 byte sequences must use
     the BinaryData field. The keys stored in Data must not overlap with the
     keys in the BinaryData field, this is enforced during validation process.

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

feng@ubuntu:~/k8s-v2$ 
```

* confirm configmap is defined in api/v1
```
feng@ubuntu:~/k8s-v2$ k proxy &
[1] 15040
feng@ubuntu:~/k8s-v2$ Starting to serve on 127.0.0.1:8001

feng@ubuntu:~/k8s-v2$ jobs
[1]+  Running                 kubectl proxy &
feng@ubuntu:~/k8s-v2$ curl http://localhost:8001/api/v1 | jq '.resources[].name' | grep configmap
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10098    0 10098    0     0   519k      0 --:--:-- --:--:-- --:--:--  519k
"configmaps"
feng@ubuntu:~/k8s-v2$ 

feng@ubuntu:~/k8s-v2$ curl http://localhost:8001/api/v1 | jq '.resources[2]'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10098    0 10098    0     0  2465k      0 --:--:-- --:--:-- --:--:-- 3287k
{
  "name": "configmaps",
  "singularName": "",
  "namespaced": true,
  "kind": "ConfigMap",
  "verbs": [
    "create",
    "delete",
    "deletecollection",
    "get",
    "list",
    "patch",
    "update",
    "watch"
  ],
  "shortNames": [
    "cm"
  ],
  "storageVersionHash": "qFsyl6wFWjQ="
}
feng@ubuntu:~/k8s-v2$ 
```

## appendix
* ssh to minikube node
```
feng@ubuntu:~/k8s-v2$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ docker images | egrep -v 'k8s'
REPOSITORY                                                       TAG                 IMAGE ID            CREATED             SIZE
fen9li/koala                                                     v1                  e9619e1f282c        4 hours ago         660MB
busybox                                                          latest              db8ee88ad75f        40 hours ago        1.22MB
nginx                                                            alpine              ea1193fd3dde        2 weeks ago         20.6MB
quay.io/kubernetes-ingress-controller/nginx-ingress-controller   0.23.0              42d47fe0c78f        4 months ago        591MB
gcr.io/google_containers/defaultbackend                          1.4                 846921f0fe0e        21 months ago       4.84MB
tutum/dnsutils                                                   latest              6cd78a6d3256        4 years ago         200MB
tutum/curl                                                       latest              01176385d84a        4 years ago         224MB
$ exit
logout
feng@ubuntu:~/k8s-v2$ 
```
