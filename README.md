# Bjurr Maven

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/se.bjurr.maven/bjurr-maven/badge.svg)](https://maven-badges.herokuapp.com/maven-central/se.bjurr.maven/bjurr-maven)

Common Maven settings for my projects.

## Get Maven

You may want to use the [Maven wrapper](https://maven.apache.org/wrapper):

```sh
./mvnw wrapper:wrapper -Dmaven=3.8.1
```

## Show updates

```sh
./mvnw versions:display-dependency-updates
./mvnw versions:display-plugin-updates
```

## Update dependencies

```sh
./mvnw versions:use-latest-versions \
  -DallowSnapshots=false \
  -DregexVersion='^\d+(\.\d+)*(-Final)?$' \
  -DgenerateBackupPoms=false
```

## Set exact version in poms

```sh
./mvnw versions:set \
 -DnewVersion=latest-SNAPSHOT \
 -DgenerateBackupPoms=false
```

## Usage

As a parent in a library:

```xml
<parent>
    <groupId>se.bjurr.maven</groupId>
    <artifactId>bjurr-maven-parent</artifactId>
    <version>X</version>
</parent>
```

As a parent in a Maven plugin:

```xml
<parent>
    <groupId>se.bjurr.maven</groupId>
    <artifactId>bjurr-maven-plugin-parent</artifactId>
    <version>X</version>
</parent>

<packaging>maven-plugin</packaging>
```

Configure plugin with the plugin plugin.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-plugin-plugin</artifactId>
            <version>3.6.4</version>
            <configuration>
                <goalPrefix>changeme-maven-plugin</goalPrefix>
                <skipErrorNoDescriptorsFound>true</skipErrorNoDescriptorsFound>
                <helpPackageName>changeme.helpmojo</helpPackageName>
            </configuration>
            <executions>
                <execution>
                    <id>mojo-descriptor</id>
                    <goals>
                        <goal>descriptor</goal>
                    </goals>
                </execution>
                <execution>
                    <id>changeme-goal</id>
                    <goals>
                        <goal>helpmojo</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## Static code analysis

Spotbugs is bundled. Violations can be ignored with `@SuppressFBWarnings`. Also [Violations Maven Plugin](https://github.com/tomasbjerre/violations-maven-plugin) is used to present, and fail build based on, violations.

```
./mvnw verify 
```

May fail the build with:

```
[ERROR] org/assertj/snapshot/internal/utils/FileUtilsImpl.java
|          |               |          |      |                              |
| Reporter | Rule          | Severity | Line | Message                      |
|          |               |          |      |                              |
+----------+---------------+----------+------+------------------------------+
|          |               |          |      |                              |
| Spotbugs | MS_EXPOSE_REP | INFO     | 30   | Public  static  method   may |
|          |               |          |      | expose              internal |
|          |               |          |      | representation by  returning |
|          |               |          |      | array <p>  A  public  static |
|          |               |          |      | method returns  a  reference |
|          |               |          |      | to an array that is part  of |
|          |               |          |      | the  static  state  of   the |
|          |               |          |      | class. Any code  that  calls |
|          |               |          |      | this   method   can   freely |
|          |               |          |      | modify the underlying array. |
|          |               |          |      | One fix is to return a  copy |
|          |               |          |      | of the array.</p>            |
|          |               |          |      |                              |
+----------+---------------+----------+------+------------------------------+
Summary of org/assertj/snapshot/internal/utils/FileUtilsImpl.java
|          |      |      |       |       |
| Reporter | INFO | WARN | ERROR | Total |
|          |      |      |       |       |
+----------+------+------+-------+-------+
|          |      |      |       |       |
| Spotbugs | 1    | 0    | 0     | 1     |
|          |      |      |       |       |
+----------+------+------+-------+-------+
|          |      |      |       |       |
|          | 1    | 0    | 0     | 1     |
|          |      |      |       |       |
+----------+------+------+-------+-------+


Summary
|          |      |      |       |       |
| Reporter | INFO | WARN | ERROR | Total |
|          |      |      |       |       |
+----------+------+------+-------+-------+
|          |      |      |       |       |
| Spotbugs | 1    | 0    | 0     | 1     |
|          |      |      |       |       |
+----------+------+------+-------+-------+
|          |      |      |       |       |
|          | 1    | 0    | 0     | 1     |
|          |      |      |       |       |
+----------+------+------+-------+-------+

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  12.007 s
[INFO] Finished at: 2023-06-26T10:40:41+02:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal se.bjurr.violations:violations-maven-plugin:1.50.4:violations (default) on project assertj-snapshot: Too many violations found, max is 0 but found 1. You can adjust this with the 'maxViolations' configuration parameter. -> [Help 1]
```

## Releasing

Release and sign with:

```sh
#!/bin/bash

./mvnw se.bjurr.gitchangelog:git-changelog-maven-plugin:semantic-version \
  && (git commit -a -m "chore: setting version in pom" && git push || echo "No new version") \
  && ./mvnw release:prepare release:perform -B \
  && ./mvnw se.bjurr.gitchangelog:git-changelog-maven-plugin:git-changelog \
  && git commit -a -m "chore: updating changelog" \
  && git push \
  || git clean -f
```

To get signing working, there is a bug, you may need to do this first:

```sh
gpg -o /tmp/$(date +%s%3N) --sign .gitignore
```
