******

<details>
<summary>Exercise 0: Clone Git repository and create your own </summary>
 <br />

```shell
git clone git@gitlab.com:devops-bootcamp3/node-project.git
cd node-project
git remote remove origin
git remote add origin git@gitlab.com:ireshniov/bootcamp-node-project.git
git branch -M main
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 1: Dockerize your NodeJS App </summary>
 <br />

```dockerfile
ARG NODE_VERSION=18.14.2
ARG DEBIAN_VERSION=buster

FROM --platform=linux/amd64 node:${NODE_VERSION}-${DEBIAN_VERSION}-slim as builder

WORKDIR /bootcamp-node-project-app

COPY ./app/package.json .
COPY ./app/package-lock.json .

RUN npm ci --ignore-scripts

COPY ./app/images images
COPY ./app/index.html .
COPY ./app/server.js .

FROM --platform=linux/amd64 node:${NODE_VERSION}-${DEBIAN_VERSION}-slim as release

WORKDIR /bootcamp-node-project-app

# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

RUN addgroup --gid 1001 --system bootcamp-node-project && \
    adduser --system --uid 1001 --gid 1001 bootcamp-node-project && \
    chown -R bootcamp-node-project:bootcamp-node-project /bootcamp-node-project-app && \
    chown bootcamp-node-project:bootcamp-node-project /tini

RUN mkdir -p /var/logs/bootcamp-node-project && \
    chown -R bootcamp-node-project:bootcamp-node-project /var/logs/bootcamp-node-project

COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/package.json package.json
COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/node_modules node_modules
COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/images images
COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/index.html index.html
COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/server.js server.js

USER 1001:1001

