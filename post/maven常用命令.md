---
title:       "maven常用命令"
subtitle:    ""
description: "pom.xml标签解释，常用mvn命令"
date:        2019-04-28
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "maven"]
categories:  ["Tech" ]
---

[TOC]



**说明： 本学习主要对尚硅谷大数据研发部《maevn视频》记录，一下文字来源这个视频PPT**

**官网： <http://maven.apache.org/guides/introduction/introduction-to-repositories.html>**

# Maven简介

Maven 是 Apache 软件基金会组织维护的一款自动化构建工具，专注服务于 Java 平台的项目构建和 依赖管理。我们主要用来处理依赖的包和编译为jar或者war包，同时也可以配置自动化部署。同时maven也是用java来编写的，所以配置环境的时候需要配置JAVA_HOME



# 构建过程的几个主要环节

清理：删除以前的编译结果，为重新编译做好准备。

编译：将 Java 源程序编译为字节码文件。

测试：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。

报告：在每一次测试后以标准的格式记录和展示测试结果。

打包：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java 工程对应 jar 包，Web 工程对应 war 包。

安装：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。

部署：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行。



# 约定的目录结构

现在 JavaEE 开发领域普遍认同一个观点：约定>配置>编码。意思就是能用配置解决的问题就不编码， 能基于约定的就不进行配置。而 Maven 正是因为指定了特定文件保存的目录才能够对我们的 Java 工程进行 自动化构建。所以我们约定了maven项目的目录结构，这样的目录结构下面，maven才会知道去哪个地方编译java代码为class文件。

![5-1](/img/5-1.png)



# 仓库



## 分类

### 本地仓库

为当前本机电脑上的所有 Maven 工程服务。

### 远程仓库

私服：架设在当前局域网环境下，为当前局域网范围内的所有 Maven 工程服务。

![5-2](/img/5-2.png)

中央仓库：架设在 Internet 上，为全世界所有 Maven 工程服务。

中央仓库的镜像：架设在各个大洲，为中央仓库分担流量。减轻中央仓库的压力，同时更快的响应用户请求。

**注意：配置私服，在setting.xml文件中进行相关配置。**



## 仓库中的文件

1. Maven 的插件

2. 我们自己开发的项目的模块

3. 第三方框架或工具的 jar 包

不管是什么样的 jar 包，在仓库中都是按照坐标生成目录结构，所以可以通过统一的方式查询或依赖。

# Maven生命周期

简单理解，就是从 maven clearn， maven install， maven deploy 这些命令按顺序执行。 运行任何一个阶段的时候，它前面的所有阶段都会被运行，例如我们运行 mvn install 的时候，代码会 被编译，测试，打包。这就是 Maven 为什么能够自动执行构建过程的各个环节的原因。

# POM

## Maven的坐标

使用如下三个向量在 Maven 的仓库中唯一的确定一个 Maven 工程。我们在使用mvn install命令的时候，放入到maven仓库时候，是一个唯一的标识。

groupid：公司或组织的域名倒序+当前项目名称
artifactId：当前项目的模块名称
version：当前模块的版本

```xml
<groupId>com.atguigu.maven</groupId>
<artifactId>Hello</artifactId>
<version>0.0.1-SNAPSHOT</version>
```



## dependency 依赖管理

需要下载其他的包，直接通过依赖下载对应的包，这个包都是在本地仓库。

```xml
<dependency>
    <groupId>com.atguigu.maven</groupId>
    <artifactId>Hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <scope>compile</scope>
</dependency>
```



### 依赖的范围

compile: 编译时候需要
test: 测试时候
provided:  依赖应用容器本身就有的。 如：tomcat应用容器里面自带的一些jar包， 如jsp-api.jar等这样架包。

我们一般的时候，感觉直接用**<scope>compile</scope>**这个就好了。



### 依赖的排除

可以指定排除其中一些jar包

```xml
<dependency>
    <groupId>com.atguigu.maven</groupId>
    <artifactId>HelloFriend</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <type>jar</type>
    <scope>compile</scope>
    <exclusions>
      <exclusion>
          <groupId>commons-logging</groupId>
          <artifactId>commons-logging</artifactId>
      </exclusion>
    </exclusions>
</dependency>
```

### 

### 依赖的原则

就是有可能在引用包的时候，如果有多个相同的包的时候，按照下面的规矩来处理。

1.  这里“声明”的先后顺序指的是 dependency 标签配置的先后顺序。
2.  路径相同时先声明者优先。
3.  路径最短者优先





## properties 定义公共属性

