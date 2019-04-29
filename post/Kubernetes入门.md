---
title:       "Kubernetes入门"
subtitle:    ""
description: ""
date:        2019-03-04
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-4-25-istio-auto-injection-with-webhook/lion.jpg"
tags:        ["k8s核心主件","yaml配置","pod,deployment,service","docker"]
categories:  ["Tech" ]
---



[TOC]



注明：资源多来源于网上资源，此文章只是记录笔记整理，点击标题链接查看细节原文。

### 路由

转载地址：https://www.zhihu.com/question/46767015/answer/103018297

 路由——url的分层解析。

 第一层  解析到服务器目标机器。这个通常是域名或ip。

 第二层  解析到服务器的特定资源文件。这个通常是pathinfo。

 第三层  解析特定资源的特定状态。包含在pathinfo参数中。

 主要是服务器，资源文件，特定状态定位。

 就是在找到目的地。
 路由器里根据目标IP，找到中间需要经过的路由器路径。
 Web里根据网址找到能处理这个URL的程序或模块。



### DNS 

是一种组织成域层次结构的计算机和网络服务命名系统。DNS命名用于TCP/IP网络，主要使用在Internet上。当网络上的一台客户机需要访问某服务器上的资源时，客户机的用户只需要在浏览器中的地址文本框中输入该服务器的网址、就可以与该服务器进行连接，计算机的硬件只能识别IP地址，而不能识别其他类型的地址。这样在用户容易记忆的地址和计算机能够识别的地址之间就需要进行转换，DNS服务器便充当了这个地址解析的角色。现在找本机上面host里面的对应，如果没有就到网上寻找。



### mac安装kubernetes

转载地址：https://yq.aliyun.com/articles/508460

```shell
git clone https://github.com/AliyunContainerService/k8s-for-docker-desktop(下载master分支)
cd k8s-for-docker-desktop
./load_images.sh
然后直接启动docker就可以直接加载了。


kubectl config use-context docker-for-desktop
验证 Kubernetes 安装
kubectl cluster-info
kubectl get nodes

kubectl create -f kubernetes-dashboard.yaml(这个需要在那个master分支的文件夹中执行)
kubectl proxy
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default
```



### 搭建kubernetes-dashboard

转载地址：https://my.oschina.net/wubiaowpBlogShare/blog/2222564

在kubernetes-dashboard.yaml父级文件夹下创建account.yaml文件用于访问kubernetes-dashboard，添加如下配置

```shell
# Create Service AccountapiVersion: v1 
kind: ServiceAccount 
metadata:   
  name: admin-user   
  namespace: kube-system 
---# Create ClusterRoleBindingapiVersion: rbac.authorization.k8s.io/v1beta1 
kind: ClusterRoleBinding 
metadata:   
  name: admin-user 
roleRef:   
  apiGroup: rbac.authorization.k8s.io   
  kind: ClusterRole   
  name: cluster-admin 
subjects: 
  - kind: ServiceAccount   
  name: admin-user   
  namespace: kube-system
```

获取登陆令牌kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')， 获取输出的token粘贴复制到kubernetes-dashboard登陆页面获取授权。



### 什么是kubernetes

转载地址：http://www.dockone.io/article/932

Kubernetes（k8s）是自动化容器操作的开源平台，这些操作包括部署，调度和节点集群间扩展。如果你曾经用过Docker容器技术部署容器，那么可以将Docker看成Kubernetes内部使用的低级别组件。Kubernetes不仅仅支持Docker，还支持Rocket，这是另一种容器技术。

使用Kubernetes可以：

- 自动化容器的部署和复制 

- 随时扩展或收缩容器规模 

- 将容器组织成组，并且提供容器间的负载均衡 

- 很容易地升级应用程序容器的新版本 

- 提供容器弹性，如果容器失效就替换它，等等...

  

### 核心主件 

原文地址：https://www.cnblogs.com/along21/p/10297756.html

#### Pod

