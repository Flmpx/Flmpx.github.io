---
title: Windows 的 VS Code 运行 C 程序乱码问题
date: 2026-11-16 22:39:14
tags: vscode
updated: 2026-11-23 14:46:00
---

## 原因分析 

本质上就是 `Windows` 系统默认采用本地化设置, 我们这一般是 `GBK` 编码, 所以终端也是 `GBK` 编码, 而 `VSCode` 的文件编码一般默认采用 `UTF-8` 编码, 二者编码不一致, 自然就导致了乱码.  


## 解决方案

只要在每次终端启动的时候自动运行一行命令 `chcp 65001` 即可, 但这只是**临时**的, 重开一个终端就又得重新输入, 下面介绍自动运行这段命令的代码:  

在 `.vscode/settings.json` 中加入如下代码即可(没有这个文件就新建一个)    

```json
{
    "terminal.integrated.profiles.windows": {
        "PowerShell": {
            "source": "PowerShell",
            "icon": "terminal-powershell",
            "args": ["-NoLogo","-NoExit","/c","chcp 65001"]
        },
        "Command Prompt": {
            "path": [
                "cmd.exe"
            ],
            "args": ["/k","chcp","65001"],
            "icon": "terminal-cmd"
        },
        "Git Bash": {
            "source": "Git Bash",
            "icon": "terminal-git-bash"
        }
    },

    "terminal.integrated.defaultProfile.windows": "PowerShell", //Powershell优先
    "files.encoding": "utf8",
}
```

如果要全局设置, 按 `Ctrl + P`, 然后打开 `首选项: 打开用户设置(JSON)`, 如下图, 然后添加和上面相同的代码块就行了   

![](../../image/vscode/Pasted%20image%2020260214103113.png)


## 其他

> 说点别的, 其实直接使用 `Git Bash` 就不用担心这种乱码问题了, 但是前提你安装了 `Git`    

```json
{
    "terminal.integrated.defaultProfile.windows": "Git Bash"
}
```