---
layout: post
title:  "Amazon Rekognition"
toc: true
---


## 1.什么是 Amazon Rekognition？  
  利用 Amazon Rekognition，可将图像和视频分析轻松添加到您的应用程序。您只需向 Rekognition API 提供图像或视频，然后此服务就能识别物体、人员、文字、场景和活动。它还可以检测到任何不合适的内容。Amazon Rekognition 还提供了高度准确的人脸分析和人脸识别。您可以检测、分析和比较各种使用案例中的人脸，包括用户确认、编录、人员计数和公共安全。Amazon Rekognition 基于同样由 Amazon 计算机视觉科学家开发的成熟且高度可扩展的深度学习技术，每天能够分析数十亿图像和视频- 无需使用任何机器学习专业技能。Amazon Rekognition 包含一个简单易用的 API，该 API 可快速分析存储在 Amazon S3 中的任何图像或视频文件。Amazon Rekognition 始终从新数据进行学习，我们会不断向此服务添加新的标签和人脸识别功能。

## 2.demo介绍

这是一个Amazon Rekognition服务的示例项目。在本演示项目中，我们将展示AWS图像识别技术分别应用于对象场景检测，图像审核，面部分析，面孔比较以及面部识别等方面的实际效果。您可以通过页面演示来体验图像识别技术的准确性与效果。
本演示仅用于试验和参考用途，如果您想开发自己的图像识别应用，请登录AWS官网查看详细信息。如果您发现了项目中的任何错误或有任何建议，请发送邮件至：mailto:lintchen@amazon.com, mailto:davwan@amazon.com进行探讨，谢谢！

## 3.如何使用本demo？
开始使用本demo，必须先注册aws账号，然后创建一个s3 bucket,并创建cognito身份池
#### cognito身份池创建步骤：

在aws控制台搜索cognito，进入之后点击*管理身份池*，*创建新的身份池*，并做如下配置： 
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/cog_1.png">
![]({{site.baseurl}}/assets/demopic/cog_1.png)
</a> 
点击*创建池，*选择允许以创建两个与您的身份池关联的默认角色，一个用于未经身份验证的用户，另一个用于经过身份验证的用户。这些默认角色会向 Amazon Cognito Sync 提供身份池访问权限。
进入新创建的身份池，点击右上角*编辑身份池，*复制*身份池ID*，用于下面步骤进行项目部署
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/cong_2.png">
![]({{site.baseurl}}/assets/demopic/cong_2.png)
</a>  
注：配置Cognito用户池所对应的IAM角色具有至少S3项目存储桶全部操作权限，以及Amazon Rekognition全部操作权限

#### 项目部署：

* 将整体项目文件克隆至本地：   
```git clone https://github.com/finishy1995/amazon-rekognition-demo```
* 修改js文件夹下的common.js文件：  
常量BUCKET_NAME，替换为项目S3存储桶名  
常量POOL_ID，替换为项目Cognito用户池ID  
其余常量按照实际情况酌情修改
* 将修改好的项目文件全部上传至S3项目存储桶中。

#### s3 bucket设置：
对新创建的s3 bucket做如下设置：  
（1）静态网站设置  
  进入所创建的存储桶，点击*属性*，*静态网站托管*，设置主页为index.html,错误页面为error.html

<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/s3_1.png">
![]({{site.baseurl}}/assets/demopic/s3_1.png)
</a>
（2）日志功能激活  
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/s3_11.png">
![]({{site.baseurl}}/assets/demopic/s3_11.png)
</a>
（3）权限设置所有人可读  
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/s3_2.png">
![]({{site.baseurl}}/assets/demopic/s3_2.png)
</a>
（4）存储桶策略设置 
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/ss.png">
![]({{site.baseurl}}/assets/demopic/ss.png)
</a>
存储桶策略代码（点击复制到剪贴板）：  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::kikirekognitiondemo/"
        }
    ]
}
```  

（5）CORS配置网站地址获得GET, PUT, POST, DELETE操作权限  
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/s3_4.png">
![]({{site.baseurl}}/assets/demopic/s3_4.png)
</a>  
CORS配置代码（点击复制到剪贴板）：
```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>DELETE</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```
## 4.创建Collection：

面部识别功能的正常使用前需要先创建Collection，使用AWS CLI可以实现  
参考：https://docs.aws.amazon.com/zh_cn/rekognition/latest/dg/create-collection-procedure.html  
注：以上所有操作如果遇到问题，请参阅AWS官方文档

