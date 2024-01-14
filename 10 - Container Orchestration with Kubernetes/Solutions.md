******

<details>
<summary>Exercise 0: Prerequisites </summary>
 <br />

1) installed helm
```shell
(base) ➜  ~ brew install helm
Running `brew update --auto-update`...
==> Auto-updated Homebrew!
Updated 2 taps (homebrew/core and homebrew/cask).
==> New Formulae
cargo-llvm-cov            hopscotch-map             libwapcaplet              pivit                     wasmedge
halp                      libnsbmp                  netsurf-buildsystem       sugarjar
==> New Casks
lightburn                                                        openthesaurus-deutsch

You have 39 outdated formulae and 1 outdated cask installed.

Warning: helm 3.13.3 is already installed and up-to-date.
To reinstall 3.13.3, run:
  brew reinstall helm
````

2) installed helmfile
```shell
(base) ➜  ~ brew install helmfile
Warning: helmfile 0.160.0 is already installed and up-to-date.
To reinstall 0.160.0, run:
  brew reinstall helmfile
```

3) create kubernetes-exercises repo in aws
Created `kubernetes-exercises` repository in AWS ECR.

4) pushed image with java application in kubernetes-exercises repo
```dockerfile
FROM gradle:7.6.0-jdk17-alpine as builder

WORKDIR /kubernetes-exercises-app

COPY --chown=gradle:gradle build.gradle .
COPY --chown=gradle:gradle settings.gradle .
COPY --chown=gradle:gradle src src

RUN gradle build

FROM openjdk:17-jdk-alpine as release

WORKDIR /kubernetes-exercises-app

RUN addgroup --gid 1001 --system kubernetes-exercises && \
    adduser -S --uid 1001 -G kubernetes-exercises kubernetes-exercises && \
    chown -R kubernetes-exercises:kubernetes-exercises /kubernetes-exercises-app

RUN mkdir -p /var/logs/kubernetes-exercises && \
    chown -R kubernetes-exercises:kubernetes-exercises /var/logs/kubernetes-exercises

COPY --chown=kubernetes-exercises:kubernetes-exercises --from=builder /kubernetes-exercises-app/build/libs/kubernetes-exercises-project.jar .

USER 1001:1001

CMD ["java", "-jar", "kubernetes-exercises-project.jar"]
```
```shell
(base) ➜  Exercises aws ecr get-login-password --profile ecs-user
[hash]