```xml
#自定义一些公共属性，一般都放到父类的maven项目中
<properties>
         <atguigu.spring.version>4.1.1.RELEASE</atguigu.spring.version>
         <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<dependencies>
    <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-core</artifactId>
          #引用自定义公共属性值
          <version>${atguigu.spring.version}</version>
    <dependency>
<dependencies>    
```



## modules

将多个工程拆分为模块后，需要手动逐个安装到仓库后依赖才能够生效。修改源码后也需要逐个手动进 行 clean 操作。而使用了聚合之后就可以批量进行 Maven 工程的安装、清理工作。

```xml
<modules>
    <module>../Hello</module>
    <module>../HelloFriend</module>
    <module>../MakeFriends</module>
</modules>
```



## parent 

创建父工程和创建一般的 Java 工程操作一致，唯一需要注意的是：打包方式处要设置为 pom。

```xml
<parent>
    <groupId>com.atguigu.maven</groupId>
    <artifactId>Parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 指定从当前子工程的pom.xml文件出发，查找父工程的pom.xml的路径 -->
    <!-- 与不配置一样，默认就是寻找上级目录下得pom.xml -->
    <relativePath>../Parent/pom.xml</relativePath>
</parent>
```



## dependencyManagement

继承可以消除重复，那是不是就没有问题了？ 答案是存在问题，假设将来需要添加一个新的子模块account-util，该模块只是提供一些简单的帮助工具，不需要依赖spring、junit，那么继承后就依赖上了，有没有什么办法了？ 有，maven已经替我们想到了，那就是dependencyManagement元素。**dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；**只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.9</version>
        <scope>test</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

在子项目中重新指定需要的依赖，删除范围和版本号，这个时候才会去继承父类

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

 **注意：感觉没啥作用呀；其实作用还是挺大的，父POM使用dependencyManagement能够统一项目范围中依赖的版本，当依赖版本在父POM中声明后，子模块在使用依赖的时候就无须声明版本，也就不会发生多个子模块使用版本不一致的情况，帮助降低依赖冲突的几率。**



## import

单继承：maven的继承跟java一样，单继承，也就是说子model中只能出现一个parent标签，但是我们在springboot开发中，我们一般需要继承一个自己定义的父类pom类。这个时候我们可以用下面方式来继承springboot的相关类。

```xml
   <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

通过上面方式，就可以获取**spring-boot-dependencies-2.1.4.RELEASE.pom**文件中**dependencyManagement**配置的jar包依赖，查看这个pom文件，可以看到dependencyManagement里面都是springboot需要的包。

![5-3](/img/5-3.png)

**注意：import scope只能用在dependencyManagement里面**



## repositories

转载地址： <https://blog.csdn.net/zlgydx/article/details/51130627>

我们可以在POM中配置其它的远程仓库。这样做的原因有很多，比如你有一个**局域网的远程仓库**，使用该仓库能大大提高下载速度，继而提高构建速度，也有可能你依赖的一个jar在central中找不到，它只存在于某个特定的公共仓库，这样你也不得不添加那个远程仓库的配置。
这里我配置一个远程仓库指向中央仓库的中国镜像：

```xml
<repositories>
     <repository>
         <id>maven-net-cn</id>
         <name>Maven China Mirror</name>
         <url>http://maven.net.cn/content/groups/public/</url>
         <releases>
             <enabled>true</enabled>
         </releases>
         <snapshots>
             <enabled>false</enabled>
         </snapshots>
    </repository>
</repositories>
```

每个<repository>都有它唯一的ID，一个描述性的name，以及最重要的，远程仓库的url。此外，<releases><enabled>true</enabled></releases>告诉Maven可以从这个仓库下载releases版本的构件，而<snapshots><enabled>false</enabled></snapshots>告诉Maven不要从这个仓库下载snapshot版本的构件。禁止从公共仓库下载snapshot构件是推荐的做法，因为这些构件不稳定，且不受你控制，你应该避免使用。当然，如果你想使用局域网内组织内部的仓库，你可以激活snapshot的支持。



## build

转载地址： <https://blog.csdn.net/u010010606/article/details/79727438#>

Maven是通过pom.xml来执行任务的，其中的build标签描述了如何来编译及打包项目，而具体的**编译和打包**工作是通过build中配置的 plugin 来完成。当然plugin配置不是必须的，默认情况下，Maven 会绑定以下几个插件来完成基本操作。

![5-4](/img/5-4.png)

即在没有配置的情况下，执行mvn clean install时，maven会调用默认的plugin来完成编译打包操作。

如果有需要可以针对各个 plugin 进行特殊配置，需要在pom.xml中的<plugins>标签中显示指定 plugin 和 属性配置。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.3</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```



