---
title: Docker知识点03
date: 2024-05-24 16:29:04
tags:
  - docker compose
  - overlay
  - hadoop
  - spark
categories:
  - 云原生
  - 基础设施
cover: 'https://s2.loli.net/2024/02/20/fnPqBZJia8KLhEW.png'
---

# 导读

- 编译生成应用的jar包
- 构建应用的docker镜像
- 编写自动化脚本
- 编写docker-compose.yaml

# 编译jar包

使用maven打包,pom文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>fateboard</groupId>
    <artifactId>fateboard</artifactId>
    <version>2.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
        <relativePath/>
    </parent>

    <packaging>jar</packaging>

    <name>fateboard</name>
    <description>fateboard</description>

    <properties>
        <druid.version>1.1.10</druid.version>
<!--        <druid.version>1.1.10</druid.version>-->
        <lombok.version>1.18.24</lombok.version>
        <jasypt.version>2.0.0</jasypt.version>

        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <log4j2.version>2.17.2</log4j2.version>
        <spring.cloud-version>2021.0.3</spring.cloud-version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.27</version>
            <scope>compile</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>

        <dependency>
            <groupId>ch.ethz.ganymed</groupId>
            <artifactId>ganymed-ssh2</artifactId>
            <version>262</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.5</version>
        </dependency>

        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.11.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.83</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>32.0.1-android</version>
        </dependency>

        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.13</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.jcraft/jsch -->
        <dependency>
            <groupId>com.jcraft</groupId>
            <artifactId>jsch</artifactId>
            <version>0.1.55</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
        </dependency>

        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.4.4</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>ma.glasnost.orika</groupId>
            <artifactId>orika-core</artifactId>
            <version>1.5.4</version>
        </dependency>

        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.10.12</version>
        </dependency>

        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.1.Final</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.13.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>

        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.7.2</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-reload4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
            <version>2.9.3</version>
        </dependency>

    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-api</artifactId>
                <version>${log4j2.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-core</artifactId>
                <version>${log4j2.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-slf4j-impl</artifactId>
                <version>${log4j2.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-jul</artifactId>
                <version>${log4j2.version}</version>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>jul-to-slf4j</artifactId>
                <version>1.7.32</version>
            </dependency>
            <dependency>
                <groupId>org.yaml</groupId>
                <artifactId>snakeyaml</artifactId>
                <version>2.0</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>2.14.0-rc3</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.*</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                </includes>
            </resource>

            <resource>
                <directory>src/main/lib</directory>
                <targetPath>BOOT-INF/lib/</targetPath>
                <includes>
                    <include>**/*.jar</include>
                </includes>
            </resource>
        </resources>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>com.github.eirslett</groupId>
                <artifactId>frontend-maven-plugin</artifactId>
                <version>1.9.1</version>
                <executions>
                    <!-- 检查是否安装node npm -->
                    <execution>
                        <id>install node and npm</id>
                        <goals>
                            <goal>install-node-and-npm</goal>
                        </goals>
                        <phase>generate-resources</phase>
                    </execution>
                    <!-- npm install -->
                    <execution>
                        <id>npm install</id>
                        <goals>
                            <goal>npm</goal>
                        </goals>
                        <phase>generate-resources</phase>
                        <configuration>
                            <arguments>install</arguments>
                        </configuration>
                    </execution>
                    <!-- build -->
                    <execution>
                        <id>npm run build</id>
                        <goals>
                            <goal>npm</goal>
                        </goals>
                        <phase>generate-resources</phase>
                        <configuration>
                            <arguments>run build</arguments>
                        </configuration>
                    </execution>
                </executions>
                <configuration>
                    <nodeVersion>v16.15.1</nodeVersion>
                    <npmVersion>8.11.0</npmVersion>
                    <!-- node安装路径 -->
                    <installDirectory>dist</installDirectory>
                    <!-- 前端代码路径 -->
                    <workingDirectory>resources-front-end</workingDirectory>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution> <!-- 复制配置文件 -->
                        <id>copy-resources</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <resources>
                                <resource>
                                    <directory>resources-front-end/packages/fate-board/dist</directory>
                                    <includes>
                                        <include>*</include>
                                        <include>*/**</include>
                                    </includes>
                                </resource>
                            </resources>
                            <outputDirectory>${project.build.directory}/classes/static</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.1</version>
                <configuration combine.self="override">
                    <includes>
                        <include>**/*</include>
                    </includes>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.2.1</version>
                <configuration>
                    <descriptors>
                        <descriptor>${project.basedir}/package.xml</descriptor>
                    </descriptors>
                    <archive>
                        <manifest>
                            <mainClass>org.fedai.fate.board.bootstrap.Bootstrap</mainClass>
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
</project>

```

编译打包命令，在pom.xml同级目录下执行

```shell
mvn clean package -Dmaven.test.skip=true
```

# 构建镜像

在应用的源码根目录下创建一个目录，如`deploy`:

```shell
mkdir deploy
cd deploy
```

创建所需要目录，拷贝所需的文件，如jar包，配置文件等需要打包到镜像中的文件夹/文件

```shell
mkdir conf ssh
cp -r ../target/fateboard-2.1.0.jar ./
cp -r ../target/lib ./
cp -r ../src/main/resources/application.properties ./conf/
cp -r ../src/main/resources/log4j2.xml ./conf/
cp -r ../src/main/resources/ssh.properties ./ssh/
```

编写Dockerfile

```shell
FROM mcr.microsoft.com/java/jre:8u192-zulu-alpine as builder

MAINTAINER maodm@yunphant.com

FROM mcr.microsoft.com/java/jre:8u192-zulu-alpine
RUN apk add tzdata

ENV TZ=Asia/Shanghai  HOME=/yunphant JAVA_OPTS="-Xmx2g -Xms1g" APP_NAME="fateboard" MAIN_CLASS="org.fedai.fate.board.bootstrap.Bootstrap"
# 日志+应用配置
ENV APP_CONF=$HOME/$APP_NAME/conf/application.properties LOG_CONFIG_PATH=$HOME/$APP_NAME/conf/log4j2.xml

WORKDIR $HOME
copy conf $HOME/$APP_NAME/conf
copy lib $HOME/$APP_NAME/lib
copy ssh $HOME/$APP_NAME/ssh
copy fateboard-2.1.0.jar $HOME/$APP_NAME

EXPOSE 8080

RUN echo "build done"
CMD java -Dspring.config.location=$HOME/$APP_NAME/conf/application.properties  -Dssh_config_file=$HOME/$APP_NAME/ssh -Xmx2g -Xms1g -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError -jar fateboard.jar
```

编译docker镜像：

```shell
docker build -t yunpcds/fateboard:2.1.0-release .
```

>  如果执行成功，就会在本地docker仓库中生成了一个docker镜像。

# 编写自动化脚本

上述过程使用一个脚本完成：

```shell
#!/bin/bash

APP_NAME=fateboard
APP_VERSION=2.1.0
DOCKER_IMAGE_NAME=yunpcds/fateboard
DOCKER_IMAGE_TAG=2.1.0-release

# 创建conf 目录
mkdir -p conf
# 创建ssh 目录，用于存放ssh公钥文件
mkdir -p ssh

# 进入上一层目录
cd ..

# 编译 fate-board生成 jar包
mvn clean package -Dmaven.test.skip=true

# 回到deploy 目录
cd deploy

# 拷贝所需要的文件
cp -r ../target/${APP_NAME}-${APP_VERSION}.jar ./
cp -r ../target/lib ./
cp -r ../src/main/resources/application.properties ./conf/
cp -r ../src/main/resources/log4j2.xml ./conf/
cp -r ../src/main/resources/ssh.properties ./ssh/

docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .

```

# 启动容器

使用docker-compose会相应简便些，编写一个docker-compose.yaml文件

```yaml
version: '3.8'

services:
  fateboard:
    image: yunpcds/fateboard:2.1.0-release
    ports:
      - "8090:8080"
    volumes:
      - ./conf:$APP_HOME/$APP_NAME/conf
      - ./logs:$APP_HOME/logs
      - /etc/localtime:/etc/localtime:ro
    restart: always
    command: [ "sh", "-c", "java -Dspring.config.location=/yunphant/fateboard/conf/application.properties -Dssh_config_file=/yunphant/fateboard/ssh/ -Xmx2048m -Xms2048m -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError -cp /yunphant/fateboard/lib/*:/yunphant/fateboard/fateboard-2.1.0.jar org.fedai.fate.board.bootstrap.Bootstrap" ]

```

因为在docker-compose.yaml中用到了`$APP_HOME` 和 `$APP_NAME` 这两个环境变量，这是可以在宿主机的环境变量中设置的，也可以创建一个`.env`文件，在文件中定义需要用到的环境变量：

```shell
APP_HOME=/yunphant
APP_NAME=fateboard
```

启动容器,在docker-compose.yaml 同级目录下执行:

```shell
docker-compose up -d
```

