---
title:  "利用 Bitbucket 创建 maven 私有仓库"
date:   2016-04-21 01:02:22
categories: 
  - programming
tags: 
  - bitbucket
  - private repository
  - maven
---

使用Android Studio开发以后，有一项特别便利的功能就是添加依赖库只需要在gradle中添加一句代码就可以了：

```gradle
dependencies {       
    compile 'com.android.support:appcompat-v7:23.2.1'
}
```

一直都想知道如何创建一个库来这样使用，后来因为工作需要，要把项目中所使用的一些常用框架和工具类封装起来，统一化使用。所以上网查了下，可以利用Bitbucket来创建免费的maven私有库，这里主要是搬运一下别人的文章，稍微翻译和总结一下方法。

### 主要参考了以下的文章
> [1. JEROEN MOLS 的 GIT AS A SECURE PRIVATE MAVEN REPOSITORY](http://jeroenmols.com/blog/2016/02/05/wagongit/?utm_source=tuicool&utm_medium=referral)

这篇文章已经很详细的介绍了如何利用Bitbucket创建私有maven库的过程。在maven库创建成功并上传了之后，在Bitbucket对应项目设置对哪些成员开放，在项目中通过在gradle中添加对应的maven url和用户名密码进行验证使用依赖库，这样就达到了仓库私有化的目的。

----

**简单来说，有以下步骤：(图片均来源于以上参考的文章)**
1. 在Bitbucket上面新建一个私有的仓库

![新建一个私有仓库](/assets/images/20160421-1.png){:class="img-responsive"}

2. 创建一个 `README.md` 文件，并把它上传到一个新分支(文中所说的是新建了一个叫 `releases` 的新分支，其实新建一个分支，名字随你定就好，这是关键的一步，后面会试用到)
3. 创建一个gradle脚本文件，用来把库上传到Bitbucket。这里会用到一个叫Wagon-git的Maven 插件，这个gradle脚本文件在上面那个网站最后已经给了出来，在原文作者的github里面，可[点击这里查看](https://github.com/JeroenMols/GitAsMaven)，直接把文件下载下来，放到你要上传的库的根目录下面，然后在要上传的库(`module`)的`build.gradle`的最上面添加

```gradle
apply from: 'xxx.gradle'  //xxx是你下载下来那个文件名
//apply from: 'https://raw.githubusercontent.com/JeroenMols/GitAsMaven/master/publish-bitbucket.gradle'
//也可以不用下载文件，直接添加上面注释的的这一句
```

如果是使用原文作者的那个gradle脚本，要在library的根目录下面创建一个`gradle.properties`的文件来定义那个gradle脚本的变量：

```
ARTIFACT_VERSION=<版本号> 
ARTIFACT_NAME=<library的名字> 
ARTIFACT_PACKAGE=<packagename> ARTIFACT_PACKAGING=aar //或者 jar，这个用哪个自己查 
COMPANY=<bitbucket的团队名或者你的用户名> 
REPOSITORY_NAME=<步骤1创建的私有仓库的名字>
```

然后在项目( `project` )的根目录下面新建一个 `gradle.properties` 文件，用来填写你Bitbucket的用户名和密码：**注意，这个文件千万别上传到版本管理那里去了，要添加在 `.gitignore` 中**

```
USERNAME=<username_here>
PASSWORD=<password_here>
```

这样可以使用那个gradle脚本文件了。

我就是在这一步被困住了，因为我按照文中的方法去上传，但是Bitbucket的仓库一直没有签入记录，在Android Studio中Terminal中显示BUILD SUCCESSFUL的，后来在[这个stackoverflow的回答中找到了答案](http://stackoverflow.com/questions/33812099/how-to-publish-an-android-library-as-a-maven-artifact-on-bitbucket),大意就是wagon-git这个插件是用SSH来连接Bitbucket的，所以要在本地和Bitbucket中设置好SSH，[这里是设置方法](https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html),没什么难度的，按照步骤走下去就OK，记得把key贴到Bitbucket上面就可以了。然后在Terminal中重新运行

```bash
./gradlew uploadArchives
```

如无意外的话，去Bitbucket查看分支(步骤2中你所创建的分支)的commit记录就看到了你上传的库了，就像下图一样
![上传成功](http://upload-images.jianshu.io/upload_images/428521-0f6a9d96b8301a72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![上传记录](http://upload-images.jianshu.io/upload_images/428521-acd4184591eb9520.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4. 上传成功之后，就可以在项目中使用了。在project的 `build.gradle` 中添加已上传的maven信息：

```gradle
allprojects {   
         repositories {
                jcenter()
                maven {
                        url "https://api.bitbucket.org/1.0/repositories/用户名或团队名/仓库名/raw/分支名"            
                        credentials {
                                username USERNAME
                                password PASSWORD
                        }
                }
         }
}
```

然后在 `module` 的 `build.gradle` 像之前那样添加依赖那样引用就OK了：

```gradle
dependencies {
        ...
        // GROUP-ID      = ARTIFACT_PACKAGE
        // ARTIFACT-ID  = ARTIFACT_NAME
        // VERSION         = ARTIFACT_VERSION
        compile 'GROUP-ID:ARTIFACT-ID:VERSION'//这个按照步骤3中定义的填补吧
}
```

认真看看上文作者的gradle脚本的代码，有些地方还是挺好理解的。在走这个步骤的过程中，除了遇到SSH那个问题坑了我较长时间之外，总的来说步骤也不复杂。

## 完成...

----

> [2. Publish with Gradle on Bitbucket](https://medium.com/@Mul0w/publish-with-gradle-on-bitbucket-1463236dc460?mc_cid=5e6ec8b400&mc_eid=603dfed976#.widt2f2mx)

这个方法是在查到SSH问题那里发现了，里面使用的gradle脚本跟上面第一篇大同小异，文中也有对应github地址可以查看gradle文件，可以作为参考。

