---
title:       "jenkins+docker+Linux+gitlab"
subtitle:    ""
description: ""
date:        2019-04-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/post-bg-2015.jpg"
tags:        ["jenkins", "gitlab","持续集成","服务部署"]
categories:  ["Tech" ]
---

## 

# Jenkins自动部署步骤

jenkins在mac中安装， docker里面分别安装了linux，作为服务器，拉取gitlab作为私人代码库。首先，我们应该要明白Jenkins的原理，他是这样的

a. 通过配置gitlab，让我们可以把代码存入到仓库。 

b. 通过配置jenkins和gitlab的关联，让jenkins可以通过SSH来获取gitlab的代码到jenkins的机子上。

c. 通过配置jenkins和linux服务器，通过SSH协议，可以把jenkins中打包好的包，传输到服务器上面。

d. 在jenkins中通过设置好上传好包后，执行服务器中固定的shell脚本，然后进行对包的运行和发布处理。 

Jenkins首先会通过Git将代码clone到本地，然后执行在Build中指定的  pom.xml文件和指定的命令，打包后，然后通过SSH协议，上传到服务器，然后在服务器上进行脚本发布。 

下面开始按照这个步骤来进行处理，主要说出需要注意的地方。 



## 第一步： docker拉取gitlab

需要注意如果直接进行安装，在打开的gitlab时候，发现：

![4-9-10](/img/4-9-10.png)

 这个地方不会是你的ip地址，会是你容器的id，这是很不方便的，所以我们在启动的时候需要加入hostname指定，如下：

```yaml
docker run -itd  -p 8443:443  -p 8081:80  -p 2222:2222   
                 -p 9090:9090  --name gitlab  --hostname 192.168.70.102  
                 --privileged=true   gitlab/gitlab-ce:latest
```

同时，我这边把SSH的默认端口修改为了2222，这个修改需要到对应的配置配置文件中修改，当然也可以不修改，我这边主要是怕端口冲突。

```yaml
$ docker run -d \
--hostname 10.10.1.70 \    # 指定容器域名,未知功能:创建镜像仓库的时候使用到
-p 8443:443 \ # 将容器内443端口映射到主机8443,提供https服务
-p 8080:80 \ # 将容器内80端口映射到主机8080,提供http服务
-p 2222:22 \ # 将容器内22端口映射到主机2222,提供ssh服务
-p 9090:9090 \  # 将容器内9090端口映射到主机9090,提供prometheus服务
--name gitlab \  # 指定容器名称

-v /docker_data/gitlab_home/config:/etc/gitlab \  # 将本地/home/gitlab/config挂载到容器内/etc/gitlab

-v /docker_data/gitlab_home/logs:/var/log/gitlab \ # 将本地/home/gitlab/logs挂载到容器内/var/log/gitlab

-v /docker_data/gitlab_home/data:/var/opt/gitlab \ # 将本地/home/gitlab/data挂载到容器内/var/opt/gitlab

gitlab/gitlab-ce:latest  # 镜像名称:版本
```

同时我如果按照上面的进行启动，发现只要加入data这个数劵，总是无法启动成功，查看这个本地文件夹权限也是够的，实在不知道是什么原因，怀疑还是权限问题导致，我的宿主机是mac，无奈我就用上面语句启动，只是一个测试。

**上面这一步，就已经把本地的代码和gitlab进行关联了起来**



## 第二步：  安装好jenkins

安装好jenkins，查看网上其他文章，就可以安装好，需要注意记得安装两个主要的插件，

 a.  Publish over SSH  用于上传包到linux服务器。（下面在说这个插件的配置）

