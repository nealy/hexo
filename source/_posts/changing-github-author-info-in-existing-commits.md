title: 修改 Github commits 的作者信息
date: 2016-04-28 13:48:43
updated: 2016-04-28 13:48:43
tags: [github]
categories: git
---

有时候在不同电脑上提交代码到 Github 上的时候，经常会忘记设置 `user.email`，某些系统（比如 Mac OS X） 就会默认使用本地地址作为 Email 地址来 commit change，这样 Github 上就出现一大堆 contribution graph 空白了，因为这些 commits 的作者并不是你（邮箱不对）。那么对这些已经提交的 changes 该如何补救呢？
<!-- more -->

首先，应该正确配置邮箱和用户名：

```text
git config --global user.email "youremail@gmail.com"
git config --global user.name "your name"
```

但这种只能对以后的commit起作用了。如果想要修改之前的信息的话，Github 也提供了完整的解决方案。
需要注意的是，想要改变已经存在的 commit 的用户名和/或邮箱地址，就必须重写整个 Github 仓库的历史记录。

```text
警告：这种行为对仓库具有破坏性。如果你与他人协同工作，重写历史记录并不是一种好的做法，应当仅用于紧急情况。
```

### 使用脚本来重写仓库的 git 历史

Github 提供了一段脚本来更改 commits 历史的作者和/或正确的邮箱地址。

```text
注意：执行这段脚本会重写仓库里所有协作者的历史。完成以下操作后，任何 fork 或 clone 的人都必须获取重写后的历史，并把所有本地修改 rebase 到重写后的历史中方可。
```

首先，在执行这段脚本之前，你需要准备的是：

* 欲修改的旧的用户名和/或邮箱地址
* 正确的用户名和/或邮箱地址

然后：

1. 打开终端（Mac 或 Linux 用户）或命令行（Windows 用户）。

2. 创建一个全新的、空的（bare） clone （repo.git 替换为你的项目，下同）

    ```bash
    git clone --bare https://github.com/user/repo.git
    cd repo.git
    ```

3. 拷贝、粘贴脚本，并根据自身情况替换以下信息。

    `OLD_EMAIL`
    `CORRECT_NAME`
    `CORRECT_EMAIL`

    脚本：

    ```bash
    #!/bin/sh

    git filter-branch --env-filter '
    OLD_EMAIL="your-old-email@example.com"
    CORRECT_NAME="Your Correct Name"
    CORRECT_EMAIL="your-correct-email@example.com"
    if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_COMMITTER_NAME="$CORRECT_NAME"
        export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
    fi
    if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_AUTHOR_NAME="$CORRECT_NAME"
        export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
    fi
    ' --tag-name-filter cat -- --branches --tags
    ```

4. 按 Enter 键执行脚本。

5. 查看 Git 历史检查是否正确。

6. 将正确的历史　push 到　Github。

7. 清除临时 clone。

    ```
    cd ..
    rm -rf repo.git
    ```

Reference: [Changing author info](https://help.github.com/articles/changing-author-info/)
