******

<details>
<summary>Exercise 1: Linux Mint Virtual Machine </summary>
 <br />

Downloaded `Linux Mint 21 "Vanessa"` from this [link](https://www.linuxmint.com/download.php)

Distribution
```shell
ireshniov@ireshniov-VirtualBox:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Linuxmint
Description:	Linux Mint 21
Release:	21
Codename:	vanessa
```
System kernel
```shell
ireshniov@ireshniov-VirtualBox:~$ uname -a
Linux ireshniov-VirtualBox 5.15.0-41-generic #44-Ubuntu SMP Wed Jun 22 14:20:53 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

It uses apt, apt-get package managers
```shell
ireshniov@ireshniov-VirtualBox:~$ ls /bin/ | grep ^apt$
apt
ireshniov@ireshniov-VirtualBox:~$ ls /bin/ | grep ^apt-get$
apt-get
ireshniov@ireshniov-VirtualBox:~$ ls /bin/ | grep ^yum$
ireshniov@ireshniov-VirtualBox:~$ yum
Command 'yum' not found, did you mean:
  command 'yum4' from deb nextgen-yum4 (4.5.2-6)
  command 'num' from deb quickcal (2.4-1)
  command 'uum' from deb freewnn-jserver (1.1.1~a021+cvs20130302-7build1)
  command 'sum' from deb coreutils (8.32-4.1ubuntu1)
  command 'zum' from deb perforate (1.2-5.1)
Try: sudo apt install <deb name>
```

So, Nano and Vi CLI editors are configured
```shell
ireshniov@ireshniov-VirtualBox:~$ ls /bin/ | grep ^nano$
nano
ireshniov@ireshniov-VirtualBox:~$ ls /usr/bin/ | grep ^nano$
nano
ireshniov@ireshniov-VirtualBox:~$ ls /bin/ | grep ^vi$
vi
ireshniov@ireshniov-VirtualBox:~$ ls /usr/bin/ | grep ^vi$
vi
ireshniov@ireshniov-VirtualBox:~$ ls /bin/ | grep ^vim$
ireshniov@ireshniov-VirtualBox:~$ ls /usr/bin/ | grep ^vim$
ireshniov@ireshniov-VirtualBox:~$ vim
Command 'vim' not found, but can be installed with:
sudo apt install vim         # version 2:8.2.3995-1ubuntu2.1, or
sudo apt install vim-tiny    # version 2:8.2.3995-1ubuntu2.1
sudo apt install neovim      # version 0.6.1-3
sudo apt install vim-athena  # version 2:8.2.3995-1ubuntu2
sudo apt install vim-gtk3    # version 2:8.2.3995-1ubuntu2
sudo apt install vim-nox     # version 2:8.2.3995-1ubuntu2
```
For my user is configured Bash shell
```shell
ireshniov@ireshniov-VirtualBox:~$ echo $SHELL
/bin/bash
```
</details>

******

<details>
<summary>Exercise 2: Bash Script - Install Java </summary>
 <br />

```shell
#!/bin/bash

apt update
apt install default-jre

version=$(java -version 2>&1 | grep "openjdk version" | awk '{print $3}' | grep -Po '(?<=^")\d{1,2}')

if [ "$version" == "" ]
then
        echo "Java not istalled."
elif [ "$version" -eq 1 ]
then
        echo "Installed old Java version"
elif [ "$version" -ge 11 ]
then
        echo "Java installation was successful"
fi
```

### Extract java version
```shell
version=$(java -version 2>&1 | grep "openjdk version" | awk '{print $3}' | grep -Po '(?<=^")\d{1,2}')
```

* `java -version` "print product version to the error stream and exit" conform docs. So we need to redirect error stream to standard output.
```shell
ireshniov@ireshniov-VirtualBox:~$ java -version
openjdk version "11.0.15" 2022-04-19
OpenJDK Runtime Environment (build 11.0.15+10-Ubuntu-0ubuntu0.22.04.1)
OpenJDK 64-Bit Server VM (build 11.0.15+10-Ubuntu-0ubuntu0.22.04.1, mixed mode, sharing)
```

* then, pipe output to `grep` and filter "openjdk version" that should find line with version or nothing, for example "openjdk version "11.0.15" 2022-04-19"
```shell
ireshniov@ireshniov-VirtualBox:~$ java -version 2>&1 | grep "openjdk version"
openjdk version "11.0.15" 2022-04-19
```

* after than we extract 3rd word using `awk` command
```shell
ireshniov@ireshniov-VirtualBox:~$ java -version 2>&1 | grep "openjdk version" | awk '{print $3}'
"11.0.15"
```

* filter with regular expression. Our goal is first 1-2 digits (As I understand, older java versions can be 1.7, 1.8 ...) after `"`. With that I extracted version number using `grep` with regular expression:
   Tested with older versions:
```shell
echo "openjdk version \"1.1.0.15\" 2022-04-19'" | awk '{print $3}' | grep -Po '(?<=^")\d{1,2}'
New versions:
```shell
java -version 2>&1 | grep "openjdk version" | awk '{print $3}' | grep -Po '(?<=^")\d{1,2}'
```

### Execution
```shell
ireshniov@ireshniov-VirtualBox:~$ ./install_java.sh
```
### Result
```shell
Ign:1 http://packages.linuxmint.com vanessa InRelease
Hit:2 http://packages.linuxmint.com vanessa Release                          
Hit:3 http://archive.ubuntu.com/ubuntu jammy InRelease                       
Get:4 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]  
Get:5 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [114 kB]
Get:7 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [99,8 kB]
Fetched 324 kB in 2s (191 kB/s)     
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
348 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
default-jre is already the newest version (2:1.11-72build2).
The following package was automatically installed and is no longer required:
  systemd-hwe-hwdb
