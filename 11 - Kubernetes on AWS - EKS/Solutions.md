******

<details>
<summary>Exercise 0: Prerequisites </summary>
 <br />

### Configure Jenkins with its agent

`Dockerfile-agent`
```dockerfile
FROM jenkins/ssh-agent:alpine

RUN apk update
RUN apk add docker docker-cli-compose git curl

RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/v3.18/main/ nodejs=18.18.2-r0 npm

RUN git config --global user.email "jenkins@example.com"
RUN git config --global user.name "jenkins"
```

`Dockerfile`
```dockerfile
FROM jenkins/jenkins:lts-jdk11

USER root

RUN apt-get -y update && \
    apt-get -y install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release && \
    mkdir -m 0755 -p /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    apt-get -y update && \
    apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Install nvm & node & npm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

#RUN nvm install node
RUN . ~/.nvm/nvm.sh && nvm install 18.14.2

RUN git config --global user.email "jenkins@example.com"
RUN git config --global user.name "jenkins"

USER jenkins
```

`run_jenkins_with_docker.sh`
```shell
#!/bin/bash

docker build -t jenkins_with_docker .

docker run --name jenkins_with_docker -p 8080:8080 -p 50000:50000 -d --restart=on-failure \
-v jenkins_home:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkins_with_docker

docker exec -u 0 -it jenkins_with_docker chmod 666 /var/run/docker.sock

docker build -t jenkins_agent --platform=linux/amd64 -f ./Dockerfile-agent .

docker run -d --rm --name=agent1 -p 22:22 --platform=linux/amd64 \
-e "JENKINS_AGENT_SSH_PUBKEY=$(cat ~/.ssh/jenkins_agent_key.pub)" \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkins_agent

docker exec -u 0 -it agent1 chmod 666 /var/run/docker.sock
```

#### Start Jenkins with its agent
```shell
(base) ➜  jenkins sh run_jenkins_with_docker.sh 
```

#### Configure Jenkins agent (install kubectl, aws-iam-authenticator)
```shell
(base) ➜  Exercises docker exec -it agent1 bash
14e9f661bc42:/home/jenkins# apk add kubectl
fetch https://dl-cdn.alpinelinux.org/alpine/v3.19/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.19/community/x86_64/APKINDEX.tar.gz
(1/1) Installing kubectl (1.28.4-r0)
Executing busybox-1.36.1-r15.trigger
OK: 463 MiB in 66 packages
14e9f661bc42:/home/jenkins# kubectl version
Client Version: v1.28.4
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
The connection to the server localhost:8080 was refused - did you specify the right host or port?
14e9f661bc42:/home/jenkins# uname -m
x86_64
14e9f661bc42:/home/jenkins# curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.14/aws-iam-authenticator_0.6.14_linux_amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 51.0M  100 51.0M    0     0  13.5M      0  0:00:03  0:00:03 --:--:-- 17.3M
14e9f661bc42:/home/jenkins# chmod +x ./aws-iam-authenticator
14e9f661bc42:/home/jenkins# mv ./aws-iam-authenticator /usr/local/bin
14e9f661bc42:/home/jenkins# ls -la /usr/local/bin/aws-iam-authenticator
-rwxr-xr-x    1 root     jenkins   53579776 Mar  3 21:33 /usr/local/bin/aws-iam-authenticator
14e9f661bc42:/home/jenkins# aws-iam-authenticator
A tool to authenticate to Kubernetes using AWS IAM credentials

Usage:
  aws-iam-authenticator [command]

Available Commands:
  add         add IAM entity to an existing aws-auth configmap
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  init        Pre-generate certificate, private key, and kubeconfig files for the server.
  server      Run a webhook validation server suitable that validates tokens using AWS IAM
  token       Authenticate using AWS IAM and get token for Kubernetes
  verify      Verify a token for debugging purpose
  version     Version will output the current build information

Flags:
  -i, --cluster-id ID                 Specify the cluster ID, a unique-per-cluster identifier for your aws-iam-authenticator installation.
  -c, --config filename               Load configuration from filename
      --feature-gates mapStringBool   A set of key=value pairs that describe feature gates for alpha/experimental features. Options are:
                                      AllAlpha=true|false (ALPHA - default=false)
                                      AllBeta=true|false (BETA - default=false)
                                      ConfiguredInitDirectories=true|false (ALPHA - default=false)
                                      IAMIdentityMappingCRD=true|false (ALPHA - default=false)
  -h, --help                          help for aws-iam-authenticator
  -l, --log-format string             Specify log format to use when logging to stderr [text or json] (default "text")

Use "aws-iam-authenticator [command] --help" for more information about a command.
14e9f661bc42:/home/jenkins# 
14e9f661bc42:/home/jenkins# pwd
/home/jenkins
14e9f661bc42:/home/jenkins# mkdir .kube
14e9f661bc42:/home/jenkins# ls -la .kube
total 12
drwxr-sr-x    2 root     jenkins       4096 Mar  3 21:41 .
drwxr-sr-x    1 jenkins  jenkins       4096 Mar  3 21:41 ..
```
#### Create Kube config file for Jenkins:
`config.yaml`
```kubernetes helm
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <certificate-data> # from Cluster info: Certificate authority
    server: <endpoint-url> # from Cluster info: API server endpoint
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: /usr/local/bin/aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "exercise-11-cluster"
```
#### Copy `config.yaml` in Jenkins agent container  as `~/.kube/config` for `jenkins` user
```shell
(base) ➜  Exercises docker cp config.yaml agent1:/home/jenkins/.kube/config
                                             Successfully copied 3.58kB to agent1:/home/jenkins/.kube/config

(base) ➜  Exercises docker exec -it agent1 bash
14e9f661bc42:/home/jenkins# cat .kube/config
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: [certificate-authority-data]
    server: [server]
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: /usr/local/bin/aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "exercise-11-cluster"
14e9f661bc42:/home/jenkins# 
```

