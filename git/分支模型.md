# 分支模型
http://nvie.com/posts/a-successful-git-branching-model/


---
# Git工作流程

#### 服务器上常驻的分支有两个：`master` 和 `develop`，可以认为master代表的是生产环境，develop代表的是测试环境。
#### 也就是说：  `master`分支上的任何一个点检出都稳定的，可以部署到生产环境服务器；  而`develop`分支上每个点都可以检出，能启动运行，但是未经测试，还存在bug。
#### 注意事项：`切记不要在master分支上面直接提交代码`  不要使用 git push origin master

---

主要流程有4个：已发布版本（生产环境）bug处理，未发布版本（测试环境）bug处理，新特性开发，合并请求处理

### 已发布版本bug处理 ：从`master`分支检出，处理完成后双向合并到`master`和`develop`，检出的分支不提交到服务器

```
# 从master分支检出
git checkout master # 切换到 master 分支
git fetch && git rebase # 从服务器下载最新代码，如果本地有修改则合并上去，用 git pull --rebase 也可以，注意别用 git pull，会导致git网络图混乱
git checkout -b hotfix-Issue#xxx

# 在${hotfix-Issue#xxx}分支上完成Bug处理，一次或多次 git commit

# 合并到develop和master

git checkout master
# 把最新的代码捡下来并合并
git fetch && git rebase
# 把hotfix-Issue#xxxx分支合并到master
git merge --no-ff ${hotfix-Issue#xxx}
# 把合并完的master分支推到服务器上
git push origin master

git checkout develop
# 把最新的代码捡下来并合并
git fetch && git rebase
# 把hotfix-Issue#xxxx分支合并到develop上
git merge --no-ff ${hotfix-Issue#xxx}
# 把合并完的develop分支推到服务器上
git push origin develop

# 删除本地开发分支
git branch -d ${hotfix-Issue#xxx}
```

### 未发布版本bug处理：直接在`develop`分支上处理，不需要创建merge request？

```
git checkout develop #切换到develop分支
git fetch && git rebase # 从服务器下载最新代码，并与本地修改合并（如果有）
# 直接在develop分支上完成Bug处理，一次或多次 git commit
git push origin develop
```

### 新特性开发：从`develop`分支检出，开发完合并到`develop`分支，检出的新特性分支需要提交到服务器上，便于多人协作

```
# 从develop分支检出
git checkout develop
git fetch && git rebase # 从服务器下载最新代码，并与本地修改合并（如果有）
git checkout -b ${feature_name} develop # 分支名用小写英文单词，多个单词以短横线"-"分隔 

# 提交新特性分支到服务器
git push origin ${feature_name}

# 在${feature_name}分支上开发，多次git commit和git push origin ${feature_name}

#将完成的新特性提交到服务器
git push origin ${feature_name} 

# 在gitlab中创建一个pull request
```

### 合并请求处理：主要指新特性开发完成后，合并到`develop`分支

```
# 代码审核

# 合并到develop分支
git checkout develop
git merge --no-ff ${feature_name}  

# 提交develop到服务器，并删除新特性分支
git push origin develop

# 删除本地的分支
git branch -d ${feature_name}
# 删除服务器上的分支
git push origin :${feature_name} 
```


---

## 注意：相同分支的合并不要用 git merge，也不要用 git pull，要用 git rebase



---

对于服务器上别人已经删除的分支，可以用 `git branch -rd origin/【分支名】` 清理掉
比如，曾庆猛已经将服务器上的 `storetrsl` 分支删除了
但是在我本地 `git branch -a` 查看，还有显示 `remotes/origin/storetrsl` 这个
这时可以用 `git branch -rd origin/storetrsl `


## 新功能开发时，如果遇到需要将develop上面代码合并到自己新功能分支上面的情况

```
#首先检出服务器端最新代码
git fetch origin

#将最新的develop分支代码合并到新功能分支，这里需要处理冲突
git rebase origin/develop

#然后删掉远程仓库的该功能分支
git push origin :${feature_name}

#最后，将本地的新的新功能分支推到服务器
git push origin ${feature_name}
```

# 命名

* hotfix#255  : 线上bug
* fix#255 ：（未部署代码中的bug）
* custom-built#270 ：（项目定制）
* feature#273 ： （新特性）
* improvement#269 ： 改进（重构等都可算改进）