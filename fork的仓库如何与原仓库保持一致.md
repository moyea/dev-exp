repo1:  https://github.com/\<user1>/xxx.git

repo2:  https://github.com/\<user2>/xxx.git  &lt;forked from &lt;user1&gt;/xxx&gt;

1.在repo2中新建一个远端仓库，地址指向repo1的仓库

```bash
git remote add upstream <原始repo地址>
```

2.获取远端仓库repo1的更新

```bash
git fetch upstream
```

3.将repo1 的变更到应用到repo2上

```bash
git rebase upstream/master
```

4.将变更推送到repo2

```bash
git push --force
```