(base) ➜  Exercises docker login --username AWS -p [hash] 199054578927.dkr.ecr.eu-central-1.amazonaws.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```
```shell
(base) ➜  Exercises cd kubernetes-exercises 
(base) ➜  kubernetes-exercises docker build -t kubernetes-exercises .
[+] Building 1.8s (17/17) FINISHED                                                                                                                                                                 docker:desktop-linux
 => [internal] load .dockerignore                                                                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                                                                                                                               0.1s
 => => transferring dockerfile: 901B                                                                                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/gradle:7.6.0-jdk17-alpine                                                                                                                                       1.5s
 => [internal] load metadata for docker.io/library/openjdk:17-jdk-alpine                                                                                                                                           1.4s
 => [builder 1/6] FROM docker.io/library/gradle:7.6.0-jdk17-alpine@sha256:74f07921e21e5ef6935b60ed014811e34b552e1d9f020a7735df6ffb7bbf7860                                                                         0.0s
 => [internal] load build context                                                                                                                                                                                  0.0s
 => => transferring context: 1.19kB                                                                                                                                                                                0.0s
 => [release 1/5] FROM docker.io/library/openjdk:17-jdk-alpine@sha256:4b6abae565492dbe9e7a894137c966a7485154238902f2f25e9dbd9784383d81                                                                             0.0s
 => CACHED [release 2/5] WORKDIR /kubernetes-exercises-app                                                                                                                                                         0.0s
 => CACHED [release 3/5] RUN addgroup --gid 1001 --system kubernetes-exercises &&     adduser -S --uid 1001 -G kubernetes-exercises kubernetes-exercises &&     chown -R kubernetes-exercises:kubernetes-exercise  0.0s
 => CACHED [release 4/5] RUN mkdir -p /var/logs/kubernetes-exercises &&     chown -R kubernetes-exercises:kubernetes-exercises /var/logs/kubernetes-exercises                                                      0.0s
 => CACHED [builder 2/6] WORKDIR /kubernetes-exercises-app                                                                                                                                                         0.0s
 => CACHED [builder 3/6] COPY --chown=gradle:gradle build.gradle .                                                                                                                                                 0.0s
 => CACHED [builder 4/6] COPY --chown=gradle:gradle settings.gradle .                                                                                                                                              0.0s
 => CACHED [builder 5/6] COPY --chown=gradle:gradle src src                                                                                                                                                        0.0s
 => CACHED [builder 6/6] RUN gradle build                                                                                                                                                                          0.0s
 => CACHED [release 5/5] COPY --chown=kubernetes-exercises:kubernetes-exercises --from=builder /kubernetes-exercises-app/build/libs/kubernetes-exercises-project.jar .                                             0.0s
 => exporting to image                                                                                                                                                                                             0.0s
 => => exporting layers                                                                                                                                                                                            0.0s
 => => writing image sha256:abbe58f113f641c1b4979f4da719b43004ea3885a0f46fa42d6dd3112d7258b0                                                                                                                       0.0s
 => => naming to docker.io/library/kubernetes-exercises                                                                                                                                                            0.0s
```
```shell
(base) ➜  kubernetes-exercises docker tag kubernetes-exercises:latest 199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises:1.0   

(base) ➜  kubernetes-exercises docker images
REPOSITORY                                                             TAG                                                                          IMAGE ID       CREATED         SIZE
kubernetes-exercises-kubernetes-exercises-app                          latest                                                                       fdada5f3b32e   6 days ago      348MB
199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises   1.0                                                                          abbe58f113f6   6 days ago      348MB
kubernetes-exercises                                                   latest                                                                       abbe58f113f6   6 days ago      348MB

(base) ➜  kubernetes-exercises docker push 199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises:1.0   
The push refers to repository [199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises]
6d39b022f34b: Pushed 
5ee71269c658: Pushed 
24430447398c: Pushed 
8678eff4549a: Pushed 
34f7184834b2: Pushed 
5836ece05bfd: Pushed 
72e830a4dff5: Pushed 
1.0: digest: sha256:3c376b64cb52c6ac999042b96e29d98bbd5ab55ec208a75655cb5f03e82db28f size: 1785
```
</details>

******

<details>
<summary>Exercise 1: Create a Kubernetes cluster </summary>
 <br />

#### Create minikube cluster
```shell
(base) ➜  ~ minikube start --driver docker
😄  minikube v1.26.0 on Darwin 13.3.1
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🏃  Updating the running docker "minikube" container ...
🐳  Preparing Kubernetes v1.24.1 on Docker 20.10.17 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.27.2, which may have incompatibilites with Kubernetes 1.24.1.
    ▪ Want kubectl v1.24.1? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

(base) ➜  ~ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m7s
```

</details>

******

<details>
<summary>Exercise 2: Deploy Mysql with 2 replicas </summary>
 <br />

#### Add bitnami repository
```shell
(base) ➜  ~ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

#### Search for mysql chart in bitnami
```shell
(base) ➜  ~ helm search repo bitnami/mysql
NAME         	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/mysql	9.15.0       	8.0.35     	MySQL is a fast, reliable, scalable, and easy t...
```

#### Based on [MySQL chart configuration](https://github.com/bitnami/charts/tree/main/bitnami/mysql) override custom values defined in chart configuration.
`helm-mysql-values-minikube.yaml`
```yaml
architecture: replication
auth:
  rootPassword: secret
  database: team-app
  username: dev
  password: secret
secondary:
  replicaCount: 2
  persistence:
    storageClass: standard
```

