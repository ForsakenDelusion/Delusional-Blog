---
title: macOS自用c环境配置指南
date: 2024-10-06 00:33:18 +08:00
filename: 2024-10-06-c-env-configuration
categories:
  - Misc
tags:
  - Environment
  - C
dir: Misc
share: true
---
## 关于clangd

系统官方的介绍可以看网上别的博客，按我个人理解，clangd就是给你提供代码补全，函数跳转之类的功能。

## 开始配置

我是MacOS，所以先直接`brew install llvm`，如果你是Debian系Linux，用`apt install clang clangd clangd-tidy llvm`即可

macOS brew的时候会出现

```
CLANG_CONFIG_FILE_SYSTEM_DIR: /opt/homebrew/etc/clang
CLANG_CONFIG_FILE_USER_DIR:   ~/.config/clang

LLD is now provided in a separate formula:
  brew install lld

We plan to build LLVM 20 with `LLVM_ENABLE_EH=OFF`. Please see:
  https://github.com/orgs/Homebrew/discussions/5654

Using `clang`, `clang++`, etc., requires a CLT installation at `/Library/Developer/CommandLineTools`.
If you don't want to install the CLT, you can write appropriate configuration files pointing to your
SDK at ~/.config/clang.

To use the bundled libunwind please use the following LDFLAGS:
  LDFLAGS="-L/opt/homebrew/opt/llvm/lib/unwind -lunwind"

To use the bundled libc++ please use the following LDFLAGS:
  LDFLAGS="-L/opt/homebrew/opt/llvm/lib/c++ -L/opt/homebrew/opt/llvm/lib/unwind -lunwind"

NOTE: You probably want to use the libunwind and libc++ provided by macOS unless you know what you're doing.

llvm is keg-only, which means it was not symlinked into /opt/homebrew,
because macOS already provides this software and installing another version in
parallel can cause all kinds of trouble.

If you need to have llvm first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> ~/.zshrc

For compilers to find llvm you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
```

这是提示你macOS里面的clang和brew下来的clang冲突了，往`~/.zshrc`中加入如下语句即可。

```bash
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
# homebrew clang, llvm
export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
```

接下来我们去vscode里面下载插件，通常是`clangd`，`CodeLLDB`，`CMake Tools`这几个一起下。

然后去clangd插件中配置参数，启用一些特性

```json
"clangd.arguments": [
        "--enable-config",
        "--background-index",
        "--compile-commands-dir=build",
        "--clang-tidy",
        "--all-scopes-completion",
        "--completion-style=detailed",
        "--header-insertion=iwyu",
        "--pch-storage=disk",
        "-j=12"
    ],
```

然后去项目根目录，新增clangd配置，以下内容放在根目录下的`.clangd`中，要注意，clangd的用户级配置的优先级是大于项目配置的，这一点非常反人类。

用户级配置：

- _Windows_: , typically .`%LocalAppData%\clangd\config.yaml``C:\Users\Bob\AppData\Local\clangd\config.yaml`

- _macOS_: `~/Library/Preferences/clangd/config.yaml`

- _Linux and others_: , typically .`$XDG_CONFIG_HOME/clangd/config.yaml``~/.config/clangd/config.yaml`

> 如果你找不到这些路径，请手动创建

具体参数可以看一下[clang官网](https://clangd.llvm.org/config)，[clang-tidy配置](https://clang.llvm.org/extra/clang-tidy/checks/list.html)

然后我们配置lldb，用于调试程序

在`vscode`中创建`lanuch.json`，然后添加配置，选择`CodeLLDB: Launch`，稍微修改一下

```json
"version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "lldb",
            "request": "launch",
            "program": "${workspaceFolder}/build/${fileBasenameNoExtension}",
            "args": [],
            "cwd": "${workspaceFolder}",
        }
    ]
```

这样即可。按下F5调试代码，但是前提是你编译的文件在根目录的`build`文件下

## 安装gcc

```shell
brew install gcc
```

然后在`~/.zshrc`里面修改原本macOS的alias（原本苹果把gcc那套alias成了clang）
```shell
# ~/.zshrc
# gcc设置
alias gcc=gcc-14 # gcc-14换成你实际安装上的gcc版本
alias g++=g++-14
```

## clangd用gcc的视角来看代码

有些时候我们只想用clangd做语法检测，所以可以换gcc做编译，并且可以以gcc的头文件库来进行语法检测。
只需要像如下修改`clangd/config.yaml`即可

```yaml
# Fragment specific to C++ source files
If:
    PathMatch: [.*\.cpp, .*\.cxx, .*\.cc, .*\.h, .*\.hpp, .*\.hxx]
CompileFlags:
  Compiler: /opt/homebrew/bin/g++-14 # 这里换成你的g++地址
  Add:
    - "-std=c++17"
---
# Fragment specific to C source files
If:
    PathMatch: [.*\.c]
CompileFlags:
  Compiler: /opt/homebrew/bin/gcc-14 # 这里换成你的gcc地址
  Add:
    - "-std=c99"

```