# Git

![image-20220117191604376](https://s2.loli.net/2022/01/17/BIY4FcPDt1NjbeH.png)

![image-20220117192059567](https://s2.loli.net/2022/01/17/cJnTzO5WNxbf6p8.png)

---

初始化本地仓库

- 创建一个专门的文件夹GitRepositories

- 设置用户名和邮箱

```
git config --global user.name "zhuweihao"
```

```
git config --global user.email "2548237631@qq.com"
```

本地仓库的初始化操作

```
git init
```

---

Git常用命令

- 添加文件：add
- 提交文件：commit

> - 创建一个文件readme.md
> - 将文件提交到暂存区
>
> ```
> git add readme.md
> ```
>
> - 将暂存区的内容提交到本地库
>
> ```
> git commit -m "说明文档" readme.md
> ```

- 查看工作区和暂存区的状态

```
git status
```

- 查看提交日志，按日期从近到远排序

```
git log
```

当日志很多时，使用空格键进行向下翻页，使用B进行向上翻页，Q退出。

```
git log --pretty=oneline
```

```
git log --oneline
```

```
git reflog
```

- 前进或者后退历史版本：reset

```
git reset --hard 索引id
```

hard参数：本地库的指针移动的同时，改变本地库，暂存区和工作区也同步改动

mixed参数：改变本地库，暂存区同步改动，但是工作区不变

soft参数：改变本地库，暂存区和工作区不变

- 找回本地库删除的文件：实际上就是将版本滚回到未删除的版本

- diff命令

```
git diff 文件名 //比较特定文件的差异
```

```
git diff //可以比较工作区和暂存区所有文件的差异
```

```
git diff 索引id 文件名//比较与历史版本的差异
```

---

分支

在版本控制过程中，通常会多个线同时推进多个任务，也就是多个分支。

同时多个分支可以实现并行开发，互不影响，提高开发效率。

- 查看分支

```
git branch -v
```

- 创建分支

```
git branch 分支名
```

- 切换分支

```
git checkout 分支名
```

- 合并分支

```
git merge 分支名
```

---

GitHub

- 拉取操作：相当于 fetch+merge

```
git pull 远程库别名 远程库分支
```

- 推送操作

```
git push 远程库别名 本地库分支
```

---

IDEA集成Git



# Maven

目前无论使用IDEA还是Eclipse等其他IDE，使用里面的ANT工具，它可以帮助我们进行编译，打包运行等工作。

Apache基于ANT进行了升级，研发出了全新的自动化构建工具Maven。

Maven是Apache的一款开源的项目管理工具，无论是普通的JavaSE还是JavaEE项目，一般都创建Maven项目。

Maven使用项目对象模型（POM-Project Object Model）的概念，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。在Maven中每个项目都相当于是一个对象，对象和对象之间是有关系的。关系包含：依赖、继承、聚合，实现Maven项目可以更加方便的实现导入jar包、拆分项目等效果。





# Typora



typora中公式使用：https://blog.csdn.net/weixin_43445441/article/details/121491437





# 曙光服务器