#### Install mysql chart
```shell
(base) ➜ helm install mysql --values ./k8s-deployment/helm-mysql-values-minikube.yaml bitnami/mysql   
NAME: mysql
LAST DEPLOYED: Tue Dec 26 21:17:19 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 9.15.0
APP VERSION: 8.0.35

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

Services:

  echo Primary: mysql-primary.default.svc.cluster.local:3306
  echo Secondary: mysql-secondary.default.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.35-debian-11-r0 --namespace default --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mysql-primary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

  3. To connect to secondary service (read-only):

      mysql -h mysql-secondary.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
```

Check mysql helm chart installation
```shell
(base) ➜  Exercises helm ls
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mysql   default         1               2023-12-26 21:28:09.109792 +0100 CET    deployed        mysql-9.15.0    8.0.35     

(base) ➜  Exercises kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
mysql-primary-0     1/1     Running   0          5m54s
mysql-secondary-0   1/1     Running   0          109s
mysql-secondary-1   1/1     Running   0          68s
```
</details>

******

<details>
<summary>Exercise 3: Deploy your Java Application with 2 replicas </summary>
 <br />

1) Create secret of type `kubernetes.io/dockerconfigjson` for pull image from private repository
```shell
(base) ➜  Exercises aws ecr get-login-password --profile ecs-user
[hash]

(base) ➜  Exercises minikube ssh
docker@minikube:~$ pwd
/home/docker
docker@minikube:~$ ls -a
.  ..  .bash_logout  .bashrc  .profile  .ssh  .sudo_as_admin_successful
docker@minikube:~$ docker login --username AWS -p [hash] 199054578927.dkr.ecr.eu-central-1.amazonaws.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/docker/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
docker@minikube:~$ ls -la
total 36
drwxr-xr-x 1 docker docker 4096 Jan  8 00:01 .
drwxr-xr-x 1 root   root   4096 Jun 15  2022 ..
-rw-r--r-- 1 docker docker  220 Jun 15  2022 .bash_logout
-rw-r--r-- 1 docker docker 3771 Jun 15  2022 .bashrc
drwx------ 2 docker docker 4096 Jan  8 00:01 .docker
-rw-r--r-- 1 docker docker  807 Jun 15  2022 .profile
drwxr-xr-x 1 docker docker 4096 Dec 26 16:39 .ssh
-rw-r--r-- 1 docker docker    0 Dec 26 16:39 .sudo_as_admin_successful
docker@minikube:~$ cat .docker/config.json 
{
        "auths": {
                "199054578927.dkr.ecr.eu-central-1.amazonaws.com": {
                        "auth": "[hash]"
                }
        }
}
docker@minikube:~$ exit
logout

(base) ➜  Exercises minikube cp minikube:/home/docker/.docker/config.json ./config.json
(base) ➜  Exercises ls 
config.json  k8s-deployment  kubernetes-exercises

(base) ➜  Exercises kubectl create secret generic my-registry-key \
> --from-file=.dockerconfigjson=./config.json \
> --type=kubernetes.io/dockerconfigjson
secret/my-registry-key created

(base) ➜  Exercises kubectl get secret my-registry-key -o yaml
apiVersion: v1
data:
  .dockerconfigjson: [hash]
kind: Secret
metadata:
  creationTimestamp: "2024-01-08T00:17:31Z"
  name: my-registry-key
  namespace: default
  resourceVersion: "229526"
  uid: 1524d760-7a8f-4c8b-82f4-d03fc339c465
type: kubernetes.io/dockerconfigjson
```
2) Create `mysql-secret` and `mysql-config` components
`mysql-config.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  DB_SERVER: "mysql-primary"
```
`mysql-secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  DB_USER: ZGV2
  DB_PWD: c2VjcmV0
  DB_NAME: dGVhbS1hcHA=
  DB_ROOT_PWD: 'dG9vcg=='
```
3) create deployment && service components for java app
`kubernetes-excercises.yaml`
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-excercises
spec:
  replicas: 2
  selector:
    matchLabels:
      app: kubernetes-excercises
  template:
    metadata:
      labels:
        app: kubernetes-excercises
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: kubernetes-excercises
        image: 199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises:1.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_SERVER
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: DB_SERVER
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: DB_USER
        - name: DB_PWD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: DB_PWD
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: DB_NAME
---
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-excercises
spec:
  type: ClusterIP
  selector:
    app: kubernetes-excercises
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```
Apply configmaps and secret components
```shell
(base) ➜  Exercises kubectl apply -f ./k8s-deployment/mysql-config.yaml -f ./k8s-deployment/mysql-secret.yaml
configmap/mysql-config created
secret/mysql-secret created

