******

<details>
<summary>Exercise 0: Clone Git repository and create your own </summary>
 <br />

```shell
git clone git@gitlab.com:devops-bootcamp3/bootcamp-java-mysql.git
cd bootcamp-java-mysql
git remote remove origin
git remote add origin https://gitlab.com/ireshniov/bootcamp-java-mysql.git
git branch -M main
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 1: Start Mysql container </summary>
 <br />

1) Create bridge network
```shell
(base) ➜  bootcamp-java-mysql git:(main) docker network create mysql-network
4a54955207b072e3707c5bea94142762ae70fd82f76041ff7797692692b55ed5
```

2) Start mysql docker container:
```shell

(base) ➜  bootcamp-java-mysql git:(main) docker run -p 3306:3306 \
--name mysql \
--platform=linux/amd64 \
--network=mysql-network \
-e MYSQL_ROOT_PASSWORD=toor \
-e MYSQL_DATABASE=team-db \
-e MYSQL_USER=admin \
-e MYSQL_PASSWORD=secret \
-d mysql:5.7
Unable to find image 'mysql:5.7' locally
5.7: Pulling from library/mysql
docker: no matching manifest for linux/arm64 in the manifest list entries.
See 'docker run --help'.
(base) ➜  bootcamp-java-mysql git:(main) docker run -p 3306:3306 --name mysql --platform=linux/amd64 \
-e MYSQL_ROOT_PASSWORD=toor \
-e MYSQL_DATABASE=team-db \
-e MYSQL_USER=admin \
-e MYSQL_PASSWORD=secret \
-d mysql:5.7
Unable to find image 'mysql:5.7' locally
e048d0a38742: Pull complete 
c7847c8a41cb: Pull complete 
351a550f260d: Pull complete 
8ce196d9d34f: Pull complete 
17febb6f2030: Pull complete 
d4e426841fb4: Pull complete 
fda41038b9f8: Pull complete 
f47aac56b41b: Pull complete 
a4a90c369737: Pull complete 
97091252395b: Pull complete 
84fac29d61e9: Pull complete 
Digest: sha256:8cf035b14977b26f4a47d98e85949a7dd35e641f88fc24aa4b466b36beecf9d6
Status: Downloaded newer image for mysql:5.7
076ba95ad3693ae9509e1d5e0408e03386518017a0656061b5aef34b665aabf4
```

2) Build app
```shell
(base) ➜  bootcamp-java-mysql git:(main) ./gradlew build
> Task :compileJava
> Task :processResources
> Task :classes
> Task :bootJar
> Task :jar SKIPPED
> Task :assemble
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> Task :check
> Task :build

BUILD SUCCESSFUL in 1s
5 actionable tasks: 5 executed
```

3) Export env variables:
```shell
(base) ➜  bootcamp-java-mysql git:(main) export DB_USER=admin
(base) ➜  bootcamp-java-mysql git:(main) export DB_PWD=secret
(base) ➜  bootcamp-java-mysql git:(main) export DB_SERVER=127.0.0.1
(base) ➜  bootcamp-java-mysql git:(main) export DB_NAME=team-db
```

4) Start app:
```shell
(base) ➜  bootcamp-java-mysql git:(main) java -jar build/libs/bootcamp-java-mysql-project-1.0-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.2.RELEASE)