CMD ["node", "server.js"]
```

```shell
(base) ➜  node-project git:(main) ✗ docker build -t bootcamp-node-project:1.1 .
[+] Building 45.2s (23/23) FINISHED                                                                                                                                                                             
 => [internal] load build definition from Dockerfile                                                                                                                                                       0.0s
 => => transferring dockerfile: 37B                                                                                                                                                                        0.0s
 => [internal] load .dockerignore                                                                                                                                                                          0.0s
 => => transferring context: 2B                                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/node:16-buster-slim                                                                                                                                     0.7s
 => CACHED https://github.com/krallin/tini/releases/download/v0.19.0/tini                                                                                                                                  0.0s
 => [internal] load build context                                                                                                                                                                          0.0s
 => => transferring context: 298B                                                                                                                                                                          0.0s
 => [release  1/11] FROM docker.io/library/node:16-buster-slim@sha256:94c1f79b70d120cf40858f5cb57929aad20922224bd06b94a04e8501da56bd96                                                                    13.3s
 => => resolve docker.io/library/node:16-buster-slim@sha256:94c1f79b70d120cf40858f5cb57929aad20922224bd06b94a04e8501da56bd96                                                                               0.0s
 => => sha256:6adc86b2ad66d4247e945336fb594b09b7ed7b95418281f41608a57b662d5a72 4.18kB / 4.18kB                                                                                                             0.5s
 => => sha256:cf282fca3438846ca1727173ae416c1084802955367c6d2e3bf32e5414d477b5 35.38MB / 35.38MB                                                                                                          12.7s
 => => sha256:94c1f79b70d120cf40858f5cb57929aad20922224bd06b94a04e8501da56bd96 776B / 776B                                                                                                                 0.0s
 => => sha256:815309603d8729c22fdea62b0bd641ec929c782919817ec2bf9274c68515c4b2 1.37kB / 1.37kB                                                                                                             0.0s
 => => sha256:fab368839545df1e39cff4b9336c2384a7c80fbcdda985f2c7c966a931507eae 6.80kB / 6.80kB                                                                                                             0.0s
 => => sha256:29cd48154c03e9242f1ff4f9895cf886a344fb94c9b71029455e76e11214328f 27.14MB / 27.14MB                                                                                                           9.1s
 => => sha256:c4f43314cde1919f9ae8333c4ce2f64ed0fbe5304d8fc61e2432abbac6615385 2.73MB / 2.73MB                                                                                                             2.5s
 => => sha256:8934e5796b5b1cef45491d068e8454088de472b30758aaaefe2684e360b570b2 453B / 453B                                                                                                                 2.7s
 => => extracting sha256:29cd48154c03e9242f1ff4f9895cf886a344fb94c9b71029455e76e11214328f                                                                                                                  0.4s
 => => extracting sha256:6adc86b2ad66d4247e945336fb594b09b7ed7b95418281f41608a57b662d5a72                                                                                                                  0.0s
 => => extracting sha256:cf282fca3438846ca1727173ae416c1084802955367c6d2e3bf32e5414d477b5                                                                                                                  0.5s
 => => extracting sha256:c4f43314cde1919f9ae8333c4ce2f64ed0fbe5304d8fc61e2432abbac6615385                                                                                                                  0.1s
 => => extracting sha256:8934e5796b5b1cef45491d068e8454088de472b30758aaaefe2684e360b570b2                                                                                                                  0.0s
 => [release  2/11] WORKDIR /bootcamp-node-project-app                                                                                                                                                     0.1s
 => [builder 3/8] COPY ./app/package.json .                                                                                                                                                                0.0s
 => [release  3/11] ADD https://github.com/krallin/tini/releases/download/v0.19.0/tini /tini                                                                                                               0.0s
 => [builder 4/8] COPY ./app/package-lock.json .                                                                                                                                                           0.0s
 => [release  4/11] RUN chmod +x /tini                                                                                                                                                                     0.2s
 => [builder 5/8] RUN npm ci --ignore-scripts                                                                                                                                                             29.6s
 => [release  5/11] RUN addgroup --gid 1001 --system bootcamp-node-project &&     adduser --system --uid 1001 --gid 1001 bootcamp-node-project &&     chown -R bootcamp-node-project:bootcamp-node-projec  0.9s
 => [release  6/11] RUN mkdir -p /var/logs/bootcamp-node-project &&     chown -R bootcamp-node-project:bootcamp-node-project /var/logs/bootcamp-node-project                                               0.2s
 => [builder 6/8] COPY ./app/images images                                                                                                                                                                 0.0s
 => [builder 7/8] COPY ./app/index.html .                                                                                                                                                                  0.0s
 => [builder 8/8] COPY ./app/server.js .                                                                                                                                                                   0.0s
 => [release  7/11] COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/package.json package.json                                                           0.0s
 => [release  8/11] COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/node_modules node_modules                                                           0.7s
 => [release  9/11] COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/images images                                                                       0.0s
 => [release 10/11] COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/index.html index.html                                                               0.0s
 => [release 11/11] COPY --chown=bootcamp-node-project:bootcamp-node-project --from=builder /bootcamp-node-project-app/server.js server.js                                                                 0.0s
 => exporting to image                                                                                                                                                                                     0.3s
 => => exporting layers                                                                                                                                                                                    0.3s
 => => writing image sha256:60f0460634c1dcd854aeddc294518b09a953b4962b5ea1ab72cb7d207b9e5f7b                                                                                                               0.0s
 => => naming to docker.io/library/bootcamp-node-project:1.1
 
(base) ➜  node-project git:(main) ✗ docker run --name bootcamp-node-project -p 3000:3000 -d --restart=always --platform=linux/amd64 \
bootcamp-node-project:1.1
8cce4207bddf13bea61c13952039753586c2d18a923ab9c9760822729465f8c6
```

Accessed main page on `http://localhost:3000/`

</details>

******

<details>
<summary>Exercise 2: Create a full pipeline for your NodeJS App </summary>
 <br />

1) Start and configure Jenkins server

* Extend jenkins by installing docker
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

* Build and run jenkins with docker image.
```shell
docker build -t jenkins_with_docker .

docker run --name jenkins_with_docker -p 8080:8080 -p 50000:50000 -d --restart=on-failure \
-v jenkins_home:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkins_with_docker
```

* Fix permissions issue on use docker in jenkins.
```shell
docker exec -u 0 -it jenkins_with_docker chmod 666 /var/run/docker.sock
```

* Access Jenkins UI on `http://127.0.0.1:8080/`. Fill admin password
```shell
docker exec -u 0 -it jenkins_with_docker cat /var/jenkins_home/secrets/initialAdminPassword
cb2bff2ea6e94791a9bbac760aba10ec
```