Use 'sudo apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 348 not upgraded.
Java installation was successful
```

</details>

******

<details>
<summary>Exercise 3, 4, 5: Bash Script - User Processes Sorted </summary>
 <br />

```shell
#!/bin/bash

PS3='Please choose sort option: '
options=("Memory" "CPU" "None")

select opt in "${options[@]}"

do
        if [[ $opt =~ ^Memory|CPU$ ]]
        then
                read -p "How many processes to print? " -r lines_number
        fi

        case $opt in
                "Memory")
                        sort_by='rss'
                        break
                        ;;
                "CPU")
                        sort_by='pcpu'
                        break
                        ;;
                "None")
                        break
                        ;;
                *)
                        echo "Invalid option $REPLY"
                        ;;
        esac
done

ps axo user:20,pid,pcpu,pmem,vsz,rss,stat,start_time,time,cmd $(if ([ "$sort_by" != "" ]); then echo "--sort $sort_by"; fi;) \
        | awk -v user=$USER '{ if ($1 == user || $1 == "USER" ) { print $0} }' \
        | { if ([ "$lines_number" != "" ]); then head -n "$((lines_number+1))" ; else cat; fi; } \
```

</details>

******

<details>
<summary>Exercise 6, 7, 8, 9: Bash Script - Node App with Log Directory - Node App with Service user </summary>
 <br />

```shell
#!/bin/bash

apt update
apt install -y nodejs npm curl

echo "Node version: " $(node --version)
echo "NPM version: " $(npm --version)

log_directory=$(realpath $1)

if [ ! -d "$log_directory" ]
then
        echo "Directory $log_directory not exists. Creating it..."
        mkdir -p "$log_directory"
fi

useradd -m myapp
chown myapp -R "$log_directory"

runuser -l myapp -c "curl -L  https://node-envvars-artifact.s3.eu-west-2.amazonaws.com/bootcamp-node-envvars-project-1.0.0.tgz > node-project.tgz"
runuser -l myapp -c "tar -xvzf ./node-project.tgz"

runuser -l myapp -c "
    export APP_ENV=dev &&
    export DB_PWD=mysecret &&
    export DB_USER=myuser &&
    export LOG_DIR=$log_directory &&
    cd package &&
    npm install &&
    node server.js &"

ps aux | grep node | grep -v grep
netstat -tlpn | grep :3000
```

### Execution
```shell
ireshniov@ireshniov-VirtualBox:~$ sudo ./start_node_app.sh test1
```

</details>

******
