---
layout: post
title:  "Amazon Services List"
toc: true
---


## 1.Introduction 
  一套网站应用,用于展示 AWS 区域对应的服务信息，并提供搜索功能以支持区域选型和区域对比。由Serverless架构实现，使用Amazon CloudWatch Events、AWS Lambda、Amazon DynamoDB、Amazon S3、Amazon CloudFront等服务生成并展示数据，前端使用了React、Webpack以及Antd。
## 2.本demo功能
* 数据每12小时自动更新。  
* 可选择显示指定区域和指定服务的 AWS 区域表。
* 下载 AWS 区域表 csv 文件。
* 特定区域和特定服务的搜索。
* 需求的服务集合在哪些区域全部能满足的搜索。
* 区域间服务不同的对比。  
示例页面： [Demo link](http://aws-status-check-website.s3-website.ap-northeast-2.amazonaws.com/)

## 3.如何使用本demo？

### 3.1 代码构建：

```
$ git clone https://github.com/AwsDemoCenter/aws-services-list
$ cd aws-services-list
$ yarn install
$ yarn build
```
注：如有需要，请先安装yarn
### 3.2 服务端配置

选择一个部署所有服务的AWS区域，确保该区域内至少包含有上述提到的所有服务。
#### (1) 配置Amazon DynamoDB
在 AWS控制台 搜索DynamoDB，点击*创建表*，创建一张命名为aws-services-list表，主分区键为id（String）。其余按照实际情况进行配置，一般情况下默认即可。
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/dynamo_1.png">
![]({{site.baseurl}}/assets/demopic/dynamo_1.png)
</a>

#### (2) 上传py文件到s3

将Lambda文件夹下的commander.py和data.json文件压缩为*.zip压缩包，并上传到任意S3存储桶中，然后将文件公开化
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/s3_list1.png">
![]({{site.baseurl}}/assets/demopic/s3_list1.png)
</a>
#### (3) 配置AWS Lambda
在 AWS控制台 搜索Lambda，点击*创建函数*，创建三个Lambda函数，分别命名为aws-services-list-commander、aws-services-list-detector、aws-services-list-transmitter。模板选择空白模板，触发器暂时留空，运行语言Python 2.7，配置好IAM角色。其中，IAM角色至少需要Amazon DynamoDB的读写权限以及Amazon S3的写入权限。  
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/lambda.png">
![]({{site.baseurl}}/assets/demopic/lambda.png)
</a>
修改aws-services-list-commander代码，选择刚上传到S3上的压缩包。设置处理程序为commander.lambda_handler，并在基本设置中将超时时间设置为5分钟。
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/lambda_2.png">
![]({{site.baseurl}}/assets/demopic/lambda_2.png)
</a>
将Lambda文件夹下的detector.py文件内容复制粘贴到aws-services-list-detector的代码栏，超时时间设置为15秒。  
将Lambda文件夹下的transmitter.py文件内容复制粘贴到aws-services-list-transmitter的代码栏，超时时间设置为10秒。
#### (3) 配置Amazon CloudWatch Events
在 AWS控制台 搜索CloudWatch，点击左侧*事件*，*规则*，创建五条新的规则，分别与lambda函数aws-services-list-commander和aws-services-list-transmitter进行匹配。
其中aws-services-list-commander需要设置四条触发规则（随着服务的增多，参数会进行相应的更改），而aws-services-list-transmitter只需要设置一条无输入参数的触发（配置输入常量处只需填写{}）
<a data-fancybox="gallery" href="{{site.baseurl}}/assets/demopic/cloudwatch_3.png">
![]({{site.baseurl}}/assets/demopic/cloudwatch_3.png)
</a>
aws-services-list-commander的触发规则(点击代码右上角复制到剪贴板)：
```
#第一条规则

Cron表达式：  0 0/12 * * ？ *  #每隔12小时触发一次

配置输入 常量：
 {
	"min_services": 0,
	"max_services": 30
 }
 
#第二条规则

Cron表达式：  4 0/12 * * ？ *  #每隔12小时触发一次，比第一次触发晚4分钟

配置输入 常量：
 {
	"min_services": 30,
	"max_services": 60
 }
 
#第三条规则

Cron表达式：  8 0/12 * * ？ *  #每隔12小时触发一次，比第二次触发晚4分钟

 {
	"min_services": 60,
	"max_services": 90
 }
 
#第四条规则

Cron表达式：  12 0/12 * * ？ *  #每隔12小时触发一次，比第三次触发晚4分钟

 {
	"min_services": 90,
	"max_services": 120
 }
 
```

#### (4) 配置Amazon S3

创建S3存储桶，将 build 文件夹下所有文件上传到储存桶中，并设置所有文件公开化（操作方法同 （2）上传py文件到s3 ）
设置S3存储桶为静态网站，主页为index.html
## 4.本地运行
执行yarn start，即可在浏览器(chrome或IE9及以上)中进行本地访问
```
$ yarn start
```


注：以上所有操作如果遇到问题，请参阅AWS官方文档和官方信息；  
如果有任何疑问和建议，请联系邮箱：david.wang@finishy.cn。