#### Install `gettext-base` tool with `envsubst` command in Jenkins agent container
```shell
(base) ➜  Exercises docker exec -it agent1 bash
14e9f661bc42:/home/jenkins# 
14e9f661bc42:/home/jenkins# apk add gettext
(1/6) Installing gettext-envsubst (0.22.3-r0)
(2/6) Installing libgomp (13.2.1_git20231014-r0)
(3/6) Installing gettext-libs (0.22.3-r0)
(4/6) Installing xz-libs (5.4.5-r0)
(5/6) Installing libxml2 (2.11.7-r0)
(6/6) Installing gettext (0.22.3-r0)
Executing busybox-1.36.1-r15.trigger
OK: 467 MiB in 72 packages
14e9f661bc42:/home/jenkins# which envsubst
/usr/bin/envsubst
```

#### Configure jenkins agent
1) Use address from `docker container inspect agent1 | grep IPAddress`
2) Add global `jenkins` credentials for agent access

</details>

******

<details>
<summary>Exercise 1: Create EKS cluster </summary>
 <br />

Conform manual:
```shell
(base) ➜  Exercises eksctl create cluster --help                                                                   
Create a cluster

Usage: eksctl create cluster [flags]

General flags:
  -n, --name string           EKS cluster name (generated if unspecified, e.g. "fabulous-party-1709474431")
      --tags stringToString   Used to tag the AWS resources. List of comma separated KV pairs "k1=v1,k2=v2" (default [])
  -r, --region string         AWS region. Defaults to the value set in your AWS config (~/.aws/config)
      --with-oidc             Enable the IAM OIDC provider
      --zones strings         (auto-select if unspecified)
      --version string        Kubernetes version (valid options: 1.23, 1.24, 1.25, 1.26, 1.27, 1.28) (default "1.27")
  -f, --config-file string    load configuration from a file (or stdin if set to '-')
      --timeout duration      maximum waiting time for any long-running operation (default 25m0s)
      --fargate               Create a Fargate profile scheduling pods in the default and kube-system namespaces onto Fargate
      --dry-run               Dry-run mode that skips cluster creation and outputs a ClusterConfig

Initial nodegroup flags:
      --nodegroup-name string          name of the nodegroup (generated if unspecified, e.g. "ng-1c6f2190")
      --without-nodegroup              if set, initial nodegroup will not be created
  -t, --node-type string               node instance type
  -N, --nodes int                      total number of nodes (for a static ASG) (default 2)
  -m, --nodes-min int                  minimum nodes in ASG (default 2)
  -M, --nodes-max int                  maximum nodes in ASG (default 2)
      --node-volume-size int           node volume size in GB (default 80)
      --node-volume-type string        node volume type (valid options: gp2, gp3, io1, sc1, st1) (default "gp3")
      --max-pods-per-node int          maximum number of pods per node (set automatically if unspecified)
      --ssh-access                     control SSH access for nodes. Uses ~/.ssh/id_rsa.pub as default key path if enabled
      --ssh-public-key string          SSH public key to use for nodes (import from local path, or use existing EC2 key pair)
      --enable-ssm                     Enable AWS Systems Manager (SSM)
      --node-ami string                'auto-ssm', 'auto' or an AMI ID (advanced use)
      --node-ami-family string         'AmazonLinux2' for the Amazon EKS optimized AMI, or use 'Ubuntu2004' or 'Ubuntu1804' for the official Canonical EKS AMIs (default "AmazonLinux2")
  -P, --node-private-networking        whether to make nodegroup networking private
      --node-security-groups strings   attach additional security groups to nodes
      --node-labels stringToString     extra labels to add when registering the nodes in the nodegroup. List of comma separated KV pairs "k1=v1,k2=v2" (default [])
      --node-zones strings             (inherited from the cluster if unspecified)
      --instance-prefix string         add a prefix value in front of the instance's name
      --instance-name string           overrides the default instance's name
      --disable-pod-imds               Blocks IMDS requests from non-host networking pods
      --managed                        Create EKS-managed nodegroup (default true)
      --spot                           Create a spot nodegroup (managed nodegroups only)
      --instance-types strings         Comma-separated list of instance types (e.g., --instance-types=c3.large,c4.large,c5.large

Cluster and nodegroup add-ons flags:
      --asg-access               enable IAM policy for cluster-autoscaler
      --external-dns-access      enable IAM policy for external-dns
      --full-ecr-access          enable full access to ECR
      --appmesh-access           enable full access to AppMesh
      --appmesh-preview-access   enable full access to AppMesh Preview
      --alb-ingress-access       enable full access for alb-ingress-controller
      --install-neuron-plugin    install Neuron plugin for Inferentia and Trainium nodes (default true)
      --install-nvidia-plugin    install Nvidia plugin for GPU nodes (default true)

VPC networking flags:
      --vpc-cidr ipNet                 global CIDR to use for VPC (default 192.168.0.0/16)
      --vpc-private-subnets strings    re-use private subnets of an existing VPC; the subnets must exist in availability zones and not other types of zones
      --vpc-public-subnets strings     re-use public subnets of an existing VPC; the subnets must exist in availability zones and not other types of zones
      --vpc-from-kops-cluster string   re-use VPC from a given kops cluster
      --vpc-nat-mode string            VPC NAT mode, valid options: HighlyAvailable, Single, Disable (default "Single")

Instance Selector options flags:
      --instance-selector-vcpus int                 an integer value (2, 4 etc)
      --instance-selector-memory string             4 or 4GiB
      --instance-selector-cpu-architecture string   x86_64, or arm64
      --instance-selector-gpus int                  an integer value

AWS client flags:
  -p, --profile string         AWS credentials profile to use (defaults to the value of the AWS_PROFILE environment variable)
      --cfn-role-arn string    IAM role used by CloudFormation to call AWS API on your behalf
      --cfn-disable-rollback   for debugging: If a stack fails, do not roll it back. Be careful, this may lead to unintentional resource consumption!

Output kubeconfig flags:
      --kubeconfig string               path to write kubeconfig (incompatible with --auto-kubeconfig) (default "/Users/ireshniov/.kube/config")
      --authenticator-role-arn string   AWS IAM role to assume for authenticator
      --set-kubeconfig-context          if true then current-context will be set in kubeconfig; if a context is already set then it will be overwritten (default true)
      --auto-kubeconfig                 save kubeconfig file by cluster name, e.g. "/Users/ireshniov/.kube/eksctl/clusters/fabulous-party-1709474431"
      --write-kubeconfig                toggle writing of kubeconfig (default true)

Common flags:
  -C, --color string   toggle colorized logs (valid options: true, false, fabulous) (default "true")
  -d, --dumpLogs       dump logs to disk on failure if set to true
  -h, --help           help for this command
  -v, --verbose int    set log level, use 0 to silence, 4 for debugging and 5 for debugging with AWS debug logging (default 3)

Use 'eksctl create cluster [command] --help' for more information about a command.


For detailed docs go to https://eksctl.io/
```
And conform [docs](https://eksctl.io/usage/creating-and-managing-clusters/) create EKS cluster:
```shell
(base) ➜  Exercises eksctl create cluster \
--name exercise-11-cluster \
--version 1.28 \
--region eu-central-1 \
--nodegroup-name exercise-11-nodes \
--node-type t2.medium \
--nodes 3 \
2024-03-03 14:16:47 [ℹ]  eksctl version 0.168.0
2024-03-03 14:16:47 [ℹ]  using region eu-central-1
2024-03-03 14:16:47 [ℹ]  setting availability zones to [eu-central-1a eu-central-1c eu-central-1b]
2024-03-03 14:16:47 [ℹ]  subnets for eu-central-1a - public:192.168.0.0/19 private:192.168.96.0/19
2024-03-03 14:16:47 [ℹ]  subnets for eu-central-1c - public:192.168.32.0/19 private:192.168.128.0/19
2024-03-03 14:16:47 [ℹ]  subnets for eu-central-1b - public:192.168.64.0/19 private:192.168.160.0/19
2024-03-03 14:16:47 [ℹ]  nodegroup "exercise-11-nodes" will use "" [AmazonLinux2/1.28]
2024-03-03 14:16:47 [ℹ]  using Kubernetes version 1.28
2024-03-03 14:16:47 [ℹ]  creating EKS cluster "exercise-11-cluster" in "eu-central-1" region with managed nodes
2024-03-03 14:16:47 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2024-03-03 14:16:47 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=eu-central-1 --cluster=exercise-11-cluster'
2024-03-03 14:16:47 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "exercise-11-cluster" in "eu-central-1"
2024-03-03 14:16:47 [ℹ]  CloudWatch logging will not be enabled for cluster "exercise-11-cluster" in "eu-central-1"
2024-03-03 14:16:47 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=eu-central-1 --cluster=exercise-11-cluster'
2024-03-03 14:16:47 [ℹ]  
2 sequential tasks: { create cluster control plane "exercise-11-cluster", 
    2 sequential sub-tasks: { 
        wait for control plane to become ready,
        create managed nodegroup "exercise-11-nodes",
    } 
}
2024-03-03 14:16:47 [ℹ]  building cluster stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:16:48 [ℹ]  deploying stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:17:18 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:17:48 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:18:48 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:19:48 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:20:48 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:21:49 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:22:49 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:23:49 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:24:49 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:25:49 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-cluster"
2024-03-03 14:27:51 [ℹ]  building managed nodegroup stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:27:52 [ℹ]  deploying stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:27:52 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:28:22 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:29:03 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:29:44 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:30:26 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-nodegroup-exercise-11-nodes"
2024-03-03 14:30:26 [ℹ]  waiting for the control plane to become ready
2024-03-03 14:30:26 [✔]  saved kubeconfig as "/Users/ireshniov/.kube/config"
2024-03-03 14:30:26 [ℹ]  no tasks
2024-03-03 14:30:26 [✔]  all EKS cluster resources for "exercise-11-cluster" have been created
2024-03-03 14:30:27 [ℹ]  nodegroup "exercise-11-nodes" has 3 node(s)
2024-03-03 14:30:27 [ℹ]  node "ip-192-168-29-27.eu-central-1.compute.internal" is ready
2024-03-03 14:30:27 [ℹ]  node "ip-192-168-42-10.eu-central-1.compute.internal" is ready
2024-03-03 14:30:27 [ℹ]  node "ip-192-168-93-19.eu-central-1.compute.internal" is ready
2024-03-03 14:30:27 [ℹ]  waiting for at least 2 node(s) to become ready in "exercise-11-nodes"
2024-03-03 14:30:27 [ℹ]  nodegroup "exercise-11-nodes" has 3 node(s)
2024-03-03 14:30:27 [ℹ]  node "ip-192-168-29-27.eu-central-1.compute.internal" is ready
2024-03-03 14:30:27 [ℹ]  node "ip-192-168-42-10.eu-central-1.compute.internal" is ready
2024-03-03 14:30:27 [ℹ]  node "ip-192-168-93-19.eu-central-1.compute.internal" is ready
2024-03-03 14:30:27 [ℹ]  kubectl command should work with "/Users/ireshniov/.kube/config", try 'kubectl get nodes'
2024-03-03 14:30:27 [✔]  EKS cluster "exercise-11-cluster" in "eu-central-1" region is ready
```
Create fargate profile conform [docs](https://eksctl.io/usage/fargate-support/):
```shell
(base) ➜  Exercises eksctl create fargateprofile --namespace dev --cluster exercise-11-cluster --name exercise-11-fp 
2024-03-03 14:48:54 [ℹ]  deploying stack "eksctl-exercise-11-cluster-fargate"
2024-03-03 14:48:54 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-fargate"
2024-03-03 14:49:24 [ℹ]  waiting for CloudFormation stack "eksctl-exercise-11-cluster-fargate"
2024-03-03 14:49:25 [ℹ]  creating Fargate profile "exercise-11-fp" on EKS cluster "exercise-11-cluster"
2024-03-03 14:51:35 [ℹ]  created Fargate profile "exercise-11-fp" on EKS cluster "exercise-11-cluster"
```

Check nodes:
```shell
(base) ➜  Exercises kubectl get nodes                                       
NAME                                             STATUS   ROLES    AGE   VERSION
ip-192-168-29-27.eu-central-1.compute.internal   Ready    <none>   48m   v1.28.5-eks-5e0fdde
ip-192-168-42-10.eu-central-1.compute.internal   Ready    <none>   48m   v1.28.5-eks-5e0fdde
ip-192-168-93-19.eu-central-1.compute.internal   Ready    <none>   48m   v1.28.5-eks-5e0fdde

(base) ➜  Exercises eksctl get fargateprofile --cluster exercise-11-cluster 
NAME            SELECTOR_NAMESPACE      SELECTOR_LABELS POD_EXECUTION_ROLE_ARN                                                                          SUBNETS                                                        TAGS     STATUS
exercise-11-fp  dev                     <none>          arn:aws:iam::173861077519:role/eksctl-exercise-11-cluster--FargatePodExecutionRole-f3VJxARt1qOy subnet-0dcd21c5a8b241157,subnet-047fc146d6a6dbea3,subnet-0cc05e945afadf8bc      <none>  ACTIVE
```

</details>

******

<details>
<summary>Exercise 2: Deploy Mysql and phpmyadmin </summary>
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

#### Create `mysql-secret` and `mysql-config` components
`mysql-config.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  DB_SERVER: "mysql-primary.default.svc.cluster.local"
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

#### Based on [MySQL chart configuration](https://github.com/bitnami/charts/tree/main/bitnami/mysql) override custom values defined in chart configuration.
`mysql-chart-values-eks.yaml`
```yaml
architecture: replication
auth:
  rootPassword: secret
  database: team-app
  username: dev
  password: secret

# enable init container that changes the owner and group of the persistent volume mountpoint to runAsUser:fsGroup
volumePermissions:
  enabled: true

primary:
  persistence:
    enabled: false

secondary:
  # 1 primary and 2 secondary replicas
  replicaCount: 2
  persistence:
    enabled: false

    # Storage class for EKS volumes
    # storageClass: gp2
    # accessModes: ["ReadWriteOnce"]
```

#### Install mysql chart
```shell
(base) ➜  Exercises helm install mysql --values ./k8s-deployment/mysql-chart-values-eks.yaml bitnami/mysql
NAME: mysql
LAST DEPLOYED: Sun Mar  3 16:10:13 2024
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

#### Check mysql helm chart installation
```shell
(base) ➜  Exercises helm ls
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mysql   default         1               2024-03-03 16:10:13.976647 +0100 CET    deployed        mysql-9.15.0    8.0.35     
(base) ➜  Exercises kubectl get pods
NAME                READY   STATUS    RESTARTS      AGE
mysql-primary-0     0/1     Running   1 (87s ago)   104s
mysql-secondary-0   0/1     Running   1 (95s ago)   3m10s
mysql-secondary-1   0/1     Running   0             119s
```

#### Apply configmaps and secret components
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
  creationTimestamp: "2024-03-03T15:20:51Z"
  name: mysql-config
  namespace: default
  resourceVersion: "21752"
  uid: a8ae46d6-1e75-4fa5-968f-9b1d01794537

(base) ➜  Exercises kubectl get secrets mysql-secret -o yaml
apiVersion: v1
data:
  DB_NAME: dGVhbS1hcHA=
  DB_PWD: c2VjcmV0
  DB_ROOT_PWD: dG9vcg==
  DB_USER: ZGV2
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"DB_NAME":"dGVhbS1hcHA=","DB_PWD":"c2VjcmV0","DB_ROOT_PWD":"dG9vcg==","DB_USER":"ZGV2"},"kind":"Secret","metadata":{"annotations":{},"name":"mysql-secret","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2024-03-03T15:20:51Z"
  name: mysql-secret
  namespace: default
  resourceVersion: "21754"
  uid: 025cec6d-c175-4514-bb20-ed9088e93bc3
type: Opaque


(base) ➜  Exercises kubectl get secrets mysql-secret -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
DB_NAME: team-app
DB_PWD: secret
DB_ROOT_PWD: toor
DB_USER: dev
```

#### Create deployment and service components for phpmyadmin
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

#### Apply deployment and service components in kubernetes cluster
```shell
(base) ➜  Exercises kubectl apply -f ./k8s-deployment/phpmyadmin.yaml  
deployment.apps/phpmyadmin-deployment created
service/phpmyadmin-service created

(base) ➜  Exercises kubectl get pods    
NAME                                    READY   STATUS             RESTARTS        AGE
mysql-primary-0                         0/1     CrashLoopBackOff   6 (2m34s ago)   15m
mysql-secondary-0                       0/1     Running            6 (2m50s ago)   16m
mysql-secondary-1                       0/1     Running            5 (101s ago)    15m
phpmyadmin-deployment-77b777d54-q2zjl   1/1     Running            0               22s

(base) ➜  Exercises kubectl get services
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                 ClusterIP   10.100.0.1       <none>        443/TCP    124m
mysql-primary              ClusterIP   10.100.228.105   <none>        3306/TCP   17m
mysql-primary-headless     ClusterIP   None             <none>        3306/TCP   17m
mysql-secondary            ClusterIP   10.100.240.121   <none>        3306/TCP   17m
mysql-secondary-headless   ClusterIP   None             <none>        3306/TCP   17m
phpmyadmin-service         ClusterIP   10.100.240.199   <none>        8081/TCP   43s
```

```shell
(base) ➜  Exercises kubectl port-forward svc/phpmyadmin-service 8081:8081
Forwarding from 127.0.0.1:8081 -> 80
Forwarding from [::1]:8081 -> 80
Handling connection for 8081
```
Access phpmyadmin at ``127.0.0.1:8081``
Login using credentials: `dev/secret`
</details>

******

<details>
<summary>Exercise 3: Deploy your Java application </summary>
 <br />

#### Create namespace dev
```shell
(base) ➜  Exercises kubectl create namespace dev
namespace/dev created
```

#### Apply secrets/configs in dev namespace
```shell
(base) ➜  Exercises kubectl -n dev apply -f ./k8s-deployment/mysql-config.yaml -f ./k8s-deployment/mysql-secret.yaml
configmap/mysql-config created
secret/mysql-secret created
```

#### Create Docker registry secret
```shell
(base) ➜  Exercises kubectl create secret docker-registry my-registry-key \
--docker-server=docker.io \
--docker-username=ireshniov1 \
--docker-password=[password] \
--namespace dev
secret/my-registry-key created
```

#### Check secrets and configmaps on dev namespace
```shell
(base) ➜  Exercises kubectl -n dev get secrets
NAME              TYPE                             DATA   AGE
my-registry-key   kubernetes.io/dockerconfigjson   1      6m6s
mysql-secret      Opaque                           4      5m38s

(base) ➜  Exercises kubectl -n dev get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      6m21s
mysql-config       1      5m47s
```

#### Create deployment && service components for java app
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-excercises
spec:
  replicas: 2
  selector:
    matchLabels:
      app: eks-excercises
  template:
    metadata:
      labels:
        app: eks-excercises
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: eks-excercises
        image: ireshniov1/eks-excercises:1.0
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
  name: eks-excercises
spec:
  type: ClusterIP
  selector:
    app: eks-excercises
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```

#### Prepare image with eks-exercises
`Dockerfile`
```dockerfile
FROM gradle:7.6.0-jdk17-alpine as builder

WORKDIR /eks-exercises-app

COPY --chown=gradle:gradle build.gradle .
COPY --chown=gradle:gradle settings.gradle .
COPY --chown=gradle:gradle src src

RUN gradle build

FROM openjdk:17-jdk-alpine as release

WORKDIR /eks-exercises-app

RUN addgroup --gid 1001 --system eks-exercises && \
    adduser -S --uid 1001 -G eks-exercises eks-exercises && \
    chown -R eks-exercises:eks-exercises /eks-exercises-app

RUN mkdir -p /var/logs/eks-exercises && \
    chown -R eks-exercises:eks-exercises /var/logs/eks-exercises

COPY --chown=eks-exercises:eks-exercises --from=builder /eks-exercises-app/build/libs/eks-exercises-project.jar .

USER 1001:1001

CMD ["java", "-jar", "eks-exercises-project.jar"]
```

#### Push image with eks-exercises in docker repository
```shell
(base) ➜  kubernetes-exercises git:(master) ✗ docker login -u ireshniov1 --password [password]
Login Succeeded

(base) ➜  kubernetes-exercises git:(master) ✗ docker build -t ireshniov1/eks-exercises:1.0 .
[+] Building 68.3s (17/17) FINISHED                                                                                                                                                                docker:desktop-linux
 => [internal] load .dockerignore                                                                                                                                                                                  0.1s
 => => transferring context: 2B                                                                                                                                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                                                                                                                               0.1s
 => => transferring dockerfile: 782B                                                                                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/openjdk:17-jdk-alpine                                                                                                                                           2.1s
 => [internal] load metadata for docker.io/library/gradle:7.6.0-jdk17-alpine                                                                                                                                       2.0s
 => CACHED [builder 1/6] FROM docker.io/library/gradle:7.6.0-jdk17-alpine@sha256:74f07921e21e5ef6935b60ed014811e34b552e1d9f020a7735df6ffb7bbf7860                                                                  0.0s
 => [internal] load build context                                                                                                                                                                                  0.0s
 => => transferring context: 13.12kB                                                                                                                                                                               0.0s
 => CACHED [release 1/5] FROM docker.io/library/openjdk:17-jdk-alpine@sha256:4b6abae565492dbe9e7a894137c966a7485154238902f2f25e9dbd9784383d81                                                                      0.0s
 => [builder 2/6] WORKDIR /eks-exercises-app                                                                                                                                                                       0.1s
 => [release 2/5] WORKDIR /eks-exercises-app                                                                                                                                                                       0.1s
 => [release 3/5] RUN addgroup --gid 1001 --system eks-exercises &&     adduser -S --uid 1001 -G eks-exercises eks-exercises &&     chown -R eks-exercises:eks-exercises /eks-exercises-app                        1.3s
 => [builder 3/6] COPY --chown=gradle:gradle build.gradle .                                                                                                                                                        0.3s
 => [builder 4/6] COPY --chown=gradle:gradle settings.gradle .                                                                                                                                                     0.1s
 => [builder 5/6] COPY --chown=gradle:gradle src src                                                                                                                                                               0.1s
 => [builder 6/6] RUN gradle build                                                                                                                                                                                65.0s
 => [release 4/5] RUN mkdir -p /var/logs/eks-exercises &&     chown -R eks-exercises:eks-exercises /var/logs/eks-exercises                                                                                         0.4s
 => [release 5/5] COPY --chown=eks-exercises:eks-exercises --from=builder /eks-exercises-app/build/libs/eks-exercises-project.jar .                                                                                0.1s
 => exporting to image                                                                                                                                                                                             0.2s
 => => exporting layers                                                                                                                                                                                            0.2s
 => => writing image sha256:80ed5e1507dbb2d3c9e3053ecf28c94aaaf2b4b8f12cb79563c4a15d90087f16                                                                                                                       0.0s
 => => naming to docker.io/library/eks-exercises:1.0  

(base) ➜  kubernetes-exercises git:(master) ✗ docker push ireshniov1/eks-exercises:1.0                 
The push refers to repository [docker.io/ireshniov1/eks-exercises]
f5158acce980: Pushed 
cbf26598f559: Pushed 
82c4b7f48330: Pushed 
d34b93192970: Pushed 
34f7184834b2: Mounted from library/openjdk 
5836ece05bfd: Mounted from library/openjdk 
72e830a4dff5: Mounted from library/openjdk 
1.0: digest: sha256:e2e8d88239c8eb91f90d87ada5ea5858ed08f6102669ab0bbf71919de23fa84e size: 1785
```

#### Apply java app deployment/service
```shell
(base) ➜  Exercises kubectl --namespace dev apply -f ./k8s-deployment/eks-exercises.yaml                                     
deployment.apps/eks-excercises created
service/eks-excercises created

(base) ➜  Exercises kubectl --namespace dev get pods                                           
NAME                              READY   STATUS    RESTARTS   AGE
eks-excercises-59f7bd44bd-fdpvv   1/1     Running   0          6m40s
eks-excercises-59f7bd44bd-w4sbw   1/1     Running   0          6m40s
```
</details>

******

<details>
<summary>Exercise 4: Automate deployment </summary>
 <br />

#### Finish Jenkins configuration
1) Create multibranch pipeline `eks-excercises`.
2) Add `gitlab-credentials` credentials for `eks-excercises` java project on gitlab (create corresponding repository before)
3) Add `docker-hub-repo` credentials in Jenkins.
4) From `cat ~/.aws/credentials`:
   Create `jenkins_aws_access_key_id` credentials
   Create `jenkins_aws_secret_access_key` credentials

