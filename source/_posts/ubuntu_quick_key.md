---
title: Ubuntu终端常用的快捷键
date: 2021-03-15 11:42:41
tags: 
- ubuntu
- Linux
categories: 
- ubuntu
- Linux
top: 36
description: 
password: 

---

Ubuntu中的许多操作在终端（Terminal）中十分的快捷，记住一些快捷键的操作更得心应手。在Ubuntu中打开终端的快捷键是Ctrl+Alt+T。其他的一些常用的快捷键如下：

<!--more-->

# terminal 
```
Tab         自动补全
Ctrl+a      光标移动到开始位置
Ctrl+e      光标移动到最末尾
Ctrl+k      删除此处至末尾的所有内容
Ctrl+u      删除此处至开始的所有内容
Ctrl+d      删除当前字符
Ctrl+h      删除当前字符前一个字符
Ctrl+w      删除此处到左边的单词
Ctrl+y      粘贴由Ctrl+u， Ctrl+d， Ctrl+w删除的单词
Ctrl+l      相当于clear，即清屏
Ctrl+r      查找历史命令
Ctrl+b      向回移动光标
Ctrl+f      向前移动光标
Ctrl+t      将光标位置的字符和前一个字符进行位置交换
Ctrl+&      恢复 ctrl+h 或者 ctrl+d 或者 ctrl+w 删除的内容
Ctrl+S      暂停屏幕输出
Ctrl+Q      继续屏幕输出
Ctrl+Left-Arrow     光标移动到上一个单词的词首
Ctrl+Right-Arrow    光标移动到下一个单词的词尾
Ctrl+p              向上显示缓存命令
Ctrl+n              向下显示缓存命令
Ctrl+d              关闭终端
Ctrl+xx             在EOL和当前光标位置移动
Ctrl+x@             显示可能hostname补全
Ctrl+c              终止进程/命令
Shift+上或下        终端上下滚动
Shift+PgUp/PgDn     终端上下翻页滚动
Ctrl+Shift+n        新终端
alt+F2              输入gnome-terminal打开终端
Shift+Ctrl+T        打开新的标签页
Shift+Ctrl+W        关闭标签页
Shift+Ctrl+C        复制
Shift+Ctrl+V        粘贴
Alt+数字            切换至对应的标签页
Shift+Ctrl+N        打开新的终端窗口
Shift+Ctrl+Q        管壁终端窗口
Shift+Ctrl+PgUp/PgDn左移右移标签页
Ctrl+PgUp/PgDn      切换标签页
F1                  打开帮助指南
F10                 激活菜单栏
F11                 全屏切换
Alt+F               打开 “文件” 菜单（file）
Alt+E               打开 “编辑” 菜单（edit）
Alt+V               打开 “查看” 菜单（view）
Alt+S               打开 “搜索” 菜单（search）
Alt+T               打开 “终端” 菜单（terminal）
Alt+H               打开 “帮助” 菜单（help）
```


另外一些小技巧包括：在终端窗口命令提示符下，连续按两次 Tab 键、或者连续按三次 Esc 键、或者按 Ctrl+I 组合键，将显示所有的命令及工具名称。Application 键即位置在键盘上右 Ctrl 键左边的那个键，作用相当于单击鼠标右键。


# Terminal终端 
```
CTRL + ALT + T: 打开终端
TAB: 自动补全命令或文件名
CTRL + SHIFT + V: 粘贴（Linux中不需要复制的动作，文本被选择就自动被复制）
CTRL + SHIFT + T: 新建标签页
CTRL + D: 关闭标签页
CTRL + L: 清楚屏幕
CTRL + R + 文本: 在输入历史中搜索
CTRL + A: 移动到行首
CTRL + E: 移动到行末
CTRL + C: 终止当前任务
CTRL + Z: 把当前任务放到后台运行（相当于运行命令时后面加&）
~: 表示用户目录路径
```

# 如何打开一个程序
```
以“系统配置”为例，先按SUPER + A，SUPER即Win键，然后切换到中文输入法，输入“系统配置”，按回车即打开程序。再按TAB键浏览系统配置里的子配置程序
```