* Install suggested plugins. Create first admin user.

2) Docker containers as build agents. Basing on [Using agents tutorial](https://www.jenkins.io/doc/book/using/using-agents) and [this](https://www.youtube.com/watch?v=99DddJiH7lM)

* Prevent builds from running on the build-in node:
Manage Jenkins -> Manage Nodes and Clouds -> Built-In Node in the list -> Configure in the menu -> Set the number of executors to 0 -> Save

* Add ssh key on host using `ssh-keygen -f ~/.ssh/jenkins_agent_key`

* Extend jenkins agent image
```dockerfile
FROM jenkins/ssh-agent:alpine

RUN apk update
RUN apk add docker docker-cli-compose git curl

RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/v3.18/main/ nodejs=18.18.2-r0 npm

RUN git config --global user.email "jenkins@example.com"
RUN git config --global user.name "jenkins"
```

* Build and run jenkins with docker image.
```shell
docker build -t jenkins_agent --platform=linux/amd64 -f ./Dockerfile-agent .

docker run -d --rm --name=agent1 -p 22:22 --platform=linux/amd64 \
-e "JENKINS_AGENT_SSH_PUBKEY=$(cat ~/.ssh/jenkins_agent_key.pub)" \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkins_agent
```

* Fix permissions issue on use docker in jenkins agent
```shell
docker exec -u 0 -it agent1 chmod 666 /var/run/docker.sock
```

#### Set jenkins node

* Add `jenkins` credentials
```text
Manage Jenkins -> Manage Credentials -> Add Credentials. Create credentials with

Kind: SSH Username with private key
id: jenkins
username: jenkins
Private Key: Enter directly -> on host `cat ~/.ssh/jenkins_agent_key`
```

* Add new node
```text
Manage Jenkins -> Manage Nodes and clouds -> New Node with

Remote root directory: /home/jenkins
label: agent1
usage: only build jobs with label expression
Launch method: Launch agents by SSH
Host -> `docker container inspect agent1 | grep IPAddress`
Credentials: jenkins
Host Key verification Strategy: Manually trusted key verification
```

3) Configure Pipeline

* Create gitlab (`gitlab-credentials`) and docker (`docker-credentials`) credentials in Jenkins

* Manage Jenkins -> System Configuration ->  Git plugin. Set `Global Config user.name Value` = `jenkins` and `Global Config user.email Value` = `jenkins@example.com`

* Add cicd/Jenkinsfile
```text
#!/usr/bin/env groovy

def gv

pipeline {
    // https://www.youtube.com/watch?v=ymI02j-hqpU
    agent {
        label 'agent1'
    }

    environment {
        GIT_REPO = 'gitlab.com/ireshniov/bootcamp-node-project.git'
        GIT_BRANCH = 'main'
        DOCKER_REPO = 'ireshniov1/demo-app'
    }

    stages {
        stage('Init') {
            steps {
                script {
                    gv = load "cicd/cicd-script.groovy"
                }
            }
        }

        stage('Increment version') {
            steps {
                script {
                    def version = gv.incrementVersion()
                    env.IMAGE_NAME = "${env.DOCKER_REPO}:${version}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Run tests') {
            steps {
               script {
                    gv.runTests()
               }
            }
        }

        stage('Build and push docker image') {
            steps {
                script {
                    gv.buildAndPushDockerImage "${env.IMAGE_NAME}"
                }
            }
        }

        stage('Deploy app') {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }

        stage('Commit version update') {
            steps {
                script {
                    gv.commitVersionUpdate "${env.GIT_REPO}", "${env.GIT_BRANCH}"
                }
            }
        }
    }
}
```