5) Configure branch source `https://gitlab.com/ireshniov/eks-excercises.git`

#### Parametrize deployment components and create `./k8s/eks-exercises.yaml` in java app project.
`./k8s/eks-exercises.yaml`
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
   name: $APP_NAME
   namespace: dev
spec:
   replicas: 2
   selector:
      matchLabels:
         app: $APP_NAME
   template:
      metadata:
         labels:
            app: $APP_NAME
      spec:
         imagePullSecrets:
            - name: my-registry-key
         containers:
            - name: $APP_NAME
              image: ireshniov1/eks-exercises:$IMAGE_NAME
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
   name: $APP_NAME
spec:
   type: ClusterIP
   selector:
      app: $APP_NAME
   ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
```

#### Set git user.name, user.email
```text
Dashboard -> Manage Jenkins -> System -> Git Plugin
    `user.name`: ireshniov
    `user.email`: ireshniov@gmail.com
```

#### Adjust Jenkinsfile
`Jenkinsfile`
```text
#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        IMAGE_NAME = "1.0-${BUILD_NUMBER}"
    }

    stages {
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."

                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t ireshniov1/eks-exercises:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push ireshniov1/eks-exercises:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('deploy') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
                APP_NAME = 'eks-exercises'
            }

            steps {
                script {
                   echo 'deploying docker image...'
                   sh 'envsubst < ./k8s/eks-exercises.yaml | kubectl apply -f -'
                }
            }
        }
    }
}
```
</details>

******

<details>
<summary>Exercise 5: Use ECR as Docker repository </summary>
 <br />

#### Create ECR repository
`173861077519.dkr.ecr.eu-central-1.amazonaws.com/eks-exercises`

#### Create Credentials in Jenkins
Password:
```shell
(base) ➜  Exercises aws ecr get-login-password --region eu-central-1 --profile [profile]
[hash]
```
```text
ID: ecr-credentials
Username: AWS
Password: [hash]
```

#### Create secret for AWS ECR - ecr-registry-key
```shell
(base) ➜  Exercises kubectl --namespace dev create secret docker-registry ecr-registry-key \
--docker-server=173861077519.dkr.ecr.eu-central-1.amazonaws.com/eks-exercises \
--docker-username=AWS \
--docker-password=[hash]

