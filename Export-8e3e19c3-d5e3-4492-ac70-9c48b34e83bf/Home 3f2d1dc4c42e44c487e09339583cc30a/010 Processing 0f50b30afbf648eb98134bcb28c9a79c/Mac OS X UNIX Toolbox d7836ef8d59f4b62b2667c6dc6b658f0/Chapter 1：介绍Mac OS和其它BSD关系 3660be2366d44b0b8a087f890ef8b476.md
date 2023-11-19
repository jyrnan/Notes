# Chapter 1：介绍Mac OS和其它BSD关系

Mac OS X is a modern operating system that combines the power of a UNIX-based operating system and the simplicity and elegance of the Macintosh UI. Its open- source core, called Darwin, is a direct descendant of various BSD projects. It is based in part on BSD 4.4 Lite, but many libraries and utilities are from FreeBSD and NetBSD. Because of this history, it’s useful to delve into the history of BSD.

一句话：Mac OS 前身叫Darwin，来源于BSD

BSD’ well earned reputation:

- Stability
- Security

# Finding Commands

reason for “command not found”

- mistyped the command name
- command not in your PATH
- may need to be the root user for the command
- command is not installed on your computer

![Untitled](Chapter%201%EF%BC%9A%E4%BB%8B%E7%BB%8DMac%20OS%E5%92%8C%E5%85%B6%E5%AE%83BSD%E5%85%B3%E7%B3%BB%203660be2366d44b0b8a087f890ef8b476/Untitled.png)

## 获取帮助的三个途径

- `—help`
- `man`
- `info`

### 获取帮助的帮助

The -h or --help options are often used to **display help messages** for a command. 
`$ **man --help**`

## using man pages

- man pages有很多类型，分为多个章节，代表不同类。最重要是第一类：命令
- 一个查找内容可能会有多个章节都有相关内容。

![Untitled](Chapter%201%EF%BC%9A%E4%BB%8B%E7%BB%8DMac%20OS%E5%92%8C%E5%85%B6%E5%AE%83BSD%E5%85%B3%E7%B3%BB%203660be2366d44b0b8a087f890ef8b476/Untitled%201.png)

最简单方式是是 `man find`

可以使用 `whatis` 会列出命令的简要形式

### using info Documents

`info ls`