2023-02-05 18:09:19.727  INFO 28615 --- [           main] com.example.Application                  : Starting Application on MacBook-Pro-4.local with PID 28615 (/Users/ireshniov/DevOps Bootcamp/7 - Containers with Docker/bootcamp-java-mysql/build/libs/bootcamp-java-mysql-project-1.0-SNAPSHOT.jar started by ireshniov in /Users/ireshniov/DevOps Bootcamp/7 - Containers with Docker/bootcamp-java-mysql)
2023-02-05 18:09:19.729  INFO 28615 --- [           main] com.example.Application                  : No active profile set, falling back to default profiles: default
2023-02-05 18:09:20.316  INFO 28615 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-02-05 18:09:20.323  INFO 28615 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-02-05 18:09:20.323  INFO 28615 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.29]
2023-02-05 18:09:20.361  INFO 28615 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-02-05 18:09:20.361  INFO 28615 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 599 ms
2023-02-05 18:09:20.676  INFO 28615 --- [           main] com.example.Application                  : Java app started
2023-02-05 18:09:20.776  INFO 28615 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2023-02-05 18:09:20.811  INFO 28615 --- [           main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page: class path resource [static/index.html]
2023-02-05 18:09:20.865  INFO 28615 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-02-05 18:09:20.867  INFO 28615 --- [           main] com.example.Application                  : Started Application in 1.35 seconds (JVM running for 1.612)
2023-02-05 18:09:35.414  INFO 28615 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2023-02-05 18:09:35.414  INFO 28615 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2023-02-05 18:09:35.420  INFO 28615 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 6 ms
```

5) Test app:
```shell
(base) ➜  bootcamp-java-mysql git:(main) curl -X GET "http://127.0.0.1:8080/get-data" | jq    
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   212    0   212    0     0  14587      0 --:--:-- --:--:-- --:--:-- 26500
[
  {
    "name": "Sarah",
    "role": "Full stack developer"
  },
  {
    "name": "Bobby",
    "role": "React developer"
  },
  {
    "name": "Ari",
    "role": "Java developer"
  },
  {
    "name": "Andrea",
    "role": "DevOps engineer"
  },
  {
    "name": "Bruno",
    "role": "IT operations"
  }
]
```

</details>

******

<details>
<summary>Exercise 2: Start Mysql GUI container </summary>
 <br />

1) Start Phpmyadmin container
```shell
(base) ➜  bootcamp-java-mysql git:(main) docker run -p 8083:80 \
--name phpmyadmin \
--platform=linux/amd64 \
--network=mysql-network \
-e PMA_HOST=mysql \
-d phpmyadmin
Unable to find image 'phpmyadmin:latest' locally
latest: Pulling from library/phpmyadmin
01b5b2efb836: Pull complete 
45244a9928d1: Pull complete 
139d4815e950: Pull complete 
9a420fd884ad: Pull complete 
1de46a46cfcd: Pull complete 
9cc46e699e97: Pull complete 
9a8d67ebc9db: Pull complete 
dc59c4386191: Pull complete 
9195cab71d03: Pull complete 
9eaa1559de60: Pull complete 
e98ea42e90a6: Pull complete 
d25586855c63: Pull complete 
0c7c6d5ce997: Pull complete 
96e5415e261a: Pull complete 
1ff2782a8ae8: Pull complete 
0b85949bbffa: Pull complete 
ed7aad4fc2fb: Pull complete 
d22a5530765a: Pull complete 
Digest: sha256:7c8751488b72255047bdc38ea1e9bf6b33b163a3c15ae276b2ac3c1a633a53c6
Status: Downloaded newer image for phpmyadmin:latest
d61ed3b90a219263ae54bc6993cd1a1746c76613da04b7d570a9ea5b8de743bf
```

2) Access admin panel on 127.0.0.1:8083
3) Login with `root/toor` or `admin/secret`

</details>

******

<details>
<summary>Exercise 3: Use docker-compose for Mysql and Phpmyadmin </summary>
 <br />

1) Create docker-compose configuration file with mysql and phpmyadmin containers with volumes
```yaml
version: '3.7'
services:
  mysql:
    platform: linux/amd64
    image: mysql:5.7
    restart: always
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=toor
      - MYSQL_DATABASE=team-db
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=secret
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - mysql-network
  phpmyadmin:
    platform: linux/amd64
    image: phpmyadmin
    restart: always
    ports:
      - "8083:80"
    environment:
      - PMA_HOST=mysql
    networks:
      - mysql-network
    depends_on:
      - "mysql"
volumes:
  mysql-data:
    driver: local
networks:
  mysql-network:
    driver: bridge
```

2) Access admin panel on 127.0.0.1:8083
3) Login with `root/toor` or `admin/secret`
</details>

******

<details>
<summary>Exercise 4: Dockerize your Java Application </summary>
 <br />

1) Fix `build.gradle` file as Gradle version = 7.5.1:
```text
compile -> implementation
testCompile -> testImplementation
```
```text
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '5.2'
    implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.22'
    testImplementation group: 'junit', name: 'junit', version: '4.12'
}
```

2) Create Dockerfile:
```dockerfile
FROM --platform=linux/amd64 gradle:7.6.0-jdk19-alpine as builder
#FROM --platform=linux/amd64 gradle:7.6.0-jdk17-alpine as builder

WORKDIR /bootcamp-java-mysql-app

COPY --chown=gradle:gradle build.gradle .
COPY --chown=gradle:gradle gradlew .
COPY --chown=gradle:gradle gradlew.bat .
COPY --chown=gradle:gradle settings.gradle .
COPY --chown=gradle:gradle gradle gradle
COPY --chown=gradle:gradle src src

RUN ./gradlew build
# works with gradle:7.6.0-jdk17-alpine
# RUN gradle build

FROM --platform=linux/amd64 openjdk:19-jdk-alpine as release

WORKDIR /bootcamp-java-mysql-app