secret/ecr-registry-key created

(base) ➜  Exercises kubectl --namespace dev get secrets
NAME               TYPE                             DATA   AGE
ecr-registry-key   kubernetes.io/dockerconfigjson   1      58s
my-registry-key    kubernetes.io/dockerconfigjson   1      8h
mysql-secret       Opaque                           4      8h
```

#### Adjust building and tagging
`./k8s/eks-exercises.yaml`
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
   name: $APP_NAME
   namespace: dev
spec:
   replicas: 2
   selector:
      matchLabels:
         app: $APP_NAME
   template:
      metadata:
         labels:
            app: $APP_NAME
      spec:
         imagePullSecrets:
            - name: ecr-registry-key
         containers:
            - name: $APP_NAME
              image: $DOCKER_REPO:$IMAGE_NAME
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
   name: $APP_NAME
spec:
   type: ClusterIP
   selector:
      app: $APP_NAME
   ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
```

#### Update Jenkinsfile
`Jenkinsfile`
```text
#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        IMAGE_NAME = "1.0-${BUILD_NUMBER}"
        DOCKER_REPO_SERVER = '173861077519.dkr.ecr.eu-central-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/eks-exercises"
    }

    stages {
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."

                    withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('deploy') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
                APP_NAME = 'eks-exercises'
            }

            steps {
                script {
                   echo 'deploying docker image...'
                   sh 'envsubst < ./k8s/eks-exercises.yaml | kubectl apply -f -'
                }
            }
        }
    }
}
```

