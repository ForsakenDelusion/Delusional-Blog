---
title: Macos在文件夹中快速打开终端
date: 2024-07-07 00:34:08 +08:00
filename: 2024-07-07-Macos-folder-terminal
categories:
  - Misc
tags:
  - macOS
dir: Misc
share: true
---
使用自动操作+AppleScript的方法

[参考文章](https://blog.csdn.net/weixin_43093163/article/details/131074213?ops_request_misc=%25257B%252522request%25255Fid%252522%25253A%252522170688932316800192259225%252522%25252C%252522scm%252522%25253A%25252220140713.130102334.pc%25255Fall.%252522%25257D&request_id=170688932316800192259225&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-131074213-null-null.142%5Ev99%5Epc_search_result_base5&utm_term=mac%2520%25E6%2596%2587%25E4%25BB%25B6%25E5%25A4%25B9%2520%25E7%25BB%2588%25E7%25AB%25AF&spm=1018.2226.3001.4187)

- 打开`自动操作`
- 新建应用程序
- 切换实用工具
- 运行AppleScript
- 贴代码

我这里使用`iTerm2`，如果你使用默认终端，直接把`iTerm`改成`Terminal`就行了，可以看一下我的代码，逻辑很简单。

```script
on run {input, parameters}

tell application "Finder"

set pathList to (quoted form of POSIX path of (folder of the front window as alias))

set command to "clear; cd " & pathList

end tell

tell application "System Events"

# some versions might identify as "iTerm2" instead of "iTerm"

set isRunning to (exists (processes where name is "iTerm")) or (exists (processes where name is "iTerm2"))

end tell

tell application "iTerm"

activate

set hasNoWindows to ((count of windows) is 0)

if isRunning and hasNoWindows then

create window with default profile

end if

select first window

tell the first window

if isRunning and hasNoWindows is false then

create window with default profile

end if

tell current session to write text command

end tell

end tell

end run
```