RUN addgroup --gid 1001 --system bootcamp-java-mysql && \
    adduser -S --uid 1001 -G bootcamp-java-mysql bootcamp-java-mysql && \
    chown -R bootcamp-java-mysql:bootcamp-java-mysql /bootcamp-java-mysql-app

RUN mkdir -p /var/logs/bootcamp-java-mysql && \
    chown -R bootcamp-java-mysql:bootcamp-java-mysql /var/logs/bootcamp-java-mysql

COPY --chown=bootcamp-java-mysql:bootcamp-java-mysql --from=builder /bootcamp-java-mysql-app/build/libs/bootcamp-java-mysql-project.jar .

USER 1001:1001

CMD ["java", "-jar", "bootcamp-java-mysql-project.jar"]
```
</details>

******

<details>
<summary>Exercise 5: Build and push Java Application Docker Image </summary>
 <br />

1) Start nexus as docker container
```yaml
# nexus-docker-compose.yaml
version: '3.7'
services:
  nexus:
#    image: sonatype/nexus3
    image: klo2k/nexus3
    restart: always
#    platform: linux/amd64
    volumes:
      - "nexus-data:/sonatype-work"
    ports:
      - "8081:8081"
      - "8085:8085"

volumes:
  nexus-data:
    driver: local
```

```shell
(base) ➜  bootcamp-java-mysql git:(main) ✗ docker-compose -f nexus-docker-compose.yaml up -d
```
2) Create volume for hosted docker repository; create hosted docker repository
3) Create `nx-docker-java` role with `nx-repository-admin-docker-docker-java-app-*` and `nx-repository-view-docker-docker-java-app-*` privileges
4) Create `docker-java-app` and apply `nx-docker-java` role
5) Add `Docker Bearer Token` Realm
6) On Linux server - to add an insecure docker registry, add the file /etc/docker/daemon.json with the following content. In my case edit Docker desktop settings
```text
{
  "insecure-registries" :   "insecure-registries": [ "http://127.0.0.1:8085" ]
}
```

7) Docker login
```shell
(base) ➜  bootcamp-java-mysql git:(main) ✗ docker login 127.0.0.1:8085                      
Authenticating with existing credentials...
Login did not succeed, error: Error response from daemon: login attempt to http://127.0.0.1:8085/v2/ failed with status: 401 Unauthorized
Username (docker-hosted): docker-java-app
Password: 
Login Succeeded
```

8) Build image
```shell
(base) ➜  bootcamp-java-mysql git:(main) ✗ docker build --target release --platform=linux/amd64 -t bootcamp-java-mysql-project:1.0 .
[+] Building 143.8s (20/20) FINISHED                                                                                                                                                                            
 => [internal] load build definition from Dockerfile                                                                                                                                                       0.0s
 => => transferring dockerfile: 1.18kB                                                                                                                                                                     0.0s
 => [internal] load .dockerignore                                                                                                                                                                          0.0s
 => => transferring context: 2B                                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/openjdk:19-jdk-alpine                                                                                                                                   0.8s
 => [internal] load metadata for docker.io/library/gradle:7.6.0-jdk19-alpine                                                                                                                               0.6s
 => [internal] load build context                                                                                                                                                                          0.0s
 => => transferring context: 903B                                                                                                                                                                          0.0s
 => [release 1/5] FROM docker.io/library/openjdk:19-jdk-alpine@sha256:1686909f4ca66f3e13463e2b00a1c53808aa155f81ae9a8aad8f4b89420d91ef                                                                     0.0s
 => [builder 1/9] FROM docker.io/library/gradle:7.6.0-jdk19-alpine@sha256:ef67cca626220edf0ceec8b6482c13e2f6c1cd9bf2af8cca236e1834597cead6                                                                 0.0s
 => CACHED [release 2/5] WORKDIR /bootcamp-java-mysql-app                                                                                                                                                  0.0s
 => CACHED [release 3/5] RUN addgroup --gid 1001 --system bootcamp-java-mysql &&     adduser -S --uid 1001 -G bootcamp-java-mysql bootcamp-java-mysql &&     chown -R bootcamp-java-mysql:bootcamp-java-m  0.0s
 => CACHED [release 4/5] RUN mkdir -p /var/logs/bootcamp-java-mysql &&     chown -R bootcamp-java-mysql:bootcamp-java-mysql /var/logs/bootcamp-java-mysql                                                  0.0s
 => CACHED [builder 2/9] WORKDIR /bootcamp-java-mysql-app                                                                                                                                                  0.0s
 => CACHED [builder 3/9] COPY --chown=gradle:gradle build.gradle .                                                                                                                                         0.0s
 => CACHED [builder 4/9] COPY --chown=gradle:gradle gradlew .                                                                                                                                              0.0s
 => CACHED [builder 5/9] COPY --chown=gradle:gradle gradlew.bat .                                                                                                                                          0.0s
 => CACHED [builder 6/9] COPY --chown=gradle:gradle settings.gradle .                                                                                                                                      0.0s
 => CACHED [builder 7/9] COPY --chown=gradle:gradle gradle gradle                                                                                                                                          0.0s
 => CACHED [builder 8/9] COPY --chown=gradle:gradle src src                                                                                                                                                0.0s
 => [builder 9/9] RUN ./gradlew build                                                                                                                                                                    142.8s
 => [release 5/5] COPY --chown=bootcamp-java-mysql:bootcamp-java-mysql --from=builder /bootcamp-java-mysql-app/build/libs/bootcamp-java-mysql-project.jar .                                                0.0s
 => exporting to image                                                                                                                                                                                     0.0s
 => => exporting layers                                                                                                                                                                                    0.0s
 => => writing image sha256:937dda78a131fb8c9b12fddf61fd5ba3a79d65b21e0ecd7e6d016fd8c3b47cfa                                                                                                               0.0s
 => => naming to docker.io/library/bootcamp-java-mysql-project:1.0
