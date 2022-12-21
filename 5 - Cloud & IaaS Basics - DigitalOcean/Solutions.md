******

<details>
<summary>Exercise 0: Clone Git Repository </summary>
 <br />

```shell
git clone git@gitlab.com:devops-bootcamp3/node-project.git
cd node-project
git remote remove origin
git remote add origin https://gitlab.com/ireshniov/node-project.git
git branch -M main
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 1: Package NodeJS App </summary>
 <br />

```shell
(base) ➜  node-project git:(main) npm pack ./app 
(base) ➜  node-project git:(main) ls -l
total 168
-rw-r--r--  1 ireshniov  staff    541 Dec 20 22:52 README.md
drwxr-xr-x  8 ireshniov  staff    256 Dec 20 23:29 app
-rw-r--r--  1 ireshniov  staff  79333 Dec 20 23:30 bootcamp-node-project-1.0.0.tgz
```

</details>

******

<details>
<summary>Exercise 2: Create a new server </summary>
 <br />

Follow [the official documentation](https://docs.digitalocean.com/products/droplets/how-to/create)

</details>

******

<details>
<summary>Exercise 3: Prepare server to run Node App </summary>
 <br />

From [community article](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04#step-1-logging-in-as-root)
```shell
(base) ➜  ~ ssh root@164.92.162.28
Welcome to Ubuntu 22.10 (GNU/Linux 5.19.0-23-generic x86_64)

root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# adduser node-app
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# usermod -aG sudo node-app
# copy the root user’s .ssh directory, preserve the permissions, and modify the file owners
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# rsync --archive --chown=node-app:node-app ~/.ssh /home/node-app
root@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~# exit

(base) ➜  ~ ssh node-app@164.92.162.28
Welcome to Ubuntu 22.10 (GNU/Linux 5.19.0-23-generic x86_64)

node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ sudo apt install nodejs npm
```

</details>

******

<details>
<summary>Exercise 4: Copy App and package.json </summary>
 <br />

```shell
(base) ➜  node-project git:(main) scp bootcamp-node-project-1.0.0.tgz node-app@164.92.162.28:/home/node-app
```

</details>

******

<details>
<summary>Exercise 5: Run Node App </summary>
 <br />

```shell
(base) ➜  ~ ssh node-app@164.92.162.28
node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ tar zxvf bootcamp-node-project-1.0.0.tgz
node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ cd package
node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ npm install
node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ node server.js &

node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ ps aux | grep node
root       41676  0.0  1.1  17388  5324 ?        Ss   00:01   0:00 sshd: node-app [priv]
node-app   41679  0.0  0.9  18376  4568 ?        Ss   00:01   0:00 /lib/systemd/systemd --user
node-app   41680  0.0  0.8 103296  3864 ?        S    00:01   0:00 (sd-pam)
node-app   41727  0.0  0.7  17644  3772 ?        S    00:01   0:00 sshd: node-app@pts/0
node-app   41728  0.0  0.8   9084  3984 pts/0    Ss+  00:01   0:00 -bash
# here we can see
node-app   42383  0.0 10.2 604984 48836 ?        Sl   00:58   0:00 node server.js
root       42399  0.0  2.2  17384 10860 ?        Ss   01:08   0:00 sshd: node-app [priv]
node-app   42474  0.0  1.4  17640  7084 ?        S    01:08   0:00 sshd: node-app@pts/1
node-app   42475  0.0  1.1   9084  5324 pts/1    Ss   01:08   0:00 -bash
node-app   42486  0.0  0.6  10264  3272 pts/1    R+   01:12   0:00 ps aux
node-app   42487  0.0  0.4   6852  2072 pts/1    S+   01:12   0:00 grep --color=auto node

node-app@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ netstat -lpnt
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -
# here we can see
tcp6       0      0 :::3000                 :::*                    LISTEN      42383/node
```

</details>

******

<details>
<summary>Exercise 6: Access from browser - configure firewall </summary>
 <br />

**Add firewall with inbound rules**
<br />
Type: Custom 
<br />
Protocol: TCP
<br />
Port Range: 3000
<br />
Sources: All IPv4 All IPv6

</details>

******