​     Pod的设计理念是支持多个容器在一个Pod中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。 

​     Pod是K8s集群中所有业务类型的基础，可以看作运行在K8s集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。目前K8s中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为Deployment、Job、DaemonSet和PetSe



#### 部署(Deployment)

​      部署表示用户对K8s集群的一次更新操作。**部署是一个比RS应用模式更广的API对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务**。滚动升级一个服务，实际是创建一个新的RS，然后逐渐将新RS中副本数增加到理想状态，将旧RS中的副本数减小到0的复合操作。感觉就是管理pod的工具。 



#### 服务（Service）

​      一个Pod只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的IP启动一个新的Pod，因此不能以确定的IP和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。在K8s集群中，客户端需要访问的服务就是Service对象。**每个Service会对应一个集群内部有效的虚拟IP，集群内部通过虚拟IP访问一个服务。在K8s集群中微服务的负载均衡是由Kube-proxy实现的**。Kube-proxy是K8s集群内部的负载均衡器。它是一个分布式代理服务器，在K8s的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的Kube-proxy就越多，高可用节点也随之增多。    

​      相比我们理解的nginx中的负载均衡的时候， 对外配置的一个入口。 这个简单理解也是可以这样。 

​      service： *如果Pods是短暂的，那么重启时IP地址可能会改变，怎么才能从前端容器正确可靠地指向后台容器呢？

​      会为Service创建一个本地集群的DNS入口，因此前端Pod只需要DNS查找主机名为 ‘backend-service’，就能够解析出前端应用程序可用的IP地址。   

​     现在前端已经得到了后台服务的IP地址，但是它应该访问2个后台Pod的哪一个呢？Service在这2个后台Pod之间提供透明的负载均衡，会将请求分发给其中的任意一个，通过每个Node上运行的代理（kube-proxy）完成。

​      RC、RS和Deployment只是保证了支撑服务的微服务Pod的数量，但是没有解决如何访问这些服务的问题。一个Pod只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的IP启动一个新的Pod，因此不能以确定的IP和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。在K8s集群中，客户端需要访问的服务就是Service对象。每个Service会对应一个集群内部有效的虚拟IP，集群内部通过虚拟IP访问一个服务。在K8s集群中微服务的负载均衡是由Kube-proxy实现的。Kube-proxy是K8s集群内部的负载均衡器。它是一个分布式代理服务器，在K8s的每个节点上都有一个；



#### 任务（Job）

　　**Job是K8s用来控制批处理型任务的API对象**。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job管理的Pod根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的spec.completions策略而不同：单Pod型任务有一个Pod成功就标志完成；定数成功型任务保证有N个任务全部成功；工作队列型任务根据应用确认的全局成功而标志成功。



#### 后台支撑服务集（DaemonSet）

　　长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的Pod，有些节点上又没有这类Pod运行；而后台支撑型服务的核心关注点在K8s集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类Pod运行。节点可能是所有集群节点也可能是通过nodeSelector选定的一些特定节点。**典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持K8s集群运行的服务**。



#### 有状态服务集（PetSet）

​        RC和RS->Replication Controller,保证pod的高可用，保证pod的数量。

​        RC和RS主要是控制提供无状态服务的，其所控制的Pod的名字是随机设置的，一个Pod出故障了就被丢弃掉，在另一个地方重启一个新的Pod，名字变了、名字和启动在哪儿都不重要，重要的只是Pod总数；而PetSet是用来控制有状态服务，PetSet中的每个Pod的名字都是事先确定的，不能更改。

​       对于RC和RS中的Pod，一般不挂载存储或者挂载共享存储，保存的是所有Pod共享的状态，适合于PetSet的业务包括数据库服务MySQL和PostgreSQL，集群化管理服务Zookeeper、etcd等有状态服务。运维人员需要不断地维护它。  



#### 存储卷（Volume）

