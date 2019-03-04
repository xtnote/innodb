### 配置文件
```
# ~/.tmux.conf
# 鼠标支持 - 设置为 on 来启用鼠标
setw -g mode-mouse on
# 开启用鼠标拖动调节pane的大小（拖动位置是pane之间的分隔线）
setw -g mouse-resize-pane on  
# 开启用鼠标点击pane来激活该pane
setw -g mouse-select-pane on  
# 启用鼠标点击来切换活动window（点击位置是状态栏的窗口名称）
setw -g mouse-select-window on
```

### ctrl+b
```
系统操作:
?	列出所有快捷键；按q返回
d	脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话
D	选择要脱离的会话；在同时开启了多个会话时使用
Ctrl+z	挂起当前会话
r	强制重绘未脱离的会话
s	选择并切换会话；在同时开启了多个会话时使用
:	进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器
[	进入复制模式；此时的操作与vi/emacs相同，按q/Esc退出
~	列出提示信息缓存；其中包含了之前tmux返回的各种提示信息

窗口操作:
c	创建新窗口
&	关闭当前窗口
0-9	切换至指定窗口
p	切换至上一窗口
n	切换至下一窗口
l	在前后两个窗口间互相切换
w	通过窗口列表切换窗口
,	重命名当前窗口；这样便于识别
.	修改当前窗口编号；相当于窗口重新排序
f	在所有窗口中查找指定文本

面板操作:
"	垂直分割，将当前面板平分为上下两块
%	水平分割，将当前面板平分为左右两块
x	关闭当前面板
!	将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板
Ctrl+方向键	以1个单元格为单位移动边缘以调整当前面板大小
Alt+方向键	以5个单元格为单位移动边缘以调整当前面板大小
Space	在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled
q	显示面板编号
o	在当前窗口中选择下一面板
方向键	移动光标以选择面板
{	向前置换当前面板
}	向后置换当前面板
Alt+o	逆时针旋转当前窗口的面板
Ctrl+o	顺时针旋转当前窗口的面板
```

### tmux命令
```
启动新会话：
tmux [new -s 会话名 -n 窗口名]

恢复会话：
tmux at [-t 会话名]

列出所有会话：
tmux ls

关闭会话：
tmux kill-session -t 会话名

关闭所有会话：
tmux ls | grep : | cut -d. -f1 | awk '{print substr($1, 0, length($1)-1)}' | xargs kill
```
