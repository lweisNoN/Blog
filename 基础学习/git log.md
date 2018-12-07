## git 回退 master

```
git checkout master

git reset --hard <commit id>

git push --force origin master

Then to prove it (it won't print any diff)

git diff master..origin/master

```



## git 回退 branch

```
git revert <commit id>
git push
```



## git 回退本地



```
git reset <commit id>
```