</details>

******

<details>
<summary>Exercise 6: Configure Autoscaling </summary>
 <br />

#### Deploy K8s Autoscaler component
From [this](https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml) is Autoscaler K8s Component configuration

1) Add annotation `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"` to deployment
2) Replace <YOUR_CLUSTE_NAME> with `exercise-11-cluster`
3) Add options to the `command`
```shell
- --balance-similar-node-groups
- --skip-nodes-with-system-pods=false
```
4) Change image name to an actual kubernetes version used in cluster
```text
Kubernetes version
1.28
```
Kubernetes autoscaler version more close to kubernetes version in cluster could be found [Here](https://github.com/kubernetes/autoscaler/tags)

`cluster-autoscaler-autodiscover.yaml`
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      containers:
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.28.2
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/exercise-11-cluster
            - --balance-similar-node-groups
            - --skip-nodes-with-system-pods=false
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt # /etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```

```shell
(base) ➜  Exercises kubectl apply -f ./k8s-deployment/cluster-autoscaler-autodiscover.yaml 
serviceaccount/cluster-autoscaler unchanged
clusterrole.rbac.authorization.k8s.io/cluster-autoscaler unchanged
role.rbac.authorization.k8s.io/cluster-autoscaler unchanged
clusterrolebinding.rbac.authorization.k8s.io/cluster-autoscaler unchanged
rolebinding.rbac.authorization.k8s.io/cluster-autoscaler unchanged
deployment.apps/cluster-autoscaler configured


(base) ➜  Exercises kubectl get deployment -n kube-system cluster-autoscaler                                                                                                          
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
cluster-autoscaler   1/1     1            1           41m
```

#### Reduce replicas
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $APP_NAME
spec:
  replicas: 1
```

#### Check autoscaler logs and find signs that it scales down
```shell
(base) ➜  Exercises kubectl -n kube-system get pods
NAME                                  READY   STATUS    RESTARTS   AGE
aws-node-br8cd                        2/2     Running   0          13h
aws-node-pjxs6                        2/2     Running   0          13h
aws-node-whk44                        2/2     Running   0          13h
cluster-autoscaler-6fb984954f-vdkq9   1/1     Running   0          14m
coredns-648485486-l66z7               1/1     Running   0          13h
coredns-648485486-smm9v               1/1     Running   0          13h
kube-proxy-gbzmq                      1/1     Running   0          13h
kube-proxy-kd25h                      1/1     Running   0          13h
kube-proxy-v9bw6                      1/1     Running   0          13h
(base) ➜  Exercises kubectl -n kube-system logs cluster-autoscaler-6fb984954f-vdkq9 > autoscaler-logs.txt

I0308 14:53:32.468568       1 static_autoscaler.go:642] Starting scale down
I0308 14:53:34.569816       1 reflector.go:790] k8s.io/client-go/informers/factory.go:150: Watch close - *v1.Pod total 24 items received
I0308 14:53:37.764941       1 reflector.go:790] k8s.io/autoscaler/cluster-autoscaler/utils/kubernetes/listers.go:359: Watch close - *v1.StatefulSet total 9 items received

(base) ➜  Exercises kubectl get nodes                                                            
NAME                                                       STATUS   ROLES    AGE    VERSION
fargate-ip-192-168-104-252.eu-central-1.compute.internal   Ready    <none>   7m8s   v1.28.5-eks-680e576
ip-192-168-26-204.eu-central-1.compute.internal            Ready    <none>   13h    v1.28.5-eks-5e0fdde
ip-192-168-60-18.eu-central-1.compute.internal             Ready    <none>   13h    v1.28.5-eks-5e0fdde
ip-192-168-65-44.eu-central-1.compute.internal             Ready    <none>   13h    v1.28.5-eks-5e0fdde
```

</details>

******