# 桌面 
```
ALT + F1: 聚焦到桌面左侧任务导航栏，可按上下键导航。
ALT + F2: 运行命令
ALT + F4: 关闭窗口
ALT + TAB: 切换程序窗口
ALT + 空格: 打开窗口菜单
PRINT: 桌面截图
SUPER: 打开Dash面板，可搜索或浏览项目，默认有个搜索框，按“下”方向键进入浏览区域（SUPER键指Win键或苹果电脑的Command键）
在Dash面板中按CTRL + TAB: 切换到下一个子面板（可搜索不同类型项目，如程序、文件、音乐）
SUPER + A: 搜索或浏览程序（Application）
SUPER + F: 搜索或浏览文件（File）
SUPER + M: 搜索或浏览音乐文件（Music）
```

# Orca读屏软件 
```
启动Orca: SUPER + A，然后输入orca，然后回车
ORCA + 空格: 显示首选项对话框（ORCA键是指Insert插入键或CAPS LOCK大小写转换键，取决于设置）
ORCA + t: 读当前时间
ORCA + tt: 读当前日期
ORCA + s: 切换合成语音开关
ORCA + /: 朗读标题
ORCA + //: 朗读状态栏
ORCA + 分号: 朗读整个文件
ORCA + Q: 退出Orca
更多快捷键请参考Orca首选项的键绑定标签页
```

# Firefox浏览器 
```
进入Firefox的方法：
1. SUPER + A，然后按firefox，回车。这个是在Dash面板中搜索应用程序运行。事实上，只要按fir就能定位到Firefox程序。
2. ALT，然后按firefox，回车。这个相当于在命令行运行一条命令。
3. 在终端中按firefox&，回车。这个适用于以终端作为主要操作窗口的用户，使用TAB键还可以自动补全命令（只需输入前几个字母再按TAB键）。&在shell中是后台运行的意思，这样终端就不会被Firefox独占。
CTRL + T: 新建标签页
CTRL + W: 关闭标签页
CTRL + SHIFT + T: 重新打开最近关闭的一个标签页
CTRL + TAB: 切换到下一个标签页
CTRL + SHIFT + TAB: 切换到上一个标签页
ALT + [1-8]: 跳到制定标签页
ALT + 9: 跳到最后一个标签页
CTRL + L: 跳到地址栏
ESC: 停止加载当前页面
CTRL + K: 跳到搜索引擎输入框
CTRL + F: 在当前页面中搜索
/: 快速查找。在Linux中很多程序（如VI、Man、Less）都使用/作为搜索的快捷键，并且可使用正则表达式查找。但在Firefox中没有正则表达式搜索的功能。
CTRL + D: 收藏到书签
ALT + 左方向键: 后退
ALT + v: 前进
CTRL + Q: 退出
```

# Gedit文本编辑器
```
启动gedit：SUPER + A，然后按gedit，回车
CTRL + N: 新建文档
CTRL + W: 关闭文档
CTRL + S: 保存
CTRL + SHIFT + S: 另存为
CTRL + S: 搜索
CTRL + H: 搜索并替换
CTRL + I: 跳到某一行
CTRL + C: 复制
CTRL + V: 粘贴
CTRL + X: 剪切
CTRL + Q: 退出
```

# Nautilus文件管理器 
```
启动Nautilus的方法：
1. SUPER + 1，这个方法仅适用于Nautilus在左边快速启动的位置没有改变的情况。
2. SUPER + A，然后输入nautilus，然后回车
F2: 重命名
CTRL + 1: 图标视图
CTRL + 2: 列表视图
CTRL + T: 新建标签页
CTRL + W: 关闭标签页
CTRL + D: 收藏到书签
CTRL + Q: 退出
Nautilus还有很多和Firefox一致的快捷键。
```

# Rhythmbox音频播放器 
```
CTRL + 空格: 播放/暂停
ALT + 右方向键: 下一首
ALT + 左方向键: 上一首
CTRL + 上方向键: 增大音量
CTRL + 下方向键: 减少音量
CTRL + U: 随机播放
CTRL + R: 重复播放
CTRL + Q: 退出
```
