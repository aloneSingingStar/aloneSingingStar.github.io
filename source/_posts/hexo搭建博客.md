---
title: hexo搭建博客
date: 2017-09-10 13:48:42
tags:
---

## 1.在github上新建项目：
```
项目名称：github用户名称.github.io
example: aloneSingingStar.github.io
注意:最好创建空项目，不带一个文件
```

## 2.本地新建目录
```
随意创建，我的是aloneSingingStar.github.io
```

## 3.进入aloneSingingStar.github.io目录，初始化hexo
```
hexo init
```
<!-- more -->

## 4.安装依赖
```
npm install
```

## 5.安装博客部署插件
```
npm install hexo-deployer-git --save
```

## 6.配置_comfig.xml，设置部署分支为master
```
deploy:
  type: git
  repository: git@github.com:aloneSingingStar/aloneSingingStar.github.io.git
  branch: master
```

## 7.将项目添加到github
```
git init

git add .

git commit -m ‘初始化hexo’

git remote add origin git@github.com:aloneSingingStar/aloneSingingStar.github.io.git

git fetch

git push -u origin master -f (必须加上-f,而且执行这句后，之前在github上的原有文件会丢失)
在github上创建项目时，里面有一个readme.md文件，而本地项目git init时，里面没有这个文件，当使用（git push -u origin master）把本地文件提交上，就会有如下问题：
To github.com:aloneSingingStar/aloneSingingStar.github.io.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:aloneSingingStar/aloneSingingStar.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

## 8.在本地和远程创建hexo分支,并且本地切换到hexo分支,并拉取代码
```
git checkout -b hexo
git push origin hexo
git pull origin hexo(从hexo分支拉取代码，git pull无效果，可以git branch --set-upstream-to=origin/hexo(跟踪hexo的流)，并且git branch --unset-upstream master(取消对master的跟踪)，这样的话，就可以直接执行git pull、git push直接提交代码到hexo分支)
```

## 9.启动本地服务器测试
```
hexo server
访问 http://localhost:4000/ 预览效果
```

## 10.预览没有问题后，执行如下操作部署到github
```
hexo clean

hexo generate

hexo deploy

访问http://aloneSingingStar.github.io
```

## 11.配置主题
```
主题网址：[主题网址](https://hexo.io/themes/)
我使用的是yilia,步骤如下：
1.使用ssh方式克隆项目到themes目录下的yilia目录
git clone git@github.com:litten/hexo-theme-yilia.git themes/yilia
2.修改aloneSingingStar.github.io目录下的_config.yml中的主题
theme:yilia
3.具体样式可以修改yilia目录下的_config.yml文件
```

## 12.配置百度统计（只能后台统计，无法前台展示）
```
注册百度统计站长版
注册成功后，会得到一段代码，其中有一段是：hm.src = "https://hm.baidu.com/hm.js?这里是你的唯一code;
把code填写到yilia主题目录下的_config.yml中的
# Miscellaneous
baidu_analytics: '填写你的code'
google_analytics: false

[百度统计网址](https://tongji.baidu.com)
```

## 13.配置不蒜子，在网页显示访问量
```
我修改的是/Users/aloneSingingStar/xyb/blog/aloneSingingStar.github.io/themes/yilia/layout/_partial/footer.ejs文件
其中不蒜子我没有下载到本地，是直接引用的<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>

参考的是这个人的配置（https://github.com/sssvip/blog-data/blob/master/themes/yilia/layout/_partial/footer.ejs）
```

## 14.提交hexo分支上的修改
```
1.git status 查看代码修改

➜ /Users/aloneSingingStar/xyb/blog/aloneSingingStar.github.io git:(hexo) ✗ >git status
On branch hexo
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   _config.yml
        modified:   package.json
        new file:   "source/_posts/hexo\346\220\255\345\273\272\345\215\232\345\256\242.md"

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        themes/hexo-theme-spfk/
        themes/next/
        themes/yilia/

2.可以看到，有2个modified,一个new file,还有未被加入本地仓库的文件夹，这3个文件夹是我下载的主题，其中我配置了一些私密信息，比如，百度统计的唯一code、支付宝、微信打赏图片等，我不想上传，如果你想上传，可以使用:[git add .]将所有这个文件夹下的文件提交到本地仓库

3. git commit -m "描述"
如果这个文件已经提交到了远程仓库，本地做了修改，想再提交到远程仓库，如果只执行 git commit -m "描述" 是不行的，会报如下问题：

➜ /Users/aloneSingingStar/xyb/blog/aloneSingingStar.github.io git:(hexo) ✗ >git commit -m "配置说明修改"
On branch hexo
Changes not staged for commit:
        modified:   "source/_posts/hexo\346\220\255\345\273\272\345\215\232\345\256\242.md"

Untracked files:
        themes/hexo-theme-spfk/
        themes/next/
        themes/yilia/

no changes added to commit

说明已跟踪文件的内容发生了变化，但还没有放到暂存区。要暂存这次更新，需要运行 git add 命令，然后再提交

4.如果修改了很多个文件，那么一个个的去[git add 被修改的文件],然后再提交，会很麻烦，所以可以先使用[git status],查看所有未提交的文件，然后把不想提交的文件或者文件夹在[.gitignore]文件中过滤掉，这样的话，就可以直接使用[git add .]将所有未提交的提交到本地仓库

5. git push origin hexo

6. git pull origin hexo
```

## 15.在博客的md文件中，加入截断标签

```
如果没有加，一篇博客有多长，就展示多长，我们想要的效果是，在主页每篇博客只显示一部分，点击more后再进入详细页面

1.在需要截断的地方加入如下标签：<!-- more -->

2.在/Users/aloneSingingStar/xyb/blog/aloneSingingStar.github.io/themes/yilia/_config.yml文件中，加入：
excerpt_link: more
```
