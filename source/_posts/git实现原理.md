---
title: git实现原理
date: 2019-12-13 00:07:49
tags:
- git
category:
- git
---





今天学习了一点`git`的实现原理，稍微记录一下

<!--more-->



# before all

我们先来新建一个测试仓库，就叫`test`吧，使用`git init`来初始化这个仓库。

现在这个仓库是全空的，打开`.git`文件夹，发现里边有文件夹，但是objects啊refs啊都是空的；

```
...
+---objects
|   +---info
|   \---pack
\---refs
    +---heads
    \---tags
```

我们就从这个仓库开始讲吧



# `git`存储原理

我们新建几个文件试一下，新建3个文件，a.txt，b.txt，c.txt，其中a和b的内容是一样的

```bash
$ cat a.txt b.txt c.txt
1.2.3 
1.2.3
4.5.6
```

然后我们使用 `git add *` 命令，然后再次查看`.git`文件夹

```
+---objects
|   +---04
|   |       95c4a88caed0f036ffab0948c17d4b5fdc96c1
|   |
|   +---bb
|   |       8dc6360fdfeabbafe0681b258a816b259a6048
|   |
|   +---info
|   \---pack
\---refs
    +---heads
    \---tags
```

看！多出来两个object文件，这两个文件内容是什么呢

```bash
/d/test/.git/objects/04 (GIT_DIR!)
$ cat 95c4a88caed0f036ffab0948c17d4b5fdc96c1  
xK??OR0c0?3?3? N 
```

发现不是明文的，这是因为

* git将文件内容进行了压缩处理，并且加入了一些自己管理所需的信息，得到了里边这个字符串的内容；
* 并且git对文件内容进行了哈希，得到了这个哈希值，作为文件名。

我们可以使用`git cat-file -p`查看内容，使用`git cat-file -t`查看文件类型

```bash
$ git cat-file -t 0495 && git cat-file -p 0495  
blob 
1.2.3 

$ git cat-file -t bb8d && git cat-file -p bb8d 
blob 
4.5.6
```

两个object文件都是Blob类型（二进制文件），内容分别是a.txt(b.txt)和c.txt的内容。

明显，这两个文件就存储了我们的文本信息，这也就是git存储的真正的文本信息。

这时候突然想到了，我们有三个文件啊，看起来很明显了，git对两个相同的文件求哈希以后发现文件相同，因此只保存了一份在数据库里。



# `Git`三个区域

`git`分为三个区域，分别为

* 工作区：就是敲代码的时候，代码都存在工作区里；
* 暂存区：这个就是使用了`git add`，然后会把这部分代码暂存起来；
* git仓库：这就是最终的存储地方了





# 未完待续