# Tmux 安装与使用教程


## 简介
1. 终端里的“窗口管理器”， 他能保存我们与服务器的连接session，而不会因为以为而终端在服务器上运行的持久性命令。
2. 多窗口操作
3. copy会话
4. 多会话操作

## 安装
1. mac
    ```shell
    brew install tmux
    ```
2. ubuntu
    ```shell
    sudo apt-get install tmux
    ```

## 基本操作
### 长命令
1. tmux list-keys，列出tmux所有命令和快捷键
2. tmux list-commands，列出tmux所有命令及其参数
3. tmux info，列出session、window、pane运行的进城号
4. tmux new -s session_name，创建tmux session
5. tmux attach -t session_name，重新开启某sesion
6. tmux switch -t session_name，切换到某session
7. tmux list-sessions / tmux ls，列出所有的session
8. tmux detach，离开当前session
9. tmx kill-server，关闭所有session
10. tmux new-window，创建新window
11. tmux list-windows，列出所有window
12. tmux select-window -t :0-9，根据索引切换window
13. tmux rename-window，重命名当前window
14. tmux split-windown， 垂直划分为两个pane
15. tmux split-window -h，水平划分为两个pane
16. tmux swap-pane -[UDLR]，按指定方向交换pane
17. tmux select-pane -[UDLR]，指定方向选择下一个pane

### 简洁命令(按下tmux前缀,ctrl+b)
1. :new<enter>，启动新会话
2. s，列出所有会话
3. $，重命名当前会话
4. c，创建新窗口
5. w，列出所有窗口
6. n，后一个窗口
7. p，前一个窗口
8. f，查找窗口
9. ,，重命名窗口
10. &，关闭窗口
11. swap-window -s 1 -t 2，交换1和2窗口
12. swap-window -t 1，交换当前和1窗口
13. move-window -t 1，移动当前至1窗口
14. %，垂直分割
15. "，水平分割
16. o，交换窗格
17. x，关闭窗格
18. space(空格键)，切换布局
19. q，显示窗口编号
20. {，与上一个窗格交换位置
21. }，与下一个窗格交换位置
22. z，切换窗口最大化/最小化
23. :setw synchronize-panes，同步窗口
24. :resize-pane -D，当前窗口向下扩大1格
25. :resize-pane -U，当前窗口向上扩大1格
26. :resize-pane -L，当前窗口向左扩大一格
27. :resize-pane -R，当前窗口向右扩大一格
28. :resize-pane -D 2，当前窗口向下扩大2格
29. :resize-pane -t 2 -L 2，2号窗口向左扩大2格
30. d，退出tmux（在后台运行）
31. t，窗口中央显示数字时钟
32. ?，列出所有快捷键
33. :，命令提示符

## 配置
1. 鼠标支持
    ```shell
    * setw -g mode-mouse off
    * set -g mouse-select-pane off
    * set -g mouse-resize-pane off
    * set -g mouse-select-window off
    ```
2. 启用活动警告
    ```shell
    setw -g monitor-activity on
    set -g visual-activity on
    ```
