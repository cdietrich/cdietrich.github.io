---
layout: post
title: Code Coverage for Xtend
date: 2016-01-12 23:00:27.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Xtext
tags:
- Xtend
- Coverage
author: Christian Dietrich
---
This is a short blog on how to do code coverage reports with Xtend.
Imagine you have some nice Xtend Code like this one
```
package demo
 
class Demo {
 
    def doSomething() {
        return 1
    }
 
    def doSomethingElse() {
        return 2
    }
 
}
```

wouldn’t it be nice to have code coverage like

![Coverage]({{ "/assets/img/blog/bildschirmfoto-2016-01-12-um-23-57-44.png" | absolute_url }})

Using the xtend-maven-plugin in version 2.9.0+ this works more or less out of the box

```
<plugin>
    <groupId>org.eclipse.xtend</groupId>
    <artifactId>xtend-maven-plugin</artifactId>
    <version>2.9.0</version>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>testCompile</goal>
                <goal>xtend-install-debug-info</goal>
                <goal>xtend-test-install-debug-info</goal>
            </goals>
            <configuration>
                <xtendAsPrimaryDebugSource>true</xtendAsPrimaryDebugSource>
                <outputDirectory>${project.build.directory}/xtend-gen/main</outputDirectory>
                <testOutputDirectory>${project.build.directory}/xtend-gen/test</testOutputDirectory>
                <writeTraceFiles>true</writeTraceFiles>
            </configuration>
        </execution>
    </executions>
</plugin>
``` 

The interesting - new - parts are the goals xtend-install-debug-info and xtend-test-install-debug-info as well as the configuration xtendAsPrimaryDebugSource.

Here is my complete sample pom

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>demo</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <build>
        <plugins>
 
 
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <compilerId>eclipse</compilerId>
                    <source>1.8</source>
                    <target>1.8</target>
                    <optimize>true</optimize>
 
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.codehaus.plexus</groupId>
                        <artifactId>plexus-compiler-eclipse</artifactId>
                        <version>2.6</version>
                        <scope>runtime</scope>
                    </dependency>
                </dependencies>
            </plugin>
 
 
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.7.5.201505241946</version>
 
 
                <executions>
                    <!-- Prepares the property pointing to the JaCoCo runtime agent which 
                        is passed as VM argument when Maven the Surefire plugin is executed. -->
                    <execution>
                        <id>pre-unit-test</id>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                        <configuration>
                            <!-- Sets the path to the file which contains the execution data. -->
                            <destFile>${project.build.directory}/coverage-reports/jacoco-ut.exec</destFile>
                            <!-- Sets the name of the property containing the settings for JaCoCo 
                                runtime agent. -->
                            <propertyName>surefireArgLine</propertyName>
 
                        </configuration>
                    </execution>
                    <!-- Ensures that the code coverage report for unit tests is created 
                        after unit tests have been run. -->
                    <execution>
                        <id>post-unit-test</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                        <configuration>
 
                            <!-- Sets the path to the file which contains the execution data. -->
                            <dataFile>${project.build.directory}/coverage-reports/jacoco-ut.exec</dataFile>
                            <!-- Sets the output directory for the code coverage report. -->
                            <outputDirectory>${project.reporting.outputDirectory}/jacoco-ut</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
 
            </plugin>
 
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.15</version>
                <configuration>
                    <!-- Sets the VM argument line used when unit tests are run. -->
                    <argLine>${surefireArgLine}</argLine>
                    <!-- Skips unit tests if the value of skip.unit.tests property is true -->
                    <skipTests>${skip.unit.tests}</skipTests>
                    <!-- Excludes integration tests when unit tests are run. -->
                    <excludes>
                        <exclude>**/IT*.java</exclude>
                    </excludes>
                </configuration>
            </plugin>
 
            <plugin>
                <groupId>org.eclipse.xtend</groupId>
                <artifactId>xtend-maven-plugin</artifactId>
                <version>2.9.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                            <goal>xtend-install-debug-info</goal>
                            <goal>xtend-test-install-debug-info</goal>
                        </goals>
                        <configuration>
                            <xtendAsPrimaryDebugSource>true</xtendAsPrimaryDebugSource>
                            <outputDirectory>${project.build.directory}/xtend-gen/main</outputDirectory>
                            <testOutputDirectory>${project.build.directory}/xtend-gen/test</testOutputDirectory>
                            <writeTraceFiles>true</writeTraceFiles>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
 
    <dependencies>
        <dependency>
            <groupId>org.eclipse.xtend</groupId>
            <artifactId>org.eclipse.xtend.lib</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
 
    </dependencies>
</project>
``` 