---
title: Transmit用iterm2替代终端
date: 2019-12-23 15:43:53
tags: 
  - mac 
  - transmit 
  - iterm2 
  - 终端
---
> mac下sftp工具transmit自带打开terminal采用的是默认，想要替换成iterm2



1. 首先创建一个文件随便什么名字 ,例如 *transmit-iterm-patch.applescript*，保存代码如下
2. 命令行运行 *defaults write com.panic.Transmit OpenTerminalScriptPath ~/transmit-iterm-patch.applescript*,++注意地址一定要写对++
3. 用transmit中的打开终端即可打开iterm2的界面

<!-- more -->
```
-- Run the following cmd to make Transmit marry iterm2 as its Terminal partner:
-- defaults write com.panic.Transmit OpenTerminalScriptPath ~/transmit-iterm-patch.applescript

on openTerminal(location, remoteHost, serverPort)
    -- Prepare sshCmd and cmd:
    set sshCmd to ""
    set cmd to "cd \"" & location & "\""

    if ((count of remoteHost) is greater than 0) then
        set sshCmd to "ssh " & remoteHost

        if (serverPort is greater than 0) then
            set sshCmd to sshCmd & " -p " & serverPort
        end if
    end if

    -- Check whether the application was open already.
    tell application "System Events"
        set wasOpen to (exists (processes where name is "iTerm")) or (exists (processes where name is "iTerm2"))
    end tell

    -- Do operations on iTerm.
    tell application "iTerm"
        activate

        -- Create window:
        if (count of windows) is 0 then
            create window with default profile
        else
            if wasOpen then
                tell first window to create tab with default profile
            end if
        end if

        -- Inside iterm2's window:
        tell first window
            if ((count of sshCmd) is greater than 0) then
                tell current session to write text sshCmd
                delay 0.5
            end if
        
        tell current session to write text cmd
        end tell
    end tell

end openTerminal

```


参考文档

[终端替换攻略](https://library.panic.com/transmit/transmit5/open-in-terminal/)

[脚本代码github地址](https://gist.github.com/donraj/86ee3507bd9ef37c5cc3b358264fc23f)