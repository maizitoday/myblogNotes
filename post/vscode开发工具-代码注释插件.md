---
title:       "代码注释插件koroFileHeader"
subtitle:    ""
description: ""
date:        2019-08-11
author:      "麦子"
image:       "https://get.pxhere.com/photo/nature-blossom-black-and-white-plant-photography-meadow-dandelion-flower-bloom-photo-summer-monochrome-close-flora-close-up-pointed-flower-seeds-eye-macro-photography-flowering-plant-daisy-family-monochrome-photography-plant-stem-land-plant-807646.jpg"
tags:        ["开发工具", "代码注释插件"]
categories:  ["Tech" ]
---

插件地址：https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE

# 常规配置

```json
  "fileheader.customMade": {  // 头部注释
        "Description": "请输入....",
        "Author": "麦子",
        "Date": "Do not edit", // 文件创建时间(不变)
        "LastEditTime": "Do not edit", // 文件最后编辑时间
        "LastEditors": "麦子" // 文件最后编辑者
    },

    "fileheader.cursorMode": { // 函数注释
        "description":"方法说明....",
        "param":"",
        "return":"",
        "Date": "Do not edit", // 文件创建时间(不变)
    },

    "fileheader.configObj": {
        "autoAdd": true, // 自动添加头部注释开启才能自动添加
        "autoAlready": true, // 默认开启
        "prohibitAutoAdd": ["json"], // 禁止.json文件，自动添加头部注释
        "headInsertLine": {
            "java": 2 // java 后缀的文件，在第二行插入文件头部注释
          },
        "annotationStr": {
            "head": "/*", // 自定义注释头部
            "middle": " * @", // 自定义注释中间部分(注意空格,这也是最终生成注释的一部分)
            "end": " */", // 自定义注释尾部
            "use": true // 是否使用自定义注释符号
        }
    },
```

# 文件头部注释快捷键

```
ctrl+cmd+i
```



# 函数注释快捷键

快捷键的命令对应的是这个：cursorTip ， 有冲突，然后我这边进行了修改。

```
 ctrl+alt+f
```



# 注释生成接口文档

<https://gitee.com/treeleaf/xDoc>， 这个是一个很好的，注释直接生成接口的开源框架。 

![屏幕快照 2019-07-13 下午10.12.41](/img/屏幕快照 2019-07-13 下午10.12.41.png)



![屏幕快照 2019-07-13 下午10.12.32](/img/屏幕快照 2019-07-13 下午10.12.32.png)

