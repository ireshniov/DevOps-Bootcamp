******

<details>
<summary>Exercise 1: Install Nexus on a server </summary>
 <br />

1) create droplet with 4GB+ Memory
2) configure firewall for port 22
3) install java 8
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# apt install openjdk-8-jre-headless
```
4) download latest version via wget in /opt
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# cd /opt
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:/opt# wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```
5) untar package
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:/opt# tar -zxvf latest-unix.tar.gz
```
6) create nexus user and user group
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:/opt# adduser nexus
```
7) change sonatype-work and nexus directories ownership to nexus user and group
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:/opt# chown -R nexus:nexus sonatype-work
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:/opt# chown -R nexus:nexus nexus-3.44.0-01

root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:/opt# ls -la
total 257868
drwxr-xr-x  5 root  root       4096 Jan  5 21:56 .
drwxr-xr-x 19 root  root       4096 Dec 26 17:38 ..
drwxr-xr-x  4 root  root       4096 Dec 14 23:27 digitalocean
-rw-r--r--  1 root  root  210157775 Dec 14 14:38 latest-unix.tar.gz
-rw-r--r--  1 root  root   53874688 Jan  5 21:56 latest-unix.tar.gz.1
drwxr-xr-x 10 nexus nexus      4096 Dec 26 20:54 nexus-3.44.0-01
drwxr-xr-x  3 nexus nexus      4096 Dec 26 20:54 sonatype-work
```
</details>

******

<details>
<summary>Exercise 2: Create npm hosted repository </summary>
 <br />

1) create file type blob with `nodejs` path in `Administration/Blob Stores/Create Blob Store`
2) create npm hosted repository `nodejs-app` in `Administration/Repositories/Create repository`. Set nodejs blob created before. Set "Allow redeploy".

</details>

******

<details>
<summary>Exercise 3: Create user for team 1 </summary>
 <br />

1) create `nx-nodejs` role with `nx-repository-admin-npm-nodejs-app-*` and `nx-repository-view-npm-nodejs-app-*` privileges
2) create `nodejs-app` user with credentials. Apply `nx-nodejs` role to this user. Set this user active.
</details>

******

<details>
<summary>Exercise 4: Build and publish npm tar </summary>
 <br />

1) login in npm registry
```shell
(base) ➜  ~ npm login --registry=http://164.92.162.28:8081/repository/nodejs-app/
npm WARN adduser `adduser` will be split into `login` and `register` in a future version. `adduser` will become an alias of `register`. `login` (currently an alias) will become its own command.
npm notice Log in on http://164.92.162.28:8081/repository/nodejs-app/
Username: nodejs-app
Password:
Email: (this IS public) test@test.com
Logged in as nodejs-app on http://164.92.162.28:8081/repository/nodejs-app/.
```

2) Login in nexus as admin. In `Security/Realams` move from `Available` to `Active` column `npm Bearer Token Realm` realm rule to be able to publish to registry and logout if needed.

3) Build tar
```shell
(base) ➜  node-project git:(main) npm pack ./app 
(base) ➜  node-project git:(main) ls -l
total 168
-rw-r--r--  1 ireshniov  staff    541 Dec 20 22:52 README.md
drwxr-xr-x  8 ireshniov  staff    256 Dec 20 23:29 app
-rw-r--r--  1 ireshniov  staff  79333 Dec 20 23:30 bootcamp-node-project-1.0.0.tgz
```

4) Publish your to registry
```shell
(base) ➜  node-project git:(main) npm publish --registry=http://164.92.162.28:8081/repository/nodejs-app/ bootcamp-node-project-1.0.0.tgz
npm notice 
npm notice 📦  bootcamp-node-project@1.0.0
npm notice === Tarball Contents === 
npm notice 60.5kB images/profile-andrea.jpg
npm notice 17.5kB images/profile-ari.jpeg  
npm notice 1.2kB  index.html               
npm notice 283B   package.json             
npm notice 724B   server.js                
npm notice 205B   server.test.js           
npm notice === Tarball Details === 
npm notice name:          bootcamp-node-project                   
npm notice version:       1.0.0                                   
npm notice filename:      bootcamp-node-project-1.0.0.tgz         
npm notice package size:  79.3 kB                                 
npm notice unpacked size: 80.4 kB                                 
npm notice shasum:        d8e5c67e689cca3e808a0cf037e5f61fd90c98a9
npm notice integrity:     sha512-elKQPsF9Qxhvn[...]e5qSLK4JLEY0A==
npm notice total files:   6                                       
npm notice 
npm notice Publishing to http://164.92.162.28:8081/repository/nodejs-app/
```

</details>

******

<details>
<summary>Exercise 5: Create maven hosted repository </summary>
 <br />

1) create file type blob with `java-team-2` path in `Administration/Blob Stores/Create Blob Store`
2) create maven2 hosted repository `java-app` in `Administration/Repositories/Create repository`. Set version policy: "Snapshot". Set nodejs blob created before. Set "Allow redeploy".

</details>

******

<details>
<summary>Exercise 6: Create user for team 2 </summary>
 <br />

1) create `nx-java-team-2` role with `nx-repository-admin-maven2-java-app-*` and `nx-repository-view-maven2-java-app-*` privileges
2) create `java-app` user with credentials. Apply `nx-java-team-2` role to this user. Set this user active.

</details>

******

<details>
<summary>Exercise 7: Build and publish jar file </summary>
 <br />

1) Clean and build app
```shell
(base) ➜  java-gradle-app git:(main) ✗ ./gradlew clean  

