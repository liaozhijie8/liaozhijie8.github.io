# Git基础

# git:分布式管理

## 一、基础概念

### 三个区域

工作区→暂存区→git仓库

### 三个状态

以修改、以暂存、已更新

### 安装配置

image.png

### 创建仓库

#### 一、将现有目录初始化为仓库

1、在项目目录中，打开git Bash，输入git init

2、查下文件状态 git status 或者 git status -s

![](/static/image-20220711220331463.png)

3、添加到暂存区 "git add ."或者 git add 指定名称

4、提交更新git commit -m "提交提示信息"

![](/assets/image/image-20220711221405180.png)

#### 二、修改文件后提交流程

如果已经添加到更新的文件发生了修改，可以用git status -s 查看

然后再次执行git add .提交更新到暂存区，利用git commit -m 提交更新

![](/assets/image/image-20220711223034666.png)

#### 三、撤销对文件的修改

注意：此时的文件并没有提交到更新仓库，撤销将是不可逆的，一般很少用的

![](/assets/image/image-20220711223634898.png)

#### 四、移除暂存区中的文件

git reset HEAD  + 指定文件名称 或者.移除全部文件出暂存区

#### 五、直接提交到仓库

git commit -a -m "描述消息"

#### 六、移除文件

1、工作区（本地）和仓库的文件都一起移除（不可逆）

git rm -f "文件名"

2、仓库文件移除，只保留工作区

git rm --cached "文件名"

#### 七、忽略文件

参考vue脚手架自带文件

#### 八、查看提交历史

git log -2 表示查看最近俩条

![](/assets/image/image-20220712011439346.png)

回退版本 git reset --hard +对应的commit号

![](/assets/image/image-20220712012337580.png)

回到回退前的版本号 先查看所有操作日记，包括已经回退了的  git reflog

git reset --hard 回退到指定版本

![](/assets/image/image-20220712012621158.png)

## 二、远程仓库操作

### 1、绑定

##### HTTPS

：不需要配置但是每次访问都需要输入github账号密码。

1）、绑定仓库

git remote add origin https://github.com/liaozhijie8/vue3.0_project_zhihu.git
git branch -M main
git push -u origin main

2）、后续的提交

利用git push 将本地commit的内容提交到远程仓库

##### SSH

：需要进行额外的配置，之后每次访问都不需要账号密码

1）、获取key

http://t.zoukankan.com/emily1130-p-7503546.html

2)、按照提示配置仓库

git push

### 2、分支

##### 1）、创建分支


git branch 分支名称 →git checkout 分支名称 切换分支

git checkout -b 分支名称 -----新建一个分支并切换到改分支下

##### 2）、合并分支
1、先切换到上一级分支 git checkout 分支名
2、合并分支 git merge 分支名

##### 3）、删除分支

**先回到上一层分支中，然后再用git branch -d 下一层分支名称删除分支**

##### 4）、分支冲突

打开冲突文件，手动解决冲突



##### 5）、将分支推送到远程仓库



##### 6）、跟踪远程仓库

参考GitHub

##### 7）、拉取远程仓库

git pull

8）、删除远程仓库的分支

**git push origin --delete 分支名称**


