---
      title: hexo--编写发布一条龙
      tags:
---
### 项目的起源
hexo可以托管在github上作为纯静态博客使用，对我这种买不起服务器或主机的穷逼是很
利好的，可是呢，静态博客的发布是极其繁琐的
1. 需要在电脑上安装nodejs，hexo。电脑换了以后就找不到了
2. 只能用本地md软件编写，很是麻烦

当下环境下云化比较火，比如云游戏，云文档，能否在云端(服务器端)编写，发布后可以自
动部署呢

###项目原理
可以直接调用github api进行推送文件，使用github key即可，具体在[github api](https://docs.github.com/en/rest/guides/getting-started-with-the-rest-api)中，首先创建一
个blog的repository存放hexo源码，创建xxx.github.io的编译后的内容，再利用github actions
自动部署推送到xxx.github.io上即可，为了能够正确推送，将公钥放在blog上，私钥放在xxx.github.io上就可以了

