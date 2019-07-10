---
title:       "PAAS、IAAS和SAAS之间的区别"
subtitle:    ""
description: ""
date:        2019-07-08
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java相关概念", "PAAS", "IAAS", "SAAS", "云计算"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.mysubmail.com/chs/blog/view/45**

# PAAS、IAAS和SAAS之间的概念区别

用户通过Internet 可以从完善的计算机基础设施获得服务。这类服务可以称为基础设施即服务，这就是通常所说的IAAS。而相应的另外两种服务就是平台即服务和软件即服务。平台及服务提供了用户可以访问的完整或部分的应用程序开发,我们通常称之为PAAS。软件及服务则提供了完整的可直接使用的应用程序，我们通常称之为SAAS。如果把他们看作层次结构，那么第一层自然叫做IAAS，第二层就是PAAS，第三层也就是SAAS。

# 云计算的概念

云计算的概念来。假设你是一家超级牛逼的技术公司，根本不需要别人提供服务，你拥有基础设施、应用等等其它一切，你把它们分为三层：基础设施(infrastructure)、平台(platform)和软件(software)，

这其实就是云计算的三个分层，基础设施在最下端，平台在中间，软件在顶端，分别是Infrastructure-as-a-Service(IAAS)，Platform-as-a-Service(PAAS)，Software-as-a-Service(SAAS)，别的一些“软”的层可以在这些层上面添加。

# 本地部署

而你的公司什么都有，现在所处的状态叫本地部署(On-Premises)，就像在自己家做披萨一样。如果你想在办公室或者公司的网站上运行一些企业应用，你需要去买服务器，或者别的高昂的硬件来控制本地应用，让你的业务运行起来，这就叫本地部署。

# IAAS

IAAS: Infrastructure-as-a-Service(基础设施即服务)，有了IAAS，你可以将硬件外包到别的地方去。IAAS公司会提供场外服务器，存储和网络硬件，你可以租用。节省了维护成本和办公场地，公司可以在任何时候利用这些硬件来运行其应用。一些大的IAAS公司包括Amazon, Microsoft, VMWare, Rackspace和Red Hat.不过这些公司又都有自己的专长，比如Amazon和微软给你提供的不只是IAAS，他们还会将其计算能力出租给你来host你的网站。

# PAAS

PAAS: Platform-as-a-Service(平台即服务)，第二层就是所谓的PAAS，某些时候也叫做中间件。你公司所有的开发都可以在这一层进行，节省了时间和资源。

PAAS公司在网上提供各种开发和分发应用的解决方案，比如虚拟服务器和操作系统。这节省了你在硬件上的费用，也让分散的工作室之间的合作变得更加容易。网页应用管理，应用设计，应用虚拟主机，存储，安全以及应用开发协作工具等。

一些大的PAAS提供者有Google App Engine,Microsoft Azure，Force.com,Heroku，Engine Yard。最近兴起的公司有AppFog,Mendix和Standing Cloud.

# SAAS

SAAS: Software-as-a-Service(软件即服务)，第三层也就是所谓SAAS。这一层是和你的生活每天接触的一层，大多是通过网页浏览器来接入。任何一个远程服务器上的应用都可以通过网络来运行，就是SAAS了。你消费的服务完全是从网页如Netflix,MOG,Google Apps,Box.net,Dropbox或者苹果的iCloud那里进入这些分类。尽管这些网页服务是用作商务和娱乐或者两者都有，但这也算是云技术的一部分。一些用作商务的SaaS应用包括Citrix的Go To Meeting，Cisco的WebEx，Salesforce的CRM，ADP，Workday和SuccessFactors。