```

9) Push to docker repository
```shell
(base) ➜  bootcamp-java-mysql git:(main) ✗ docker tag bootcamp-java-mysql-project:1.0 127.0.0.1:8085/bootcamp-java-mysql-project:1.0
(base) ➜  bootcamp-java-mysql git:(main) ✗ docker push 127.0.0.1:8085/bootcamp-java-mysql-project:1.0                               
The push refers to repository [127.0.0.1:8085/bootcamp-java-mysql-project]
ee7852095a3d: Pushed 
cca755010743: Pushed 
867ee171e7fb: Pushed 
806288a60aa0: Pushed 
c82dcf304676: Pushed 
0828297c8137: Pushed 
994393dc58e7: Pushed 
1.0: digest: sha256:7da6017a13d41a85dac8faa20fe199153637e12bb4d9f9ab1266d487a0b33bec size: 1785
```
</details>

******

<details>
<summary>Exercise 6: Add application to docker-compose </summary>
 <br />

1) Add .env file
```
DB_USER=admin
DB_PWD=secret
DB_SERVER=mysql
DB_NAME=team-db
```

2) Add `bootcamp-java-mysql-app` service to docker-compose
```yaml
version: '3.7'
services:
  bootcamp-java-mysql-app:
    platform: linux/amd64
    image: 127.0.0.1:8085/bootcamp-java-mysql-project:1.0
    restart: always
    ports:
      - '8080:8080'
    env_file: ./.env
    depends_on:
      - mysql
    networks:
      - bootcamp-java-mysql-network
  mysql:
    platform: linux/amd64
    image: mysql:5.7
    restart: always
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=toor
      - MYSQL_DATABASE=team-db
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=secret
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - bootcamp-java-mysql-network
  phpmyadmin:
    platform: linux/amd64
    image: phpmyadmin
    restart: always
    ports:
      - "8083:80"
    environment:
      - PMA_HOST=mysql
    networks:
      - bootcamp-java-mysql-network
    depends_on:
      - mysql
volumes:
  mysql-data:
    driver: local
networks:
  bootcamp-java-mysql-network:
    driver: bridge
```

</details>

******

<details>
<summary>Exercise 7: Run application on server with docker-compose </summary>
 <br />

1) Docker compose up
```shell
(base) ➜  bootcamp-java-mysql git:(main) ✗ docker-compose up -d      
[+] Running 4/4
 ⠿ Network bootcamp-java-mysql_bootcamp-java-mysql-network  Created                                                                                                                                        0.0s
 ⠿ Container bootcamp-java-mysql-mysql-1                    Started                                                                                                                                        0.4s
 ⠿ Container bootcamp-java-mysql-phpmyadmin-1               Started                                                                                                                                        0.7s
 ⠿ Container bootcamp-java-mysql-bootcamp-java-mysql-app-1  Started 
```

2) Test app
```shell
(base) ➜  bootcamp-java-mysql git:(main) ✗ curl -X GET "http://127.0.0.1:8080/get-data" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   212    0   212    0     0   2243      0 --:--:-- --:--:-- --:--:--  2554
[
  {
    "name": "Sarah",
    "role": "Full stack developer"
  },
  {
    "name": "Bobby",
    "role": "React developer"
  },
  {
    "name": "Ari",
    "role": "Java developer"
  },
  {
    "name": "Andrea",
    "role": "DevOps engineer"
  },
  {
    "name": "Bruno",
    "role": "IT operations"
  }
]
```

</details>

******