(base) ➜  Exercises kubectl get configmaps mysql-config -o yaml
apiVersion: v1
data:
  DB_SERVER: mysql-primary
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"DB_SERVER":"mysql-primary"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"mysql-config","namespace":"default"}}
  creationTimestamp: "2024-01-08T00:38:21Z"
  name: mysql-config
  namespace: default
  resourceVersion: "230401"
  uid: 859c9c74-070a-45f8-93b7-cf1ddde99825

(base) ➜  Exercises kubectl get secrets mysql-secret -o yaml
apiVersion: v1
data:
  DB_NAME: dGVhbS1hcHA=
  DB_PWD: c2VjcmV0
  DB_USER: ZGV2
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"DB_NAME":"dGVhbS1hcHA=","DB_PWD":"c2VjcmV0","DB_USER":"ZGV2"},"kind":"Secret","metadata":{"annotations":{},"name":"mysql-secret","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2024-01-08T00:38:21Z"
  name: mysql-secret
  namespace: default
  resourceVersion: "230402"
  uid: ca246371-b741-4fb7-817c-acefb2c506a6
type: Opaque

(base) ➜  Exercises kubectl get secrets mysql-secret -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'    
DB_NAME: team-app
DB_PWD: secret
DB_USER: dev
```
Apply deployment and service components
```shell
(base) ➜  Exercises kubectl apply -f ./k8s-deployment/kubernetes-exercises.yaml 
deployment.apps/kubernetes-excercises created
service/kubernetes-excercises created

(base) ➜  Exercises kubectl get pods
NAME                                     READY   STATUS    RESTARTS        AGE
kubernetes-excercises-8596658f79-kx9rt   1/1     Running   0               66s
kubernetes-excercises-8596658f79-ss5wj   1/1     Running   0               66s
mysql-primary-0                          1/1     Running   2 (4h25m ago)   12d
mysql-secondary-0                        1/1     Running   2 (4h25m ago)   12d
mysql-secondary-1                        1/1     Running   2 (4h25m ago)   12d

(base) ➜  Exercises kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP    12d
kubernetes-excercises      ClusterIP   10.98.9.199      <none>        8080/TCP   76s
mysql-primary              ClusterIP   10.106.250.66    <none>        3306/TCP   12d
mysql-primary-headless     ClusterIP   None             <none>        3306/TCP   12d
mysql-secondary            ClusterIP   10.107.105.158   <none>        3306/TCP   12d
mysql-secondary-headless   ClusterIP   None             <none>        3306/TCP   12d

(base) ➜  Exercises kubectl logs svc/kubernetes-excercises
Found 2 pods, using pod/kubernetes-excercises-8596658f79-ss5wj

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::       (v3.1.0-SNAPSHOT)

2024-01-08T00:43:39.688Z  INFO 1 --- [           main] com.example.Application                  : Starting Application using Java 17-ea with PID 1 (/kubernetes-exercises-app/kubernetes-exercises-project.jar started by kubernetes-exercises in /kubernetes-exercises-app)
2024-01-08T00:43:39.703Z  INFO 1 --- [           main] com.example.Application                  : No active profile set, falling back to 1 default profile: "default"
2024-01-08T00:43:52.101Z  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2024-01-08T00:43:52.389Z  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2024-01-08T00:43:52.390Z  INFO 1 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.8]
2024-01-08T00:43:54.593Z  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2024-01-08T00:43:54.689Z  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 14297 ms
2024-01-08T00:43:59.920Z  INFO 1 --- [           main] com.example.Application                  : Java app started
2024-01-08T00:44:03.681Z  INFO 1 --- [           main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [static/index.html]
2024-01-08T00:44:05.682Z  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2024-01-08T00:44:05.888Z  INFO 1 --- [           main] com.example.Application                  : Started Application in 30.39 seconds (process running for 34.166)
```

</details>

******

<details>
<summary>Exercise 4: Deploy phpmyadmin </summary>
 <br />

1) Create deployment and service components for phpmyadmin
`phpmyadmin.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin-deployment
  labels:
    app: phpmyadmin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
      - name: phpmyadmin
        image: phpmyadmin/phpmyadmin:5
        ports:
        - containerPort: 80
          protocol: TCP
        env:
          - name: PMA_HOST
            valueFrom:
              configMapKeyRef:
                name: mysql-config
                key: DB_SERVER
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-secret
                key: DB_ROOT_PWD
---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
spec:
  selector:
    app: phpmyadmin
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 80
```
2) Apply deployment and service components in kubernetes cluster
```shell
(base) ➜  Exercises kubectl apply -f ./k8s-deployment/phpmyadmin.yaml  
deployment.apps/phpmyadmin-deployment created
service/phpmyadmin-service created

(base) ➜  Exercises kubectl get pods                                 
NAME                                     READY   STATUS    RESTARTS        AGE
kubernetes-excercises-8596658f79-kx9rt   1/1     Running   4 (88s ago)     2d22h
kubernetes-excercises-8596658f79-ss5wj   1/1     Running   5 (88s ago)     2d22h
mysql-primary-0                          1/1     Running   4 (2m41s ago)   15d
mysql-secondary-0                        1/1     Running   4 (2m41s ago)   15d
mysql-secondary-1                        1/1     Running   4 (2m41s ago)   15d
phpmyadmin-deployment-5d4485dfc9-v2dxh   1/1     Running   0               112s

(base) ➜  Exercises kubectl get services
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP    15d
kubernetes-excercises      ClusterIP   10.98.9.199      <none>        8080/TCP   2d22h
mysql-primary              ClusterIP   10.106.250.66    <none>        3306/TCP   15d
mysql-primary-headless     ClusterIP   None             <none>        3306/TCP   15d
mysql-secondary            ClusterIP   10.107.105.158   <none>        3306/TCP   15d
mysql-secondary-headless   ClusterIP   None             <none>        3306/TCP   15d
phpmyadmin-service         ClusterIP   10.100.85.38     <none>        8081/TCP   5m24s
```

</details>

******

<details>
<summary>Exercise 5: Deploy Ingress Controller </summary>
 <br />

```shell
(base) ➜  Exercises minikube addons enable ingress
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.2.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```
</details>

******

<details>
<summary>Exercise 6: Create Ingress rule </summary>
 <br />

1) Create ingress component
`kubernetes-exercises-ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-exercises-ingress
spec:
  rules:
  - host: kubernetes-exercises.com
    http:
      paths:
        - path: /
          pathType: Exact
          backend:
            service:
              name: kubernetes-excercises
              port:
                number: 8080
  ingressClassName: nginx
```
2) Add `127.0.0.1 kubernetes-exercises.com` in `/etc/hosts` file
3) Apply ingress component in kubernetes cluster
```shell
(base) ➜  Exercises kubectl apply -f ./k8s-deployment/kubernetes-exercises-ingress-minikube.yaml 
ingress.networking.k8s.io/kubernetes-exercises-ingress created

(base) ➜  Exercises kubectl get ingress
NAME                           CLASS   HOSTS                      ADDRESS        PORTS   AGE
kubernetes-exercises-ingress   nginx   kubernetes-exercises.com   192.168.49.2   80      10s

(base) ➜  Exercises kubectl describe ingress kubernetes-exercises-ingress 
Name:             kubernetes-exercises-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.49.2
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host                      Path  Backends
  ----                      ----  --------
  kubernetes-exercises.com  
                            /   kubernetes-excercises:8080 (172.17.0.5:8080,172.17.0.6:8080)
Annotations:                <none>
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    15s (x2 over 21s)  nginx-ingress-controller  Scheduled for sync
```
4) Run `minikube tunnel`
```shell
(base) ➜  Exercises minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress kubernetes-exercises-ingress requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service kubernetes-exercises-ingress.
Password:
```
5) Access application at `kubernetes-exercises.com`

</details>

******

<details>
<summary>Exercise 7: Port-forward for phpmyadmin </summary>
 <br />

```shell
(base) ➜  Exercises kubectl port-forward svc/phpmyadmin-service 8081:8081
Forwarding from 127.0.0.1:8081 -> 80
Forwarding from [::1]:8081 -> 80
Handling connection for 8081
```
Access phpmyadmin at `127.0.0.1:8081`
Login using credentials: `dev/secret`

</details>

******

<details>
<summary>Exercise 8: Create Helm Chart for Java App </summary>
 <br />

1) Create helm chart
```shell
(base) ➜  Exercises mkdir charts 
(base) ➜  Exercises cd charts 
(base) ➜  charts helm create kubernetes-exercises
Creating kubernetes-exercises
```
2) Remove non-needed templates
3) Create templates for `kubernetes-exercises-deployment`, `kubernetes-exercises-service`, `kubernetes-exercises-ingress`, `mysql-config`, `mysql-secret`
`kubernetes-exercises-deployment.yaml`
```kubernetes helm
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}-deployment
  labels:
    app: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      imagePullSecrets:
        - name: {{ .Values.registrySecretName }}
      containers:
        - name: {{ .Values.appName }}
          image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
          ports:
            - containerPort: {{ .Values.containerPort }}
          env:
          {{- range $key, $value := .Values.containerEnvVars }}
            - name: {{ $key }}
              value: {{ $value | quote }}
          {{- end }}

          {{- range $key, $value := .Values.containerConfigMapEnvVars }}
            - name: {{ $key }}
              valueFrom:
                configMapKeyRef:
                  # in loop, we lose global context, but can access global context with $
                  # $ is 1 variable that is always global and will always point to the root context
                  # so $.Value instead of .Values
                  name: {{ $.Values.configMapName }}
                  key: {{ $key }}
          {{- end }}

          {{- range $key, $value := .Values.containerSecretKeyEnvVars }}
            - name: {{ $key }}
              valueFrom:
                secretKeyRef:
                  name: {{ $.Values.secretName }}
                  key: {{ $key }}
          {{- end }}