### 作用

转载地址： <https://blog.csdn.net/cuixianlong/article/details/78961218>

#### 定义（或声明）项目的目录结构 

```xml
<build>
    <!-- 指定打包文件名称（可用于除去jar文件版本号） -->
    <finalName>maven-build-demo</finalName>
    <!-- 指定过滤资源目录 -->
    <filters>
      <filter>${basedir}/profiles/test/test.properties</filter>
    </filters>
    <!-- 项目资源清单（可以配置多个项目资源） -->
    <resources>
      <!-- 项目资源  -->
      <resource>
        <!-- 资源目录（编译时会将指定资源目录中的内容复制到输出目录） -->
        <directory>src/main/resources</directory>
        <!-- 包含内容（编译时仅复制指定包含的内容） -->
        <includes>
          <include>*.properties</include>
          <include>*.xml</include>
          <include>*.json</include>
        </includes>
        <!-- 排除内容（编译时不复制指定排除的内容） -->
        <excludes>
          <exclude>*.txt</exclude>
        </excludes>
        <!-- 输出目录（默认为${build.outputDirectory}，即target/classes） -->
        <targetPath>${build.outputDirectory}</targetPath>
        <!-- 是否开启资源过滤（需要引入maven-resources-plugin插件）
         |   true：将用过滤资源（filters标签）中的内容 替换 资源中相应的占位符（${Xxxx}）内容
         |   false：不做过滤替换操作
         -->
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
```



#### 使用maven的插件（maven plugins）

转载地址： <https://blog.csdn.net/cuixianlong/article/details/78961218>

```xml
 <build>
    <!-- 项目插件清单（需要用到什么插件，就添加什么插件） -->
    <plugins>
      <!-- 项目插件 -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <!-- 配置参数 -->
        <configuration>
          <!-- 设为可执行程序 -->
          <executable>true</executable>
        </configuration>
        <executions>
          <!-- 执行操作 -->
          <execution>
            <!-- 执行目标 -->
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      <!-- 项目插件 -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-resources-plugin</artifactId>
        <!-- 执行操作清单 -->
        <executions>
          <!-- 执行操作（示例说明：将项目的父级目录下的profiles下的指定文件复制到指定输出目录） -->
          <execution>
            <!-- 标识ID -->
            <id>copy-resources</id>
            <!-- 执行阶段 -->
            <phase>validate</phase>
            <!-- 执行目标（执行的操作） -->
            <goals>
              <goal>copy-resources</goal>
            </goals>
            <!-- 相关参数 -->
            <configuration>
              <outputDirectory>${basedir}/target/classes/</outputDirectory>
              <resources>
                <resource>
                  <directory>${basedir}/../profiles</directory>
                  <filtering>false</filtering>
                  <includes>
                    <include>**/*.xml</include>
                    <include>*.json</include>
                  </includes>
                </resource>
              </resources>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

 

## profiles

**转载地址： <https://blog.csdn.net/mhmyqn/article/details/24501281**>

**注意：这个地方没有实际验证**

profile可以让我们定义一系列的配置信息，然后指定其激活条件。这样我们就可以定义多个profile，然后每个profile对应不同的激活条件和配置信息，从而达到不同环境使用不同配置信息的效果。比如说，我们可以通过profile定义在jdk1.5以上使用一套配置信息，在jdk1.5以下使用另外一套配置信息；或者有时候我们可以通过操作系统的不同来使用不同的配置信息，比如windows下是一套信息，linux下又是另外一套信息，等等。



### 不同环境pom配置

```xml
<profiles>
	<profile>
		<!-- 本地开发环境 -->
		<id>development</id>
		<properties>
			<profiles.active>development</profiles.active>
			<deploy.url>http://host:port/manager/text</deploy.url>
		</properties>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
	</profile>
	<profile>
		<!-- 测试环境 -->
		<id>test</id>
		<properties>
			<profiles.active>test</profiles.active>
			<deploy.url>http://host:port/manager/text</deploy.url>
		</properties>
	</profile>
	<profile>
		<!-- 生产环境 -->
		<id>production</id>
		<properties>
			<profiles.active>production</profiles.active>
			<deploy.url>http://host:port/manager/text</deploy.url>
		</properties>
	</profile>