BUILD SUCCESSFUL in 369ms
1 actionable task: 1 executed

(base) ➜  java-gradle-app git:(main) ✗ ./gradlew build

BUILD SUCCESSFUL in 1s
4 actionable tasks: 4 executed

```

2) Configure publishing with gradle
* Add and configure `maven-publish` plugin in `build.gradle`:
```text
apply plugin: 'maven-publish'

publishing {
    publications {
        maven(MavenPublication) {
            artifact("build/libs/bootcamp-java-project-$version"+".jar") {
                extension 'jar'
            }
        }
    }

    repositories {
        maven {
            name 'nexus'
            url "http://164.92.162.28:8081/repository/java-app/"
            allowInsecureProtocol = true
            credentials {
                username project.repoUser
                password project.repoPassword
            }
        }
    }
}
```
* Add `gradle.properties` with properties below:
```text
repoUser=java-app
repoPassword=java-app
```

3) Publish with gradle
```shell
(base) ➜  java-gradle-app git:(main) ✗ ./gradlew publish

BUILD SUCCESSFUL in 20s
2 actionable tasks: 2 executed
```

</details>

******

<details>
<summary>Exercise 8: Download from Nexus and start application </summary>
 <br />

1) Create `nexus` role with `nx-repository-admin-npm-nodejs-app-*`, `nx-repository-view-npm-nodejs-app-*`, `nx-repository-admin-maven2-java-app-*`, `nx-repository-view-maven2-java-app-*` privileges
2) Create `nexus` user with credentials. Apply `nexus` role to this user. Set this user active.
3) Using Nexus REST API, fetch the download URL info for latest nodejs app
```shell
(base) ➜  ~ ssh root@164.92.162.28
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# curl -u nexus:nexus -X GET 'http://164.92.162.28:8081/service/rest/v1/components?repository=nodejs-app&sort=version'
{
  "items" : [ {
    "id" : "bm9kZWpzLWFwcDpjNjBiYzZiOTYxMmY0N2QzNzRhODA3NDJlNzZlMjhmMg",
    "repository" : "nodejs-app",
    "format" : "npm",
    "group" : null,
    "name" : "bootcamp-node-project",
    "version" : "1.0.0",
    "assets" : [ {
      "downloadUrl" : "http://164.92.162.28:8081/repository/nodejs-app/bootcamp-node-project/-/bootcamp-node-project-1.0.0.tgz",
      "path" : "bootcamp-node-project/-/bootcamp-node-project-1.0.0.tgz",
      "id" : "bm9kZWpzLWFwcDpkZmJlZjA5ZWZlMTY0NGVhZGM1YTFkMTAwOTk1YzZmYQ",
      "repository" : "nodejs-app",
      "format" : "npm",
      "checksum" : {
        "sha1" : "d8e5c67e689cca3e808a0cf037e5f61fd90c98a9"
      },
      "contentType" : "application/gzip",
      "lastModified" : "2023-01-18T23:18:37.092+00:00",
      "lastDownloaded" : null,
      "uploader" : "nodejs-app",
      "uploaderIp" : "158.181.78.72",
      "fileSize" : 79333,
      "blobCreated" : "2023-01-18T23:18:37.092+00:00",
      "lastDownloaded" : null,
      "npm" : {
        "name" : "bootcamp-node-project",
        "version" : "1.0.0"
      }
    } ]
  } ],
  "continuationToken" : null
}
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~#
```
4) Fetch artefact (tar file) using `downloadUrl`
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# curl -u nexus:nexus -X GET 'http://164.92.162.28:8081/repository/nodejs-app/bootcamp-node-project/-/bootcamp-node-project-1.0.0.tgz' --output bootcamp-node-project-1.0.0.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 79333  100 79333    0     0  8564k      0 --:--:-- --:--:-- --:--:-- 9684k
```
5) Untar artefact
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# tar -zxvf bootcamp-node-project-1.0.0.tgz
package/index.html
package/images/profile-ari.jpeg
package/images/profile-andrea.jpg
package/server.js
package/server.test.js
package/package.json
```
6) Run on server
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# npm --prefix package run start &
[1] 70900
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~#
> bootcamp-node-project@1.0.0 start
> node server.js

app listening on port 3000!

root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# ps aux | grep node
root       70911  0.0  0.0   2736   964 pts/1    S    Jan18   0:00 sh -c -- node server.js
root       70912  0.0  1.2 611564 50164 pts/1    Sl   Jan18   0:00 node server.js
root       71075  0.0  0.0   6852  2064 pts/0    S+   00:12   0:00 grep --color=auto node

root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# netstat -lpnt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      567/systemd-resolve
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      2211/java
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      567/systemd-resolve
tcp        0      0 127.0.0.1:34245         0.0.0.0:*               LISTEN      2211/java
tcp6       0      0 :::22                   :::*                    LISTEN      1/init
tcp6       0      0 :::3000                 :::*                    LISTEN      70912/node
```