```

`kubernetes-exercises-service`
```kubernetes helm
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}-service
spec:
  type: ClusterIP
  selector:
    app: {{ .Values.appName }}
  ports:
    - protocol: TCP
      port: {{ .Values.servicePort }}
      targetPort: {{ .Values.containerPort }}
```

`kubernetes-exercises-ingress.yaml`
```kubernetes helm
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: {{ .Values.appName }}-ingress
spec:
  rules:
  - host: {{ .Values.ingressHostName }}
    http:
      paths:
        - path: {{ .Values.ingressPath }}
          pathType: {{ .Values.ingressPathType }}
          backend:
            service:
              name: {{ .Values.appName }}-service
              port:
                number: {{ .Values.servicePort }}
```

`mysql-config.yaml`
```kubernetes helm
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Values.configMapName }}
data:
  {{- range $key, $value := .Values.containerConfigMapEnvVars }}
  {{ $key }}: {{ $value }}
  {{- end }}
```

`mysql-secret.yaml`
```kubernetes helm
apiVersion: v1
kind: Secret
metadata:
    name: {{ .Values.secretName }}
type: Opaque
data:
  {{- range $key, $value := .Values.containerSecretKeyEnvVars }}
  {{ $key }}: {{ $value }}
  {{- end }}
```

4) Set default values
`values.yaml`
```yaml
appName: app