```

这里定义了三个环境，分别是development（开发环境）、test（测试环境）、production（生产环境），其中开发环境是默认激活的（activeByDefault为true），这样如果在不指定profile时默认是开发环境。

同时每个profile还定义了两个属性，其中profiles.active表示被激活的profile的名称，deploy.url表示发布服务器的地址。我们需要在下面使用到这两个属性。

另外host和port分别是发布服务器的主机地址和端口号。

**注意：http://host:port/manager/text，这个地址就是Tomcat一个默认的页面。只要修改ip和端口就好。**



### **项目文件目录**

![5-5](/img/5-5.png)

### 资源插件pom配置

maven编译时候目录会按下面修改，只会加载激活的文件目录。

```xml
<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<!-- 资源根目录排除各环境的配置，使用单独的资源目录来指定 -->
				<excludes>
					<exclude>test/*</exclude>
					<exclude>production/*</exclude>
					<exclude>development/*</exclude>
				</excludes>
			</resource>
			<resource>
				<directory>src/main/resources/</directory>
				<includes>
				     <include>${profiles.active}/*</include>
				</includes>
			</resource>
		</resources>
	</build>
```



### 配置tomcat-maven-plugin插件

**构建**

```xml
<plugin>
      <groupId>org.apache.tomcat.maven</groupId>
      <artifactId>tomcat7-maven-plugin</artifactId>
      <version>2.2</version>
      <configuration>
            <url>${deploy.url}</url>
            <username>admin</username>
            <password>admin</password>
            <path>/appcontext</path>
            <update>true</update>
      </configuration>
</plugin>
```

其中发布的<url>节点就是在前面profile中配置的deploy.url属性，这样不同的环境就指定了不同的发布地址。<server>和<path>节点分别是发布服务器的用户配置的id以及应用的context名称。



### 部署

通过在运行maven命令时指定不同的profile即可构建不同环境需要的war包或发布到不同的环境了 。如：

```properties
#即构建出生产环境需要的war包
mvn clean package -Pproduction 

#部署项目到远程服务器
mvn clean install tomcat7:deploy
```

**注意：远程服务器也需要对应的进行配置。**

自动部署显示成功，war包也上传成功，但是war不自动解压自动部署。

如果你在tomcat的server.xml中通过设置<Context>标签来部署相同名称的项目的话，maven发布到该服务器的war不会被自动解压，部署，更新，需要去掉server.xml中该项目的<Context>标签。



## deploy

转载地址： <https://blog.csdn.net/woshixuye/article/details/8133050>

**注意：这个地方没有实际验证，只是大致思路**

mvn:deploy在整合或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享。**修改settings.xml的<servers></servers>**

### 上传到nexus

因为nexus是需要登陆操作，当然可以通过配置免登陆

```xml
<server>   
     <id>thirdparty</id>   
     <username>admin</username>
     <password>admin123</password>   
</server>
使用命令： maven deploy
```



### 上传包到远程服务器

```xml
<server>   
    <id>tomcat7</id>   
    <username>admin</username>   
    <password>admin</password>   
</server>
#如果使用tomcat7-maven-plugin这个插件，使用下面命令
mvn clean install tomcat7:deploy
```

**注意：** 如果进行deploy时返回Return code is: 401错误，则需要进行用户验证或者你已经验证的信息有误。



# 常用maven命令

**转载地址：<https://www.cnblogs.com/phoebus0501/archive/2011/05/10/2042511.html>**



## 运行命令条件

cd到需要打包项目的pom.xml目录下才运行



## 创建Maven的普通java项目

```properties
mvn archetype:create 
   -DgroupId=packageName 
   -DartifactId=projectName  
```



## 创建Maven的Web项目

```properties
 mvn archetype:create 
    -DgroupId=packageName    
    -DartifactId=webappName 
    -DarchetypeArtifactId=maven-archetype-webapp  
```

  

## 其他一些命令

```properties
mvn package     根据pom.xml打成war或jar,生成target目录，编译、测试代码，生成测试报告，生成jar/war文件 
mvn install     把jar放在本地Repository中
mvn clean       清除产生的项目,清除target目录
mvn jar:jar     只打jar包
mvn war:war     只打war包
mvn idea:idea   生成idea项目
mvn eclipse:eclipse  生成eclipse项目 


mvn -Dtest package  打包但不测试
完整命令为  mvn -Dmaven.test.skip=true package

mvn clean install -Dmaven.test.skip=true -Dmaven.javadoc.skip=true

一般使用情况是这样
1. 通过git clone代码到本地
2. 执行mvn idea:idea生成idea项目文件
3. 然后导入到idea即可

```



## package，install，deploy区别



package命令完成了项目编译、单元测试、打包功能，但没有把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库



install命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库，但没有布署到远程maven私服仓库



deploy命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库　



  