</details>

******

<details>
<summary>Exercise 9: Automate </summary>
 <br />

```shell
#!/bin/bash

user="${NEXUS_USER}"
password="${NEXUS_PASSWORD}"
nexus_ip="${NEXUS_IP}"
node_repo="${NODE_REPO}"

# todo validate input

# save the artifact details in a json file
curl -u $user:$password -X GET "http://$nexus_ip:8081/service/rest/v1/components?repository=$node_repo&sort=version" | jq "." > artifact.json


# grab the download url from the saved artifact details using 'jq' json processor tool
artifactDownloadUrl=$(jq '.items[].assets[].downloadUrl' artifact.json --raw-output)

# fetch the artifact with the extracted download url using 'wget' tool
# wget --http-user=$user --http-password=$password $artifactDownloadUrl
# or via curl
curl -u $user:$password -X GET $artifactDownloadUrl --output bootcamp-node-project.tgz

# untar artefact
tar -zxvf bootcamp-node-project.tgz

# run application
cd package
npm install
npm run start &
```
Execute script
```shell
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# NEXUS_USER=nexus NEXUS_PASSWORD=nexus NEXUS_IP=164.92.162.28 NODE_REPO=nodejs-app bash start-nodejs-app.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1153  100  1153    0     0  32171      0 --:--:-- --:--:-- --:--:-- 32942
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 79333  100 79333    0     0  4208k      0 --:--:-- --:--:-- --:--:-- 4842k
package/index.html
package/images/profile-ari.jpeg
package/images/profile-andrea.jpg
package/server.js
package/server.test.js
package/package.json
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN deprecated source-map-resolve@0.5.3: See https://github.com/lydell/source-map-resolve#deprecated
npm WARN deprecated w3c-hr-time@1.0.2: Use your platform\'s native performance.now() and performance.timeOrigin.
npm WARN deprecated sane@4.1.0: some dependency vulnerabilities fixed, support for node < 10 dropped, and newer ECMAScript syntax/features added

added 564 packages, and audited 565 packages in 29s

29 packages are looking for funding
  run `npm fund` for details

3 high severity vulnerabilities

To address issues that do not require attention, run:
  npm audit fix

To address all issues, run:
  npm audit fix --force

Run `npm audit` for details.
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~#
> bootcamp-node-project@1.0.0 start
> node server.js

app listening on port 3000!
```

</details>

******
