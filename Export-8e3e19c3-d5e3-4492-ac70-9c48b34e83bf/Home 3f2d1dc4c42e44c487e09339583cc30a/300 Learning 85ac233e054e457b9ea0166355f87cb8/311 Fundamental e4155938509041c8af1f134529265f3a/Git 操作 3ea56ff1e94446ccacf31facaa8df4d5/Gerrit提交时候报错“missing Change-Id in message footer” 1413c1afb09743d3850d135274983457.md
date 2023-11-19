# Gerrit提交时候报错“missing Change-Id in message footer”

```bash
remote: ERROR: commit 8d1595d: missing Change-Id in message footer
remote: 
remote: Hint: to automatically insert a Change-Id, install the hook:
remote:   gitdir=$(git rev-parse --git-dir); scp -p -P 29418 jinyong@gerrit.konka.com:hooks/commit-msg ${gitdir}/hooks/
remote: and then amend the commit:
remote:   git commit --amend --no-edit
remote:
```

这是因为在`.git/hooks` 目录下少了一个`commit-msg` 文件。

本来应该是直接拉下来的，但是不知道为啥有问题，少了这个文件，需要从别的项目里面手动拷贝到这个hooks文件夹。