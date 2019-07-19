---
layout: post
title: "Git Commands Related"
description: ""
category: "Git"
tags: [Git]
---

1. 撤销某次提交，恢复删除的文件

    - 通过git log 命令查找到想要恢复的commit id号
    <pre><code> commit cf9cb9fea61d5adeaf8967f2d51b04b3fd995a0e
    Author: Yitao Jiang <willierjyt@gmail.com>
    Date:   Thu Apr 17 00:03:21 2014 +0800
        Delete tags.
    </code></pre>
- git revert cf9cb9fea61d5adeaf8967f2d51b04b3fd995a0e
    <pre><code>[root@dvlp jiangyt.github.io]# git revert cf9cb9fea61d5adeaf8967f2d51b04b3fd995a0e
    Finished one revert.
    Revert tags.html and Upload new posts

    This reverts commit cf9cb9fea61d5adeaf8967f2d51b04b3fd995a0e.

    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.
    # On branch master$
    # Changes to be committed:$
    #   (use "git reset HEAD <file>..." to unstage)
    #
    # new file:   tags.html</code></pre>

- 保存，退出后:wq 文件已恢复
- 查看当前提交记录
    _git log_
    <pre><code>  commit bba1aaa176b12a5abd47f852cbb28f187d1d5e5d
    Author: Yitao Jiang <willierjyt@gmail.com>
    Date:   Sun May 25 22:12:12 2014 +0800

    Revert tags.html and Upload new posts

    This reverts commit cf9cb9fea61d5adeaf8967f2d51b04b3fd995a0e.
    </code></pre>

    可以看到在revert后，产生的commit记录中包含revert的commit id号
