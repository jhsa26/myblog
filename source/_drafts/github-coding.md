## 准备工作

首先在需要[Github](github.com)和[Coding](coding.net)两个网站注册账号，[Github](github.com)可以免费托管开源代码，私有库的建立需要付费，是国外的一个网站。[Coding](coding.net)可以免费托管开源代码，同时他让用户免费建立私有库，提供的服务挺好的，是国内的网站。

## Hexo搭建博客

[Hexo](http://hexo.io)是一款基于 Node.js 的静态博客框架，Hexo 生成的静态页面可以部署在 Github 或 Coding 上，并且能够免费绑定自己的域名，可以用来很方便地搭建个人博客。用Hexo如何搭建博客的技术贴网上有很多，这里主要介绍一下Hexo源码的文件结构：

~~~
.
|-- public
|-- source
|-- theme
|-- _config.yml等文件
~~~
下面介绍一下每个目录
### public
  `public`文件夹是在执行`hexo generate`后生成的。
所以`/public` 文件夹中放置的文件，自然就是最终网页显示需要的静态网页文件，这个是因每个人博客的内容不同而不同，所以源码中不必要存在这个文件夹，这一点在我们查看` .gitignore`时会发现，这个文件夹是被忽略的，并不需要上传。当我们写好博客之后，执行`hexo g`生成网页文件，然后执行 `hexo deploy`，就会将`public`里面的文件push到github的`username.github.io`这个仓库和coding的`username.coding.me`这个仓库（如果你是同时托管），之后就能访问你的博客了。
### source
这个文件夹是用来存放你博客的文本内容。
打开`/source`之后，发现有一个`_post`文件夹，此文件夹中就是存放你所有博文内容的文件夹，博文是用markdown 来写的，写好之后，放入 _post即可。除此之外 `source` 还可以加入 `about`等文件夹，用来写 `关于我` 等页面的文本内容。
### theme
`theme` 自然就是放置博客主题的文件夹，比如源生的主题：`landscape`。自己通过`clone` 安装的三方主题也会出现在这里。
### _config.yml等
剩下的这些就是配置文件等

## 同时部署博客网页文件和托管博客源码
在向github或者Coding上推送代码之前，需要简单了解一下Git传输资料的四种协议：

   - **本地协议**
   - **HTTPS协议** 使用加密的网页访问通道**读写**仓库，使用用户名及密码进行鉴权，推荐轻量级用户使用。
   - **SSH协议**  使用加密通道**读写**仓库，无单次上传限制，需先设置“账户 SSH 公钥”，完成配对验证，推荐资深用户或经常推送大型文件用户使用
   - **Git协议** 网络传输协议里最快的，但缺乏授权机制，而且仅有**只读**权限，故仅适用于公开项目 clone 使用，

我使用的是SSH协议，所以我先在github和coding上设置SSH密钥，具体怎么设置网上有很多教程。
### 部署博客网页文件
在github和Coding的自己账户下各自建立项目，项目名称分别为`username.github.io`和`username.coding.me`，username是你自己的账户名，我的是`Ohaiyofei`。然后将博客根目录下的_config.yml的deploy修改成类似以下代码即可：

  ```
   deploy:
   type: git
   repo: 
          github: git@github.com:ohaiyofei/ohaiyofei.github.io.git,master
          coding: git@git.coding.net:Ohaiyofei/Ohaiyofei.coding.me.git,master 
    
  ```
 然后`hexo g`和`hexo d`即可，因为同时部署到了两个服务器上，所以通过[http://ohaiyofei.github.io/](http://ohaiyofei.github.io/)和[http://ohaiyofei.coding.me/](http://ohaiyofei.coding.me/)这两个域名都能访问我的博客，如果绑定了自己的域名，还可以通过第三个域名访问自己的博客。
 
### 托管博客源码
为什么想托管自己的博客源码，如果换个新环境想写博客时，只要从github或者coding上clone一下，就可以接着写。为了托管博客的源码，我在github和Coding自己账户下各自建立项目，项目名称都是Ohaiyofei.blog,你可以安装自己的爱好给项目区名字。然后需要将本地博客的git仓库与这两个远程库建立关联，然后同时推送。在博客目录下使用以下命令即可：

```bash
$ git remote add all git@github.com:ohaiyofei/ohaiyofei.blog.git
$ git remote set-url --add all git@git.coding.net:Ohaiyofei/Ohaiyofei.blog.git
$ git push all --all
```
当然使用上述命令的前提是你已经用git来管理自己的博客源码了。

## 总结

这篇博客写得有点长，算是对自己搭建博客的一个总结吧。
