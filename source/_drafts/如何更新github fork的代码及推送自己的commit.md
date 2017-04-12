---
title: spring cloud
tags:
---

以GitHub用户appframe为例子，在你操作的时候要把下面的GitHub用户名appframe换成你自己的GitHub用户名：

注意事项：在更新自己Fork的代码之前，需要先把自己在本地的更改进行提交。

1、检出自己在github上fork的APDPlat分支（如果已经从netbenas中检出了代码，则此步骤为切换到APDPlat根目录然后执行第二步）

git clone https://github.com/appframe/APDPlat.git

cd APDPlat
2、增加APDPlat的远程原始分支（用户ysc的分支）到本地（如果以前已经执行过本操作，则可忽略，当然，需要用git remote -v命令里确认是否有APDPlat-ysc分支）

git remote add APDPlat-ysc https://github.com/ysc/APDPlat.git

运行命令：git remote -v你会发现多出来了一个APDPlat-ysc的远程分支。如下：

APDPlat-ysc     https://github.com/ysc/APDPlat.git (fetch)

APDPlat-ysc     https://github.com/ysc/APDPlat.git (push)

origin  https://github.com/appframe/APDPlat.git (fetch)

origin  https://github.com/appframe/APDPlat.git (push)
3、然后把远程原始分支APDPlat-ysc的代码拉到本地

git fetch APDPlat-ysc
4、然后合并对方远程原始分支APDPlat-ysc的代码

git merge APDPlat-ysc/master
5、最后把最新的代码推送到你的github上

git push origin master
6、给APDPlat-ysc发送Pull Request

用自己的github账号登陆github网站

打开https://github.com/appframe/APDPlat

点击Pull Request

点击New Pull Request

输入Title简要描述你改进的功能

输入详细的功能说明

点击Send pull request

这样就把你的所有commit发送给APDPlat-ysc了