appReplicas: 1

registrySecretName: registry-secret

appImage: image

appVersion: version

containerPort: 80

containerEnvVars: {}

configMapName: config

containerConfigMapEnvVars: {}

secretName: secret

containerSecretKeyEnvVars: {}

servicePort: 80

ingressHostName: app.com

ingressPathType: Exact

ingressPath: /
```

5) Create real values for components that override default
`kubernetes-exercises-values.yaml`
```yaml
appName: kubernetes-exercises

appReplicas: 3

registrySecretName: my-registry-key

appImage: 199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises

appVersion: "1.0"

containerPort: 8080

containerEnvVars: {}

configMapName: mysql-config

containerConfigMapEnvVars:
  DB_SERVER: mysql-primary

secretName: mysql-secret

containerSecretKeyEnvVars:
  DB_USER: ZGV2
  DB_PWD: c2VjcmV0
  DB_NAME: dGVhbS1hcHA=
  DB_ROOT_PWD: 'dG9vcg=='

servicePort: 8080

ingressHostName: kubernetes-exercises.com

ingressPathType: Exact

ingressPath: /
```

6) Validate template files
```shell
(base) ➜  Exercises helm lint -f kubernetes-exercises-values.yaml ./charts/kubernetes-exercises    
==> Linting ./charts/kubernetes-exercises
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed

(base) ➜  Exercises helm template -f kubernetes-exercises-values.yaml ./charts/kubernetes-exercises
---
# Source: kubernetes-exercises/templates/mysql-secret.yaml
apiVersion: v1
kind: Secret
metadata:
    name: mysql-secret
type: Opaque
data:
  DB_NAME: dGVhbS1hcHA=
  DB_PWD: c2VjcmV0
  DB_ROOT_PWD: dG9vcg==
  DB_USER: ZGV2
---
# Source: kubernetes-exercises/templates/mysql-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: mysql-config
data:
  DB_SERVER: mysql-primary
---
# Source: kubernetes-exercises/templates/kubernetes-exercises-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-exercises-service
spec:
  type: ClusterIP
  selector:
    app: kubernetes-exercises
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
---
# Source: kubernetes-exercises/templates/kubernetes-exercises-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-exercises-deployment
  labels:
    app: kubernetes-exercises
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubernetes-exercises
  template:
    metadata:
      labels:
        app: kubernetes-exercises
    spec:
      imagePullSecrets:
        - name: my-registry-key
      containers:
        - name: kubernetes-exercises
          image: "199054578927.dkr.ecr.eu-central-1.amazonaws.com/kubernetes-exercises:1.0"
          ports:
            - containerPort: 8080
          env:
            - name: DB_SERVER
              valueFrom:
                configMapKeyRef:
                  # in loop, we lose global context, but can access global context with $
                  # $ is 1 variable that is always global and will always point to the root context
                  # so $.Value instead of .Values
                  name: mysql-config
                  key: DB_SERVER
            - name: DB_NAME
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: DB_NAME
            - name: DB_PWD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: DB_PWD
            - name: DB_ROOT_PWD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: DB_ROOT_PWD
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: DB_USER
---
# Source: kubernetes-exercises/templates/kubernetes-exercises-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: kubernetes-exercises-ingress
spec:
  rules:
  - host: kubernetes-exercises.com
    http:
      paths:
        - path: /
          pathType: Exact
          backend:
            service:
              name: kubernetes-exercises-service
              port:
                number: 8080
