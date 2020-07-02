### Git上传完整流程：

1. 在项目文件夹下，启动`git branch`,输入`git init`

2. `git add . `添加所有文件

3. `git commit -m ''` 上传到本地仓库

4. `git remote add origin git@github.com XXXX.git` 关联`GitHub`仓库

   ​	如果出现错误：fatal: `remote origin already exists`，则执行以下语句：`git remote rm origin`，再执行4

5. `git push origin master`(git push -u origin master,第一次加上 -u,创建)

   ​	git push <远程主机名> <本地分支名>:<远程分支名>：推送本地master分支到远程仓库的master分支

6. 如果5出现错误:`failed to push som refs to……`,则执行以下语句：`git pull origin master`，再执行5

7. 如果执行6出现错误:fatal: `refusing to merge unrelated histories`(拒绝合并不想关历史)，则执行`git pull origin master --allow-unrelated-histories`，将两个独立的仓库合并（会合并历史），再执行6

8. <font color=ff0000>**冲突**</font>：执行第五步`git pull origin master`时，

   ~~~
   error: Your local changes to the following files would be overwritten by merge:XXX
   Please commit your changes or stash them before you merge.
   Aborting
   ~~~

   三种解决方法：

   - 你先提交自己对文件的修改，再去拉代码

     ~~~git
     git add .  #修改放到暂存区 
     git commit -m '哥不服'  #提交到本地仓库
     git push  #假设就一个远程分支，该省略的全省了
     ~~~

   - 你先把自己修改的内容藏起来

     ```git
     git stash #先把自己的修改都存储起来
     git pull  #拉新代码
     git stash pop  #再把存储的代码拿出来，在本地仓库自动合并
     ```

   - 远程分支为准，自己在本地的修改不要了

     ```git
     git reset --hard #放弃本地修改
     git pull #拉远程分支代码
     ```

---

### git push

git push命令用于将本地分支的更新，推送到远程主机。它的格式与git pull命令相仿。

```
git push <远程主机名> <本地分支名>:<远程分支名>
1
```

**注意：这里的:前后是必须没有空格的。**

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

```
git push origin master
1
```

上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

```
#慎用！删除远程仓库的分支
git push origin :master
# 等同于
git push origin --delete master
1234
```

上面命令表示删除origin主机的master分支。

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

```
 git push origin
1
```

上面命令表示，将当前分支推送到origin主机的对应分支。

如果当前分支只有一个追踪分支，那么主机名都可以省略。

```
git push
1
```

如果当前分支与多个主机存在追踪关系，则可以使用-u选项指定一个默认主机，这样后面就可以不加任何参数使用git push。

```
git push -u origin master
1
```

上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。

---

### git pull 

pull ≈ fetch + merge，fetch并不会自动合并或修改当前的工作，必须手动将其合并到工作当中。

如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。

---

### git clone

在本地文件夹下(非本地仓库，无.git文件),从gitHub上down下文件，用`git clone` 拷贝，pull和push只能在`git remote add origin XXXX.git`操作(关联相关的远程库)之后使用

---

### git add

- **git add . ** 提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件

- **git add -u**  提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
- **git add -A**  提交所有变化

.**gitignore 不起作用问题**

git忽略目录中，新建的文件在git中会有缓存，如果某些文件已经被纳入了版本管理中，就算是在.gitignore中已经声明了忽略路径也是不起作用的，这时候我们就应该先把本地缓存删除，然后再进行git的push，这样就不会出现忽略的文件了。git清除本地缓存命令如下：

```
git ``rm` `-r --cached .
git add .
```

