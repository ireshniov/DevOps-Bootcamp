******

<details>
<summary>Exercise 0: Clone project and create own Git repository </summary>
 <br />

```shell
git clone git@gitlab.com:devops-bootcamp3/java-gradle-app.git
cd java-gradle-app
git remote remove origin
git remote add origin https://gitlab.com/ireshniov/java-gradle-app.git
git branch -M main
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 1: Build jar artifact </summary>
 <br />

```shell
./gradlew build
```

</details>

******

<details>
<summary>Exercise 2: Run tests </summary>
 <br />

```shell
./gradlew build
```

</details>

******

<details>
<summary>Exercise 2: Run tests </summary>
 <br />

```shell
# change "true" string to true boolean
boolean result = myApp.getCondition(true);

./gradlew test    
```

</details>

******

<details>
<summary>Exercise 3: Clean and build App </summary>
 <br />

```shell
./gradlew clean
./gradlew build   
```

</details>

******

<details>
<summary>Exercise 4: Start application </summary>
 <br />

```shell
java -jar ./build/libs/bootcamp-java-project-1.0-SNAPSHOT.jar
```

</details>

******

<details>
<summary>Exercise 5: Start App with 2 Parameters </summary>
 <br />

```shell
# Add this code snippet at line 16
Logger log = LoggerFactory.getLogger(Application.class);
try {
    String one = args[0];
    String two = args[1];
    log.info("Application will start with the parameters {} and {}", one, two);
} catch (Exception e) {
    log.info("No parameters provided");
}

./gradlew build
java -jar ./build/libs/bootcamp-java-project-1.0-SNAPSHOT.jar test1 test2
```

</details>

******