```
7) Deploy kubernetes-exercises into k8s cluster using helm chart
```shell
(base) ➜  Exercises helm install -f ./kubernetes-exercises-values.yaml kubernetes-exercises ./charts/kubernetes-exercises
NAME: kubernetes-exercises
LAST DEPLOYED: Sun Jan 14 14:36:24 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

(base) ➜  Exercises helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
kubernetes-exercises    default         1               2024-01-14 14:36:24.569023 +0100 CET    deployed        kubernetes-exercises-0.1.0      1.16.0     
mysql                   default         1               2023-12-26 21:53:42.690612 +0100 CET    deployed        mysql-9.15.0                    8.0.35     

(base) ➜  Exercises kubectl get pods
NAME                                               READY   STATUS    RESTARTS      AGE
kubernetes-exercises-deployment-5649886f9d-dg4mg   1/1     Running   0             49s
kubernetes-exercises-deployment-5649886f9d-nbnh4   1/1     Running   0             49s
kubernetes-exercises-deployment-5649886f9d-xwpzh   1/1     Running   0             49s
mysql-primary-0                                    1/1     Running   5 (38h ago)   18d
mysql-secondary-0                                  1/1     Running   5 (38h ago)   18d
mysql-secondary-1                                  1/1     Running   5 (38h ago)   18d
```

8) Uninstall Helm charts
```shell
(base) ➜  Exercises helm uninstall kubernetes-exercises
release "kubernetes-exercises" uninstalled

(base) ➜  Exercises kubectl get pods
NAME                READY   STATUS    RESTARTS      AGE
mysql-primary-0     1/1     Running   5 (38h ago)   18d
mysql-secondary-0   1/1     Running   5 (38h ago)   18d
mysql-secondary-1   1/1     Running   5 (38h ago)   18d
```

9) Create Helmfile
`helmfile.yaml`
```yaml
releases:
  - name: kubernetes-exercises
    chart: charts/kubernetes-exercises
    values:
      - kubernetes-exercises-values.yaml
```

10) Deploy kubernetes-exercises into k8s cluster using helmfile (Sync helmfile)
```shell
(base) ➜  Exercises helmfile sync
Building dependency release=kubernetes-exercises, chart=charts/kubernetes-exercises
Upgrading release=kubernetes-exercises, chart=charts/kubernetes-exercises
Release "kubernetes-exercises" does not exist. Installing it now.
NAME: kubernetes-exercises
LAST DEPLOYED: Sun Jan 14 14:46:10 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

Listing releases matching ^kubernetes-exercises$
kubernetes-exercises    default         1               2024-01-14 14:46:10.587352 +0100 CET    deployed        kubernetes-exercises-0.1.0      1.16.0     


UPDATED RELEASES:
NAME                   CHART                         VERSION   DURATION
kubernetes-exercises   charts/kubernetes-exercises   0.1.0           1s

(base) ➜  Exercises helmfile list
NAME                    NAMESPACE       ENABLED INSTALLED       LABELS  CHART                           VERSION
kubernetes-exercises                    true    true                    charts/kubernetes-exercises            
       
(base) ➜  Exercises kubectl get pods
NAME                                               READY   STATUS    RESTARTS      AGE
kubernetes-exercises-deployment-5649886f9d-44dfq   1/1     Running   0             21s
kubernetes-exercises-deployment-5649886f9d-6qg6c   1/1     Running   0             21s
kubernetes-exercises-deployment-5649886f9d-fz2vn   1/1     Running   0             21s
mysql-primary-0                                    1/1     Running   5 (38h ago)   18d
mysql-secondary-0                                  1/1     Running   5 (38h ago)   18d
mysql-secondary-1                                  1/1     Running   5 (38h ago)   18d
```

11) Host the chart in its own git repository
<br />
[Kubernetes exercises helm chart GitHub Repository](https://github.com/ireshniov/kubernetes-exercises-helm-chart)
</details>

******
