# Docker challenge from docker-hub

Created by: Susandar Htet
Created time: October 3, 2023 3:17 PM
Tags: Kubernetes, docker

OS version - Ubuntu 18.04 LTS

Docker version - Client: 24.0.2, Server: 24.0.2

Kubectl version - v1.27.4

```bash
~$ kubectl version --client
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:20:54Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
```

Pull the docker image to your local server

```bash
$docker pull derekdemo/tech-challenge:latest
```

Then run the docker with port 8080:

```bash
$docker run -p 8080:80 -d derekdemo/tech-challenge:latest
$docker ps
```

Then access from browser

[http://localhost:8080]

```

			Follow the below challenges to uncover the answers:
				#1: Check the logs of this container
				#2: Perform a web request to the container using 'example.com' as the host header
				#3: Create a Kubernetes service named 'tech-challenge' and use the container image in a pod in the same namespace, tip: check the logs once it's started
				#4: Check the contents of /app/challenge-4.txt inside the container
				#5: Add a readiness probe to check the /healthz path of the container, tip: check the logs once a probe has run
```

#1 Check the logs of this container

Ans : 

```bash
$docker logs 600ffac7e14a
Answer #1: I-found-this-in-the-log
```

#2 Perform a web request to the container using 'example.com' as the host header
Results:

~ ➜ curl -H "Host: [example.com](http://example.com/)" [http://localhost:8080](http://localhost:8080/)
Answer #2: Nice-keep-it-up!%



```bash
~ ➜ curl -H "Host: example.com" http://localhost:8080
Answer #2: Nice-keep-it-up!%
```

#3: Create a Kubernetes service named 'tech-challenge' and use the container image in a pod in the same namespace, tip: check the logs once it's started

Create tech challenge deployment and service in default namespace

```bash
~$ kubectl run tech-challenge --image=derekdemo/tech-challenge --port=80 --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: tech-challenge
  name: tech-challenge
spec:
  containers:
  - image: derekdemo/tech-challenge
    name: tech-challenge
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

~$ mkdir tech-challenge
~$ cd tech-challenge/
$ vi tech-challenge.yaml
#insert the above yaml output

$ kubectl apply -f tech-challenge.yaml
pod/tech-challenge created
$ kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
tech-challenge   1/1     Running   0          80s
$ kubectl expose pod tech-challenge --type=NodePort --name=tech-challenge-service
service/tech-challenge-service exposed
$ kubectl get svc
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1        <none>        443/TCP        83m
tech-challenge-service   NodePort    10.104.249.124   <none>        80:31815/TCP   7s
$ kubectl logs tech-challenge
Answer #1: I-found-this-in-the-log

```



#4: Check the contents of /app/challenge-4.txt inside the container

Results:

```bash

~# docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED             STATUS             PORTS                                                                                                                                  NAMES
26db08bff6e6   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   About an hour ago   Up About an hour   127.0.0.1:32777->22/tcp, 127.0.0.1:32776->2376/tcp, 127.0.0.1:32775->5000/tcp, 127.0.0.1:32774->8443/tcp, 127.0.0.1:32773->32443/tcp   minikube
4752199a524a   derekdemo/tech-challenge:latest       "./app"                  5 hours ago         Up 5 hours         0.0.0.0:8080->80/tcp, :::8080->80/tcp                                                                                                  awesome_heisenberg
d25e8f2aaa4b   derekdemo/tech-challenge:latest       "./app"                  25 hours ago        Up 25 hours                                                                                                                                               romantic_einstein
~# docker exec -it 4752199a524a /bin/bash
OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin/bash": stat /bin/bash: no such file or directory: unknown

$docker cp  awesome_heisenberg:/app/challenge-4.txt /home/frontiir/tech-challenge/
Successfully copied 2.05kB to /home/frontiir/tech-challenge/
$cd /home/frontiir/tech-challenge
$ls
challenge-4.txt  tech-challenge-service.yaml  tech-challenge.yaml
$cat challenge-4.txt

Answer #4: I-found-the-random-string
```

#5: Add a readiness probe to check the /healthz path of the container, tip: check the logs once a probe has run

Results

```bash
$vi tech-challenge.yaml
apiVersion: v1
kind: Pod
metadata:
  name: tech-challenge
spec:
  containers:
  - name: tech-challenge
    image: derekdemo/tech-challenge
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10

$kubectl replace --force -f tech-challenge.yaml
pod "tech-challenge" deleted
pod/tech-challenge replaced
$kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
tech-challenge   0/1     Running   0          10s
$kubectl logs tech-challenge
Answer #1: I-found-this-in-the-log
Answer #5: Thanks-the-probes-are-working!
Answer #5: Thanks-the-probes-are-working!
Answer #5: Thanks-the-probes-are-working!
Answer #5: Thanks-the-probes-are-working!
```


References >> [https://linuxhint.com/how-to-copy-directory-from-container-to-host/#:~:text=To copy a particular directory from the container to the,it to the host machine](https://linuxhint.com/how-to-copy-directory-from-container-to-host/#:~:text=To%20copy%20a%20particular%20directory%20from%20the%20container%20to%20the,it%20to%20the%20host%20machine).

[https://support.sitecore.com/kb?id=kb_article_view&sysparm_article=KB0383441](https://support.sitecore.com/kb?id=kb_article_view&sysparm_article=KB0383441)

[https://manpages.ubuntu.com/manpages/xenial/man1/docker-cp.1.html#:~:text=For example%2C this command%3A %24,file%2C it will be overwritten](https://manpages.ubuntu.com/manpages/xenial/man1/docker-cp.1.html#:~:text=For%20example%2C%20this%20command%3A%20%24,file%2C%20it%20will%20be%20overwritten).
