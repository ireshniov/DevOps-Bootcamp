******

<details>
<summary>Exercise 1: Clone and create new repository </summary>
 <br />

```shell
git clone git@gitlab.com:devops-bootcamp3/git-project.git
cd git-project
rm -rf ./.git
git init --initial-branch=main
git remote add origin git@gitlab.com:ireshniov/git-project.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 2: .gitignore </summary>
 <br />

```shell
touch .gitignore
echo .idea >> .gitignore
echo .DS_Store >> .gitignore
git rm --cached -r .idea
git rm --cached -r .DS_Store
git add .
git commit -m "Add gitignore file"
git push origin main
```

</details>

******

<details>
<summary>Exercise 3: Feature branch </summary>
 <br />

```shell
git checkout -b feature/changes
# edit build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    # this line
    compile group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '6.6'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
git diff
git add .
git commit -m "upgrade logstash-logback-encoder up to 6.6"
# edit src/main/webapp/index.html
    <h1>Team member roles</h1>
    # this line
     <img src="https://www.careeraddict.com/uploads/article/58721/illustration-group-people-team-meeting.jpg" width="" />
    <ul>
git diff
git add .
git commit -m "add image to index.html"
git push --set-upstream origin feature/changes
```

</details>

******

<details>
<summary>Exercise 4: Bugfix branch </summary>
 <br />

```shell
git checkout -b bugfix/spelling-error
# edit src/main/java/com/example/Application.java file
@PostConstruct
public void init()
{
    Logger log = LoggerFactory.getLogger(Application.class);
    # this line
    log.info("Java app started");
}
git diff
git add .
git commit -m "fix spelling error"
git push --set-upstream origin bugfix/spelling-error
```

</details>

******

<details>
<summary>Exercise 5: Merge request </summary>
 <br />

```shell
git checkout main
git merge feature/changes 
git push origin
```

Of course better to merge via PR in repository UI

</details>

******

<details>
<summary>Exercise 6: Fix merge conflict </summary>
 <br />

```shell
git checkout -b bugfix/spelling-error
# edit build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    # this line
    compile group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '6.2'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
git diff
git add .
git commit -m "upgrade logstash-logback-encoder up to 6.2"
git merge main
# fix merge conflict
# ---
git push 
```

</details>

******

<details>
<summary>Exercise 7: Revert commit </summary>
 <br />

```shell
git checkout bugfix/spelling-error

# edit src/main/webapp/index.html line 11
<li>Sarah - Full stack devloper</li>

git add .
git commit -m "fix spelling error"

# edit src/main/webapp/index.html line 9
<img src="other src" width="" />

git add .
git commit -m "set other image url"

git push

git revert HEAD
git push
```

</details>

******

<details>
<summary>Exercise 8: Reset commit </summary>
 <br />

```shell
git checkout bugfix/spelling-error

# edit src/main/webapp/index.html line 15
<li>Bruno - DevOps engineer</li>

git add .
git commit -m "edit Bruno's job description"
git reset --hard HEAD~
```

</details>

******

<details>
<summary>Exercise 9: Merge </summary>
 <br />

```shell
git checkout main
git merge bugfix/spelling-error
```

</details>

******

<details>
<summary>Exercise 10: Delete branches </summary>
 <br />

```shell
# delete branch remotely
git push origin --delete bugfix/spelling-error

# delete branch locally
git branch -D bugfix/spelling-error
```

</details>

******
