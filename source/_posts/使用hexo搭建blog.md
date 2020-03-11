---
title: 使用hexo搭建blog
date: 2020-02-23 16:20:54
tags: 博客
---

1. 安装node.js和hexo框架(具体步骤可参照[视频](https://www.bilibili.com/video/av44544186?from=search&seid=8099617244954028635))

2. 进入BlogRoot/blog文件夹
```linux
$ cd BlogRoot/blog
```

3. 初始化一个博客,初始化的时候使用sudo,否则会出现permission denied的提示
```linux
$ sudo hexo init
```
4. 创建一篇博客正文
```linux
$ sudo hexo n "文件名"
```

5. 启动博客,就可以在本地预览博客内容了
```linux
$ sudo hexo s
```

6. 清理public folder和database folder
```linux
$ sudo hexo clean
```
7. 生成
```linux
$ sudo hexo g
```

8. 将博客部署到github上
> 1. 新建一个repository,名字必须为 昵称.github.io
> 2. 在blog目录安装git部署插件
>```linux
>$ cnpm install --save hexo-deployer-git
>```
> 3. 修改博客的部署配置
>> 1. 打开 blg目录下的 _config.yml文件,在文件的最下方可以看到默认的部署配置如下
>> ```linux
>> # Deployment
>>## Docs: https://hexo.io/docs/deployment.html
>>deploy:
>>  type: ''
>>```
>> 2. 将其type设置为git,repo设置为刚才创建的github reposit地址, branch设置为master分支
>> ```linux
>> # Deployment
>>## Docs: https://hexo.io/docs/deployment.html
>>deploy:
>>  type: git
>>  repo: https://github.com/JohnBuffalo/JohnBuffalo.github.io.git    
>>  branch: master
>>```
>> 3. 配置文件修改完毕后,输入下面的命令就可以开始部署了
>>```linux
>>$ hexo d
>>```
9. 之后需要创建新的博客文章了只需要重复4,5,6,7步即可,需要注意的是上面所有的操作都是在 blog/目录下进行的, 而且都需要管理员权限
