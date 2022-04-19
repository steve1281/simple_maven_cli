# CLI and mvn


## Switch to openjdk11. Basically, later version of java and maven don't work together.

```
sudo apt-get install openjdk-11-jdk
```

On a MAC,

```
brew install openjdk@11
```

(this is going to take a few minutes)

Switch installed java...


```
steve@minty:~/projects/dropwizard_example$ sudo update-alternatives --config java
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-17-openjdk-amd64/bin/java      1711      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
  2            /usr/lib/jvm/java-17-openjdk-amd64/bin/java      1711      manual mode
  3            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Select 1...

and

steve@minty:~/projects/dropwizard_example$ java --version
openjdk 11.0.14.1 2022-02-08
OpenJDK Runtime Environment (build 11.0.14.1+1-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.14.1+1-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)

```

### some MAC notes

My MAC uses Oracle JDKs.  In order to use OpenJDK, as installed above, I had to step through some hoops.

First, the command `/usr/libexec/java_home -V` doesn't list my openJDK installation:

```
[~/projects/home/maven_cli]  $ /usr/libexec/java_home -V
Matching Java Virtual Machines (5):
    17.0.2 (x86_64) "Oracle Corporation" - "Java SE 17.0.2" /Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
    17.0.1 (x86_64) "Oracle Corporation" - "Java SE 17.0.1" /Library/Java/JavaVirtualMachines/jdk-17.0.1.jdk/Contents/Home
    11.0.13 (x86_64) "Oracle Corporation" - "Java SE 11.0.13" /Library/Java/JavaVirtualMachines/jdk-11.0.13.jdk/Contents/Home
    1.8.311.11 (x86_64) "Oracle Corporation" - "Java" /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home
    1.8.0_311 (x86_64) "Oracle Corporation" - "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_311.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
```

So, we can't use that handy tool.

So, I used `find ~ -name java | grep java 2>/dev/null`  to locate where brew installed openjdk, then I aliased it.

```
[~/projects/home/maven_cli]  $ alias java
alias java='/Users/steve/OpenJDK/jdk-18.jdk/Contents/Home/bin/java'
```

And manually set the JAVA_HOME that maven needs:

```
[~/projects/home/maven_cli]  $ export JAVA_HOME=/Users/steve/OpenJDK/jdk-18.jdk/Contents/Home
```

This is a band-aid solution at best, but mvn does work after this.


## install maven

```
sudo apt install maven

# check that it installed
steve@minty:~/projects/dropwizard_example$ mvn -v
Apache Maven 3.6.3
Maven home: /usr/share/maven
Java version: 17.0.2, vendor: Private Build, runtime: /usr/lib/jvm/java-17-openjdk-amd64
Default locale: en_CA, platform encoding: UTF-8
OS name: "linux", version: "5.4.0-107-generic", arch: "amd64", family: "unix"

```

## create a HelloWorld demo project

```
cd maven

mvn archetype:generate -DgroupId=com.project777767ont.demo -DartifactId=HelloWorld -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

cd HelloWorld

steve@minty:~/projects/maven/HelloWorld$ tree
.
├── pom.xml
├── readme.md
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── project777767ont
    │               └── demo
    │                   └── App.java
    └── test
        └── java
            └── com
                └── project777767ont
                    └── demo
                        └── AppTest.java


```

## set the version in the pom.xml 

```
vim pom.xml
```

add in:

```
  </dependencies>
    <properties>
      <maven.compiler.source>11</maven.compiler.source>
      <maven.compiler.target>11</maven.compiler.target>
    </properties>
</project>
```

## try a compile

```
steve@minty:~/projects/maven$ mvn compile
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.982 s
[INFO] Finished at: 2022-04-15T11:20:15-04:00
[INFO] ------------------------------------------------------------------------
```


## package

```
steve@minty:~/projects/maven/HelloWorld$ mvn package
...

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.362 s
[INFO] Finished at: 2022-04-15T11:23:19-04:00
[INFO] -------------------------------------------------------------------

steve@minty:~/projects/maven/HelloWorld$ find . -name *.jar
./target/HelloWorld-1.0-SNAPSHOT.jar


```

## run

```
steve@minty:~/projects/maven/HelloWorld$ java -cp ./target/HelloWorld-1.0-SNAPSHOT.jar com.project777767ont.demo.App
Hello World!
```

## runnable jar

add to pom.xml

```
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>2.4</version>
        <configuration>
          <archive>
            <manifest>
                <mainClass>com.project777767ont.demo.App</mainClass>
            </manifest>
          </archive>
        </configuration>
      </plugin>
    </plugins>
  </build>
    <properties>

```

Rebuild `mvn package` and run:

```
steve@minty:~/projects/maven/HelloWorld$ java -jar ./target/HelloWorld-1.0-SNAPSHOT.jar
Hello World!
```