* Add cicd/cicd-script.groovy:
```text
def incrementVersion() {
    echo 'Incrementing app version...'

    // enter app directory, because that's where package.json is located
    dir("app") {
        // # update application version in the package.json file with one of these release types: patch, minor or major
        // # this will commit the version update
        sh "npm version minor"

        // # Pre: install Pipeline Utility Steps
        // # read the updated version from the package.json file
        // def packageJson = readJSON file: 'package.json'
        // def version = packageJson.version

        // # set the new version as part of IMAGE_NAME
        // # alternative solution without Pipeline Utility Steps plugin:
        return sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr -d '\n'")
    }
}

def runTests() {
    echo "Running tests..."
    // enter app directory, because that's where package.json and tests are located
    dir("app") {
        // install all dependencies needed for running tests
        sh "npm install"
        sh "npm run test"
    }
}

def buildAndPushDockerImage(String imageName) {
    echo "Building the docker image..."
    sh "docker build -t $imageName ."

    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
        sh "echo $PWD | docker login -u $USER --password-stdin"
    }

    sh "docker push $imageName"
}

def deployApp() {
    echo "Deploying the application..."
}

def commitVersionUpdate(String gitRepository, String branchName) {
    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
        sh "git remote set-url origin https://$USER:$PWD@$gitRepository"
    }

    sh 'git add .'
    sh 'git commit -m "ci: version bump"'
    sh "git push origin HEAD:$branchName"
}

return this
```

* Create Pipeline with
```text
Name: my-pipeline
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://gitlab.com/ireshniov/bootcamp-node-project.git
Credentials: gitlab-credentials
Branch Specifier: */main
Script Path: cicd/Jenkinsfile
```

* Dashboard -> my-pipeline -> Build

### TODO on remote server:
#### Docker containers as build agents (with Docker Pipeline plugin) as explained [here](https://www.youtube.com/watch?v=ymI02j-hqpU)
#### As above but fix permission issue as in Dockerfile will be installed docker with socket mounted on host
#### Use simple Pipeline script (instead of Pipeline script from SCM) and add checkout step with manual implementation.
#### Triggers
</details>

******

<details>
<summary>Exercise 3: Manually deploy new Docker Image on server </summary>
 <br />

```shell
(base) ➜  node-project git:(main) docker run -p 3000:3000 -d --platform=linux/amd64 ireshniov1/demo-app:1.2.0-78
c6bc63d9c48036743734578c83bc4a9069bfa3c6101802fa3894e81289bc432d
```

* Access main page on `http://localhost:3000/`

</details>

******

<details>
<summary>Exercise 4: Extract into Jenkins Shared Library </summary>
 <br />

1) Create `bootcamp-jenkins-shared-library` repo with
* src/com/example/Docker.groovy
```text
#!/usr/bin/env groovy
package com.example

class Docker implements Serializable {
    Script script

    Docker(Script script) {
        this.script = script
    }

    void buildImage(String imageName) {
        script.sh "docker build -t $imageName ."
    }

    void login() {
        script.withCredentials([script.usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
            script.sh "echo $script.PWD | docker login -u $script.USER --password-stdin"
        }
    }

    void push(String imageName) {
        script.sh "docker push $imageName"
    }
}
```

* src/com/example/Git.groovy
```text
#!/usr/bin/env groovy
package com.example

class Git implements Serializable {
    Script script

    Git(Script script) {
        this.script = script
    }

    void commitVersionUpdate(String gitRepository, String branchName) {
        script.withCredentials([script.usernamePassword(credentialsId: 'gitlab-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
            script.sh "git remote set-url origin https://$script.USER:$script.PWD@$gitRepository"
        }

        script.sh 'git add .'
        script.sh 'git commit -m "ci: version bump"'
        script.sh "git push origin HEAD:$branchName"
    }
}
```

* src/com/example/Npm.groovy
```text
#!/usr/bin/env groovy
package com.example

class Npm implements Serializable {
    Script script

    Npm(Script script) {
        this.script = script
    }

    void runTests() {
        script.echo "Running tests..."
        // enter app directory, because that's where package.json and tests are located
        script.dir("app") {
            // install all dependencies needed for running tests
            script.sh "npm install"
            script.sh "npm run test"
        }
    }

    String incrementVersion() {
        script.echo 'Incrementing app version...'

        // enter app directory, because that's where package.json is located
        script.dir("app") {
            // # update application version in the package.json file with one of these release types: patch, minor or major
            // # this will commit the version update
            script.sh "npm version minor"

            // # Pre: install Pipeline Utility Steps
            // # read the updated version from the package.json file
            // def packageJson = readJSON file: 'package.json'
            // def version = packageJson.version

            // # set the new version as part of IMAGE_NAME
            // # alternative solution without Pipeline Utility Steps plugin:
            return script.sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr -d '\n'")
        }
    }
}
```

