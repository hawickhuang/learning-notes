# 拉取github项目的所有分支

<!-- more -->

1. 编辑文件fetch-all.sh
    ```bash
    # fetch all branches
    # From: http://stackoverflow.com/questions/10312521/how-to-fetch-all-git-branches

    #!/bin/bash
    for branch in `git branch -a | grep remotes | grep -v HEAD | grep -v master`; do
        git branch --track ${branch##*/} $branch
    done

    git fetch --all
    git pull --all
    ```
2. 给文件添加可执行权限
    ```bash
    chmod +x fetch-all.sh
    ```
3. 在git项目下执行脚本
    ```bash
    ./fetch-all.sh
    ```