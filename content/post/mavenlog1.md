---
title: "maven踩坑"
date: 2020-02-24T13:23:51+08:00
draft: false
tags: ["java"]
---

mavne配置：
```xml
 <mirrors>
     <mirror>
            <id>nexus-aliyun</id>
            <mirrorOf>central</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
  </mirrors>
   <profiles>
    <profile>   
    <id>jdk1.8</id>    
    <activation>   
        <activeByDefault>true</activeByDefault>    
        <jdk>1.8</jdk>   
    </activation>    
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>    
        <maven.compiler.target>1.8</maven.compiler.target>    
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>   
    </properties>   
</profile> 
  </profiles>
```

项目配置：
```xml
<build>
    <plugins>
        <!-- 使用 mvn assembly:assembly 打包 -->
        <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass>com.github.lwxntm.Main</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
    </plugins>
</build>
```
