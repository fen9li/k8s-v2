
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