* vars/commitVersionUpdate.groovy
```text
#!/usr/bin/env groovy
import com.example.Git

def call(String gitRepository, String branchName) {
    return new Git(this).commitVersionUpdate(gitRepository, branchName)
}
```

* vars/dockerBuildImage.groovy
```text
#!/usr/bin/env groovy
import com.example.Docker

def call(String imageName) {
    return new Docker(this).buildImage(imageName)
}
```

* vars/dockerLogin.groovy
```text
#!/usr/bin/env groovy
import com.example.Docker

def call() {
    return new Docker(this).login()
}
```

* vars/dockerPush.groovy
```text
#!/usr/bin/env groovy
import com.example.Docker

def call(String imageName) {
    return new Docker(this).push(imageName)
}
```

* vars/incrementVersion.groovy
```text
#!/usr/bin/env groovy
import com.example.Npm

def call() {
    return new Npm(this).incrementVersion()
}
```

* vars/runTests.groovy
```text
#!/usr/bin/env groovy
import com.example.Npm

def call() {
    return new Npm(this).runTests()
}
```

2) Create remote repo `bootcamp-jenkins-shared-library` on gitlab and push local repo (https://gitlab.com/ireshniov/bootcamp-jenkins-shared-library)
```shell
git init --initial-branch=main
git remote add origin git@gitlab.com:ireshniov/bootcamp-jenkins-shared-library.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

3) Сonfigure shared library on jenkins controller.
* `http://127.0.0.1:8080/manage/configure` -> Global Pipeline Libraries
```text
name: bootcamp-jenkins-shared-library
Default version: main (as branch is this case is main). Could be tag, hash, commit and etc.
Retrieval method: Modern SCM
    Source Code Management: Git
    Project Repository: https://gitlab.com/ireshniov/bootcamp-jenkins-shared-library
    Credentials: gitlab-credentials
    Filter by name (with regular expression): */main
```

4) Apply library in Jenkinsfile
```text
#!/usr/bin/env groovy

@Library('bootcamp-jenkins-shared-library')_

// library identifier: 'bootcamp-jenkins-shared-library@main', retriever: modernSCM([
//     $class: 'GitSCMSource',
//     remote: 'https://gitlab.com/ireshniov/bootcamp-jenkins-shared-library.git',
//     credentialsId: 'gitlab-credentials'
// ])

def gv

pipeline {
    agent {
        label 'agent1'
    }

    environment {
        GIT_REPO = 'gitlab.com/ireshniov/bootcamp-node-project.git'
        GIT_BRANCH = 'main'
        DOCKER_REPO = 'ireshniov1/demo-app'
    }

    stages {
        stage('Init') {
            steps {
                script {
                    gv = load "cicd/cicd-script.groovy"
                }
            }
        }

        stage('Increment version') {
            steps {
                script {
                    def version = incrementVersion()
                    env.IMAGE_NAME = "${env.DOCKER_REPO}:${version}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Run tests') {
            steps {
               script {
                    runTests()
               }
            }
        }

        stage('Build and push docker image') {
            steps {
                script {
                    dockerBuildImage "${env.IMAGE_NAME}"
                    dockerLogin()
                    dockerPush "${env.IMAGE_NAME}"
                }
            }
        }

        stage('Deploy app') {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }

        stage('Commit version update') {
            steps {
                script {
                    commitVersionUpdate "${env.GIT_REPO}", "${env.GIT_BRANCH}"
                }
            }
        }
    }
}
```

* Add `cicd-script.groovy`
```text
def deployApp() {
    echo "Deploying the application..."
}

return this

```
5) Build job
6) Check build works
* Manually deploy to check it works:
```shell
(base) ➜  node-project git:(main) docker run -p 3000:3000 -d --platform=linux/amd64 ireshniov1/demo-app:1.3.0-86
c41486e455773da689968a6315f18e01ecffdd571e321fda4670fbb49430b1a5
```
* Access main page on `http://localhost:3000/`
</details>

******
