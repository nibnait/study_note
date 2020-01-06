# git reset 代码回滚

回滚到远程分支的最新版本：

```
git reset --head origin/master
```



回滚到某次commit（并保留当前暂存区的所有修改）

```
git reset 7bfab4e
```



回滚到上一次commit状态，并放弃当前暂存区的所有修改

```
git reset --hard HEAD
```