​         K8s集群中的存储卷跟Docker的存储卷有些类似，**只不过Docker的存储卷作用范围为一个容器，而K8s的存储卷的生命周期和作用范围是一个Pod**。每个Pod中声明的存储卷由Pod中的所有容器共享。持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）  **PV和PVC使得K8s集群具备了存储的逻辑抽象能力**，使得在配置Pod的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给PV的配置者，即集群的管理者。相当于提出了一个接口，和JDBC的接口一样，后面不管是mysql还是oracle



#### 密钥对象（Secret）

​        **Secret是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象**。使用Secret的好处是可以避免把敏感信息明文写在配置文件里。



#### 用户帐户（User Account）和服务帐户（Service Account）

​       顾名思义，用户帐户为人提供账户标识，而服务账户为计算机进程和K8s集群中运行的Pod提供账户标识。用户帐户和服务帐户的一个区别是作用范围；用户帐户对应的是人的身份，人的身份与服务的namespace无关，所以用户账户是跨namespace的；而服务帐户对应的是一个运行中程序的身份，与特定namespace是相关的。



#### 名称空间（Namespace）

​      **名称空间为K8s集群提供虚拟的隔离作用**，K8s集群初始有两个名字空间，分别是**默认名称空间default和系统名称空间kube-system**，除此以外，管理员可以可以创建新的名字空间满足需要。



#### RBAC访问授权

​       RBAC主要是引入了角色（Role）和角色绑定（RoleBinding）的抽象概念



#### 基本联系流程

1.    创建Deployment文件， 里面配置好需要下载的镜像，运行后，生成pod。这个Deployment里面就包含了对应pods
2.    创建Deployment时候，有对应的tag

​         创建nginx.ymal

```yaml
apiVersion: extensions/v1beta1                  # K8S对应的API版本
kind: Deployment                # 对应的类型
metadata:
  name: nginx-dev
  labels:
    name: nginx-dev
spec:
  replicas: 2                    # 镜像副本数量
  template:
    metadata:
      labels:                    # 容器的标签 可和service关联
        app: nginx
    spec:
      containers:
        - name: nginx            # 容器名和镜像
          image: nginx
          imagePullPolicy: Always
          securityContext:
               privileged: true   # 特权进入 
```

​         通过命令  kubectl create -f nginx.yaml   创建pods，查看效果 

```yaml
kubectl get pods

NAME                         READY   STATUS    RESTARTS   AGE
nginx-dev-57b967f787-dr58z   1/1     Running   0          4h43m
nginx-dev-57b967f787-vdc54   1/1     Running   0          4h43m

kubectl get deployment

NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-dev   2/2     2            2           4h43m

上面分别看到了pods 和 deployment，  可以查看2个nginx容器分别分配的虚拟ip

kubectl get pods -o wide 

NAME                         READY   STATUS    RESTARTS   AGE     IP          NODE              
nginx-dev-57b967f787-dr58z   1/1     Running   0          4h44m   10.1.0.70   docker-desktop    
nginx-dev-57b967f787-vdc54   1/1     Running   0          4h44m   10.1.0.69   docker-desktop

```



3. ​    启动service，用上面的Deployment中的tag，关联起来，创建服务。

​           

```yaml
创建：kubectl expose deployment nginx-dev --name=nginx-service --port=80

查看
kubectl get svc nginx-service

NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.108.53.71   <none>        80:32632/TCP   4h45m

```



4.  这个时候，node里面的容器，可以通过service创建出来的CLUSTER-IP来进行互相通信。 

   

5.  外网直接访问容器里面的容器，需要修改这个服务的 type: NodePort 为这个就可以了。 

```
kubectl edit svc nginx-service

执行后，修改这个文件的  type: NodePort 为这个。

注意：  在 kubectl edit svc nginx-service 这个命令的时候， 如果用sublime软件打开，发现，还没有修改就关闭了，无法修改的问题， 执行这个命令的时候，我们可以指定用哪一个软件打开，如下：

 KUBE_EDITOR="vi"  kubectl edit svc nginx-service
 
同时， 如果是mac系统做了修改， 还是无法直接用容器ip:32632这个来访问， 但是可以通过mac的主机:32632来进行访问， 感觉这个是mac系统问题， 在docker里面的容器也是一样，直接用本机ip访问就OK。 如：

http://localhost:32632  网上Linux就是可以  10.1.0.70:32632 直接访问到容器里面了。
同时容器里面的也可以直接curl <mac主机ip>:32632可以访问外面。
```

  

