# Chapter 3 describes tools for using the shell

There are several different shells to choose from on Mac OS X systems. In the past, tcsh was used as the default shell. 

More recently, the **Bourne Again Shell (bash)** has been the default.?并不是了

`echo $SHELL` 可以查看当前shell 

目前系统是`/bin/zsh`

# Using the Shell

shell的设置保存在用户目录 `.zprofile`

## Using bash history

- shell的命令历史保存在 `～/.zsh_history`
- This file is set by the value of `$HISTFILE`.

![Untitled](Chapter%203%20describes%20tools%20for%20using%20the%20shell%2047cd2fc6a59d421a93ddb7b97c161d50/Untitled.png)

- 用`history`来显示之前的命令

## redirecting stdin and stdout

Like most UNIX systems, Mac OS X supports stdin, stdout, and stderror.

`2>` 可以将stderror进行重导向

`>>` 将导向内容添加而不是覆盖

Using **pipes**, you can **redirect output from one process to another process,** 
`# **ls /tmp | sort**`

## Acquiring Super User power

On Mac OS X, the root user is disabled by default. If you need to do something as root, use the sudo command.

`$ **sudo /usr/bin/less /var/log/apache2/error_log**`

系统会保持5分钟不需要输入密码

## Using environment Variables

Small chunks of information that are useful to your shell environment are stored in
what are referred to as ***environment variables*.** 
By convention, environment variable names are all **uppercase** (although that convention is not enforced).
`set`可以显示当前的环境变量

- 直接利用=可以创建新的变量

```bash
$ ABC=123
$ echo $ABC 123
```

> 但是所有的变量都只存在于创建它的shell当中。如果新建一个shell就看不到了
> 
- 利用`export`将变量变成环境变量并且可以被子进程继承？还可以链式添加字串

```bash
export PATH=$PATH:/home/fcaen //常用来添加路径
```

- `env` 可以查看环境变量

# Creating Simple Shell Scripts

Shell scripts are good for automating repetitive shell tasks. Bash and other shells include the basic constructs found in various programming languages, such as loops, tests, case statements, and so on. The main difference is that **there is only one type of variable: strings.**

### 编辑和运行脚本

- 脚本其实就是文本。`.sh`结尾
- 脚本要运行，需要先给予权限

```bash
$ chmod u+x myscript.sh
```

- 如果不将其设置成可执行，在必须是通过shell命令来运行，`bash [myscript.sh](http://myscript.sh)` 并且第一行需要添加如下

```bash
#!/bin/bash
```

- 脚本执行时候有可能出现 command not found错误提示，这时候需要将脚本的路径添加到环境变量，或者添加完整路径

```bash
$ mkdir ~/bin ; cp myscript.sh ~/bin/ ; PATH=$PATH:~/bin //添加路径到环境变量
$ cp myscript.sh /usr/local/bin //拷贝到已存在路径
$ ./myscript.sh //在当前目录执行
$ /tmp/myscript.sh //完整路径
```

### 编辑脚本内容

略……