b.  [Maven Integration plugin](https://wiki.jenkins.io/display/JENKINS/Maven+Project+Plugin) 这个插件用户jenkins创建时候，创建maven任务，对应maven项目处理。



## 第三步： jenkins,gitlab代码关联

![4-9-11](/img/4-9-11.png)

这一步就是两个进行了关联，需要注意的是，在填写SSH key 私钥的时候，这个地方填写的事私钥，gitlab那边填写的事公钥，同时，需要在jenkins的这台机子上生成公钥和私钥。

经过这个步骤，我们就把gitlab和jenkins进行了联系，通过这个jenkins来clone我们的提交到gitlab上面的代码。同时，我们需要配置jenkins的一些jdk和maven这样的基础配置，用于jenkins对代码的打包处理。 



## 第四步  gitlab,jenkins构建触发器

![4-9-12](/img/4-9-12.png)

图上画线的地方，需要在gitlab中，点击你需要关联的项目，》设置》集成

![4-9-13](/img/4-9-13.png)

在这个地方填入， 我这边选择的触发形式是，提交代码就自动构建一次。 填写成功后需要注意，在下放的 SSL verification 这个选项去掉， 不进行SSL验证， 然后点击  add  webhook 

![4-9-14](/img/4-9-14.png)

然后下面会生成一个 选项， 然后选择第一个 push events，模拟推送，如果在这个页面最上面显示

![4-9-15](/img/4-9-15.png)

显示200， 表示模拟推送成功，那个jenkins就在进行构建了。 

需要注意的如果报错了， 有可能是gitlab为了安全考虑， 不允许本地网络使用这个钩子，这个时候，我们就需要设置允许本地网络使用， 具体设置路径， 我们用root账号登录，然后点击 管理中心（就是一个小🔧的图标），然后进入设置，然后选择网络， 如下：

![4-9-16](/img/4-9-16.png)

这样就可以了。



## 第五步： jenkins和Linux关联

![4-9-17](/img/4-9-17.png)

这个地方就是打通了， jenkins和linux之间的通道， 在这里需要确定， 你远程的Linux已经开启了SSH service，你可以查看进程是否 ps -elf | grep ssh 是否有， 这个地方配置，可以有地方修改默认端口。 需要注意这个地方的remote directory 是你Linux的地址，和下面的图中的 Remote directory 是集成关系。 这里我直接使用了账号和密码登录，测试如果现实success表示和Linux关联成功， 这个地方也就是把你的jar包通过SSH发送到Linux服务器的地方。



##  第六步：jar发送到服务器，执行脚本

![4-9-18](/img/4-9-18.png)



图中

Source files：  这个地址的怎么写，可以查看你构建时候的日志， 查看如下两句话，然后把不同的找不出来，就是这个地址了， 应为这个原文件的地址是相对于jenkins home的路径的。 

09:51:48 构建中 在工作空间 /Users/Shared/Jenkins/Home/workspace/SpringBootJenkins 中

09:51:59 [INFO] Building jar: /Users/Shared/Jenkins/Home/workspace/SpringBootJenkins/target/springbootdemoweb-0.0.1-SNAPSHOT.jar

Remove prefix： 这个就是去除前面不需要的  

Remote directory： 这个是集成上面那个图的地址，相当于把这个jar包上传到linux的具体的位置。

Exec command：  这个就是执行shell脚本了。 这里都是相对于linux服务器写的，需要linux上面这个目录有这个shell脚本。

注意：不要加入 /  , 我这边加入 / 后，包总是不能上传成功。   



## 构建成功日志

```yaml
09:52:01 [JENKINS] Archiving /Users/Shared/Jenkins/Home/workspace/SpringBootJenkins/target/springbootdemoweb-0.0.1-SNAPSHOT.jar to com.example/springbootdemoweb/0.0.1-SNAPSHOT/springbootdemoweb-0.0.1-SNAPSHOT.jar
09:52:02 SSH: Connecting from host [localhost]
09:52:02 SSH: Connecting with configuration [192.168.70.102] ...
09:52:02 channel stopped
09:52:03 SSH: EXEC: STDOUT/STDERR from command [cd  /var/dev/springboot
09:52:03 sh ./springbootshell.sh restart] ...
09:52:03 >>> api PID = 1084 begin kill 1084 <<<
09:52:05 >>> springbootdemoweb-0.0.1-SNAPSHOT.jar is not running <<<
09:52:05 >>> start springbootdemoweb-0.0.1-SNAPSHOT.jar successed PID=1140 <<<
09:52:05 SSH: EXEC: completed after 2,425 ms
09:52:05 SSH: Disconnecting configuration [192.168.70.102] ...
09:52:05 SSH: Transferred 1 file(s)
```

这样，你每次提交代码，然后jenkins拉取代码，然后自动部署，发包到Linux，然后在自动启动shell脚本，这样就服务包就发布了。 