### K8S架构图

(转载地址：http://www.dockone.io/article/932)

 ![](http://dockone.io/uploads/article/20151230/d56441427680948fb56a00af57bda690.png)

![](http://dockone.io/uploads/article/20151230/5e2bad1a25e33e2d155da81da1d3a54b.gif)



### 常用k8的常用命令

(转载地址：https://www.cnblogs.com/along21/p/10304362.html)

#### 查看集群信息

​      kubectl cluster-info  

#### 运行一个nginx的pod

​      kubectl run nginx-deploy --image=nginx:1.14-alpine --port=80 --replicas=1 

#### 查看pod,service,deployment信息 

​      kubectl get pods/svc/deployment 

#### 删除

​      kubectl delete ([-f FILENAME] | TYPE [(NAME | -l label | --all)]) 

​      常用如：kubectl delete pods nginx-deploy-5b595999-6kw54

#### 资源暴露为service

​      kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]

​     常用如：kubectl expose deployment nginx-deploy --name=nginx --port=80 --target-port=80 --protocol=TCP

​                --port：暴露在service上的端口
​                --target-port：容器中的端口

#### scale动态扩容/缩容

​     扩容或缩容 Deployment、ReplicaSet、Replication Controller或 Job 中Pod数量

​    kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME)

​    常用如：kubectl scale --replicas=5 deployment myapp

#### set动态升级版本，配置应用资源

​     配置应用资源。

​     set：使用这些命令能帮你更改现有应用资源一些信息。子命令：image、resources、selector、subject）set image

​     set image：更新现有的资源对象的容器镜像。可使用资源对象包括（不区分大小写）：pod (po)、replicationcontroller (rc)、deployment (deploy)、  daemonset (ds)、job、replicaset (rs)

​     $ kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N

​     版本升级：kubectl set image deployment myapp myapp=ikubernetes/myapp:v2

#### rollout undo 回滚版本，暂停，恢复

​    （1）rollout

​            对资源进行管理：可用资源包括：deployments、daemonsets，子命令：history（查看历史版本）、pause（暂停资源）、resume（恢复暂停资源）、status（查看资源状态）、undo（回滚版本）

   （2）rollout undo

​           回滚pod到之前的版本。

​       kubectl rollout undo (TYPE NAME | TYPE/NAME) [flags]

​       回滚    kubectl rollout undo deployment myapp

#### edit编辑修改

​     使用默认编辑器，编辑服务器上定义的资源， KUBE_EDITOR指定编辑器打开

​     $ kubectl edit (RESOURCE``/NAME` `| -f FILENAME)，

​     常用：  kubectl edit svc myapp

#### label标签

   更新（增加、修改或删除）资源上的 label（标签）。

   kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]

   运行一个标签为app=nginx的pod

   kubectl run nginx --image=nginx:1.14-alpine --replicas=1 --labels="app=nginx"  

#### 标签选择器

  等值关系：=，==，!=  

​    kubectl get pods -l release=stable --show-labels

 集合关系

-  KEY in (VALUE1,VALUE2,...)：集合中存在此健
-  KEY notin (VALUE1,VALUE2,...)：集合中不存在此健
-  KEY：存在此健
-  !KEY：不存在此健

   kubectl get pods -l "release in (stable,beta,alpha)"

### kubernetes资源配置

(转载地址：https://www.cnblogs.com/along21/p/10313341.html)



#### 主流资源的配置清单

```yml
api server: group/version        #查询当前支持哪些api server: kubectil api-version
kind: #资源类别
metadata: #元数据
      name: #名字
      namespace: #命名空间
      labels: #标签
      annotation: #资源注解
      selfLink: #每个资源的引用
 spec: #期望的状态 ，期望资源应该用于什么特性
 status: #当前状态，本字段由k8s集群维护，用户不需要自己处理
 
 #使用  kubectil explain,可以查看每个资源如何配置
 #如：  kubectil explain pod 
 #还可以层级递进  kubectil explain pod.spec 
```

​               

#### pod资源

##### pod格式和常用命令

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  #labels: {app:myapp, tier:frontend} #映射可以写为{}形式;
  labels: #也可以在下边分级写
    app: myapp
    tier: frontend
  annotations: #资源注解
    along.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp #容器名称 
    image: ikubernetes/myapp:v1 #镜像
    ports: #从容器中公开的端口列表
    - name: http #暴露这个端口的名称
      containerPort: 80 #Pod中服务的端口号
      #hostIP：暴露绑定在主机哪个IP上
      #hostPort：暴露在主机的端口号
    - name: https
      containerPort: 443
  - name: busybox ##容器名称
    image: busybox:latest #镜像
    imagePullPolicy: IfNotPresent #下载镜像规则，镜像时latest标签，默认是Always；否则默认IfNotPresen
    #Always总是镜像，Never不下载镜像，IfNotPresent本地有则不下载，k8s就会从本地拉取镜像了
    
    #command:  执行命令， ["/bin/sh","-c","sleep 3600"]  #列表可以写为[]形式;   
    command: #也可以在下边分级写,要加-
    - "/bin/sh"
    - "-c"
    - "sleep 3600"
  nodeSelector: #节点标签选择器
    disktype: ssd
 
 #查看现在pod的yaml格式   kubectl get pod nginx-dev-57b967f787-vdc54 -o yaml
 #创建     kubectl create -f pod-demo.yaml
 #查看pod信息  kubectl get pods -o wide
 #查看详细信息  kubectl describe pods pod-demo
 #删除基于yaml文件pod  kubectl delete -f pod-demo.yaml
```



##### pod健康检测

    1. pod健康检测分为存活性探测、 就绪型探测；这在生产环境几乎是必须配置的；
   2. 如果没有就绪型探测；pod一启动就会被分配用户流量；若pod中的服务像tomcat等，需要时间启动；就会导致有一定时间，用户访问不到服务；
   3. 如果没有存活性探测：pod中服务一旦失败，没有检测，不会将容器重启关闭；也会导致用户访问服务失败

- **livenessProbe 存活性探测**
  -  exec：指定检测的命令
  -  failureThreshold：连续失败次数被认为失败，默认为3，最小值为1
  -  httpGet：指定要执行的http请求
  -  initialDelaySeconds：在容器启动多少秒后再检测
  -  periodSeconds：每隔多少秒探测一次；默认为10秒。最低限度值是1
  -  successThreshold：连续成功次数认为服务正常
  -  tcpSocket：定涉及TCP端口的操作
  -  timeoutSeconds:探测超时的秒数，默认为1秒
-  **readinessProbe 就绪型探测**（和livenessProbe 存活性探测选项一样）

```yaml
#pod健康检测选项
# 在spec字段下、containers字段配置，可使用explain查看详细用法 一般配置有三个

# kubectl explain pod.spec.containers.
  # Always：总是重启（默认）
  # OnFailure：只有容器状态为错误时，才重启
  # Never：绝不重启

```

下面列子： 

当探测到/tmp/healthy文件不存在时，认为服务故障；容器在30秒后执行删除/tmp/healthy文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec-pod
  namespace: default
spec:
  containers:
  - name: liveness-exec-container
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 3600"]
    livenessProbe:
      exec:
        command: ["test","-e","/tmp/healthy"]
      initialDelaySeconds: 1  #在容器启动后1秒开始检测
      periodSeconds: 3  #每隔3秒探测一次
  restartPolicy: Always  #总是重启pod
  
  #个人理解    bash -c "./atest hello world"实际上和./atest hello world等价
  #直接进入容器   kubectl exec -it nginx-dev-57b967f787-dr58z -- /bin/sh 
  #通过  kubectl describe pods liveness-httpget-pod 命令可以查看详细信息，是否重启过。
  
  
```



##### pod启动前/后钩子

-  pod在启动前后都可以设置钩子hook；在spec.containers.lifecycle字段下设置；
-  postStart：创建容器后立即调用PostStart操作；如果失败，根据重启策略终止；
-  preStop：在容器终止之前立即调用PreStop操作，该容器在处理程序完成后终止

   语法

​    $ kubectl explain pod.spec.containers.lifecycle

-  postStart
  -  exec：指定了要采取的命令；
  -  httpGet：指定要执行的http请求;
  -  tcpSocket：指定涉及TCP端口的操作
-  preStop （和postStart命令一样）

   

```yaml
# 启动容器前，先创建准备一个httpd服务的主页面文件/tmp/index.html
apiVersion: v1
kind: Pod
metadata:
  name: poststart-pod
  namespace: default
spec:
  containers:
  - name:  poststart-container
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    lifecycle:
      postStart:
        exec:
          command: ['/bin/sh','-c','echo hello > /tmp/index.html']
    command: ['/bin/sh','-c','/bin/httpd -f -h /tmp']
```



### pod控制器详解

(转载地址：https://www.cnblogs.com/along21/p/10318478.html)

#### pod控制器有多种类型

-  ReplicationController（RC）：RC保证了在所有时间内，都有特定数量的Pod副本正在运行，如果太多了，RC就杀死几个，如果太少了，RC会新建几个
-  **ReplicaSet（RS）**：代用户创建指定数量的pod副本数量，确保pod副本数量符合预期状态，并且支持滚动式自动扩容和缩容功能。
-  **Deployment**（重要）：工作在ReplicaSet之上，用于管理无状态应用，目前来说最好的控制器。支持滚动更新和回滚功能，还提供声明式配置。
-  **DaemonSet**：用于确保集群中的每一个节点只运行特定的pod副本，通常用于实现系统级后台任务。比如ELK服务
-  Job：只要完成就立即退出，不需要重启或重建。
-  CronJob：周期性任务控制，不需要持续后台运行
-  StatefulSet：管理有状态应用

我们一般用Deployment来管理ReplicaSet。 



#### Deployment



##### Deployment定义资源清单

```yaml
 apiVersion： app/v1  #版本
 kind： Deployment  #类型
 metadata  #元数据
 spec  #期望状态
       #--------------replicaset 也有的选项---------------
       minReadySeconds：应为新创建的pod准备好的最小秒数
       replicas：副本数； 默认为1
       selector：标签选择器
       template：模板（必须的）
           metadata：模板中的元数据
           spec：模板中的期望状态
       #--------------deployment 独有的选项---------------
       strategy：更新策略；用于将现有pod替换为新pod的部署策略
           Recreate：重新创建
           RollingUpdate：滚动更新
               maxSurge：可以在所需数量的pod之上安排的最大pod数；例如：5、10%
               maxUnavailable：更新期间可用的最大pod数；
       revisionHistoryLimit：要保留以允许回滚的旧ReplicaSet的数量，默认10
       paused：表示部署已暂停，部署控制器不处理部署
       progressDeadlineSeconds：在执行此操作之前部署的最长时间被认为是失败的
 status  当前状态
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 2 #副本数
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v1 #镜像
        ports:
        - name: http
          containerPort: 80    
   strategy:
     rollingUpdate:
        maxSurge: 1  #每次更新一个pod
        maxUnavailable: 0  #最大不可用pod为0       
```



##### deployment扩容/缩容/升级/回滚

```yaml
-----扩容/缩容 
#修改配置文件  
vim deploy-damo.yaml
#命令修改  
kubectl apply -f deploy-damo.yaml 
#可以查看pod扩容的一个数量过程 
kubectl get pods -w  相当于tail -f 时刻监控 

-----升级策略，可以在yaml文件中，设置每次升级一个pod
#先更新一个pod，然后立即暂停，等版本运行没问题了，再继续发布
kubectl set image deployment myapp-deploy myapp=ikubernetes/myapp:v3 && kubectl rollout pause deployment myapp-deploy

#中间可以一直监控过程
kubectl rollout status deployment myapp-deploy

#等版本运行没问题了，解除暂停，继续发布更新
kubectl rollout resume deployment myapp-deploy

------版本回滚

#查询版本变更历史
kubectl rollout history deployment myapp-deploy

#undo回滚版本；--to-revision=  回滚到第几版本
kubectl rollout undo deployment myapp-deploy --to-revision=1
```



#### DaemonSet

DaemonSet保证在每个Node上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用。

DaemonSet确保集群中每个（部分）node运行一份pod副本，当node加入集群时创建pod，当node离开集群时回收pod。如果删除DaemonSet，其创建的所有pod也被删除，DaemonSet中的pod覆盖整个集群。

```yaml
 apiVersion： app/v1  版本
 kind： DaemonSet  类型
 metadata  元数据
 spec  期望状态
       --------------replicaset 也有的选项---------------
       minReadySeconds：应为新创建的pod准备好的最小秒数
       selector：标签选择器
       template：模板（必须的）
             metadata：模板中的元数据
             spec：模板中的期望状态
       --------------daemonset 独有的选项---------------
       revisionHistoryLimit：要保留以允许回滚的旧ReplicaSet的数量，默认10
       updateStrategy：用新pod替换现有DaemonSet pod的更新策略
 status  当前状态
```

创建并创建一个简单的DaemonSet，启动pod，只后台运行filebeat手机日志服务

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat-ds
  namespace: default
spec:
  selector:
    matchLabels:
      app: filebeat
      release: stable
  template:
    metadata:
      labels:
        app: filebeat
        release: stable
    spec:
      containers:
      - name: filebeat
        image: ikubernetes/filebeat:5.6.5-alpine
        env: # 通过配置文件的env或者envFrom 字段来设置环境变量，进入容器可以获取这个值
        - name: REDIS_HOST
          value: redis.default.svc.cluster.local
        - name: REDIS_LOG_LEVEL
          value: info
```

scheduler调度来使用这种pod，然后一些数据放到数据卷中处理，有这样的场景。 



#### StatefulSet

StatefulSet是为了解决有状态的问题（对应Deployments和ReplicaSets是为无状态服务而设计），其应用场景包括

-  稳定的持久化存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
-  稳定的网络标志，即Pod重新调度后其PodName和HostName不变，基于Headless Service（即没有Cluster IP的Service）来实现
-  有序部署，有序扩展，即Pod是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从0到N-1，在下一个Pod运行之前所有之前的Pod必须都是Running和Ready状态），基于init containers来实现
-  有序收缩，有序删除（即从N-1到0）

三个必要组件

从上面的应用场景可以发现，StatefulSet由以下几个部分组成：

-  用于定义网络标志（DNS domain）的 **Headless Service**（无头服务）
-  定义具体应用的**StatefulSet控制器**
-  用于创建PersistentVolumes 的 **volumeClaimTemplates存储卷模板**



### service 

(转载地址：https://www.cnblogs.com/along21/p/10333086.html)



```yaml
 apiVersion： v1  版本
 kind： Service  类型
 metadata  元数据
 spec  期望状态
     ports：服务公开的端口列表；把哪个端口和后端建立联系
         port：此服务将公开的端口
         targetPort：要在服务所针对的pod上访问的端口的编号或名称
         nodePort：K8S 集群节点上的端口
     #selector：标签选择器；关联到哪些pod资源上
     #clusterIP：服务的IP地址，通常由主服务器随机分配，如果设置None，就是无头service
     type：确定服务的公开方式。 默认为ClusterIP
          #ClusterIP（默认） 仅用于集群内通信，集群内部可达，可以被各pod访问，节点本身可访问
          #NodePort  构建在ClusterIP上，并在路由到clusterIP的每个节点上分配一个端口；
          client ---> NodeIP:NodePort ---> ClusterIP:ServicePort ---> PodIP:containePort 
          #LoadBalancer 
          构建在NodePort上，并创建一个外部负载均衡器（如果在当前云中受支持），它将路由到clusterIP；
          #ExternelName
          通过CNAME将service与externalName的值(比如：foo.bar.example.com)映射起来. 
          要求kube-dns的版本为1.7或以上.
     sessionAffinity：service 负载均衡，默认值是None，根据iptables规则随机调度；
                      可使用sessionAffinity保持会话连线；
                      (如果设置为ClientIP，请求就只发往一个pod中，而不是随机)
  status  当前状态
  
  
  
  # 开发人员不使用默认的负载均衡策略，自定义负载均衡;或者希望知道同组服务的其他实例。 
  # 方法：创建无头服务，不为service设置clusterIP，
  
  
  
```



### Ingress

(转载地址https://www.cnblogs.com/along21/p/10333086.html)

Ingress是授权入站连接到达集群服务的规则集合。

```yaml
  internet
      |
------------
[ Services ]


 apiVersion： v1  版本
 kind： Ingress  类型
 metadata  元数据
 spec  期望状态
       backend： 默认后端，能够处理与任何规则不匹配的请求
       rules：用于配置Ingress的主机规则列表
       tls：目前Ingress仅支持单个TLS端口443
 status  当前状态
```

 创建ingress，绑定后端nginx服务

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
spec:
  rules:
  - host: myapp.along.com
    secretName: tomcat-ingress-secret #https 访问业务
    http:
      paths:
      - path:
        backend:
          serviceName: myapp #nginx镜像的service 
          servicePort: 80
```



#### https协议访问服务

```yaml
#创建私钥  
openssl genrsa -out tls.key 2048
#创建证书  
 openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Beijing/L=Beijing/O=DevOps/CN=tomcat.along.com
#创建secret
 kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key
 

```



### 存储劵详解

(转载地址：https://www.cnblogs.com/along21/p/10338242.html)

从另外一个方面讲，**Kubernetes volume，拥有明确的生命周期**，与所在的Pod的生命周期相同。因此，Kubernetes volume独立与任何容器，与Pod相关，所以数据在重启的过程中还会保留，当然，如果这个Pod被删除了，那么这些数据也会被删除。更重要的是，**Kubernetes volume 支持多种类型，任何容器都可以使用多个Kubernetes volume**。

它的核心，**一个 volume 就是一个目录，可能包含一些数据，这些数据对pod中的所有容器都是可用的，这个目录怎么使用，什么类型，由什么组成都是由特殊的volume 类型决定的。**

要使用Volume，pod需要指定Volume的类型和内容（**spec.volumes字段**），和映射到容器的位置（**spec.containers.volumeMounts字段**）。



#### 存储卷常用类型



```yaml
#非持久性存储#

#emptyDir
使用emptyDir，当Pod分配到Node上时，将会创建emptyDir，并且只要Node上的Pod一直运行，Volume就会一直存。当Pod（不管任何原因）从Node上被删除时，emptyDir也同时会删除，存储的数据也将永久删除。常用于作为临时目录、或缓存使用。
#hostPath
hostPath允许挂载Node（宿主机）上的文件系统到Pod里面去。如果Pod需要使用Node上的文件，可以使用hostPath。

#网络连接性存储#

#NFS存储卷
NFS 是Network File System的缩写，即网络文件系统。Kubernetes中通过简单地配置就可以挂载NFS到Pod中，而NFS中的数据是可以永久保存的，同时NFS支持同时写操作。Pod被删除时，Volume被卸载，内容被保留。这就意味着NFS能够允许我们提前对数据进行处理，而且这些数据可以在Pod之间相互传递.
　　
```





























