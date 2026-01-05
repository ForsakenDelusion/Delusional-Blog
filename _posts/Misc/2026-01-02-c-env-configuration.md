---
title: macOS自用c环境配置指南
date: 2026-01-02 00:33:18 +08:00
filename: 2026-01-02-c-env-configuration
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

## vscode debug设置

### 添加debug前的预编译（pre tasks）

macOS按住cmd+shift+p打开设置菜单，输入'Open User Tasks'，填入如下代码，分别是用clang和gcc工具链编译源文件，并放在目录下的build文件夹下。
```json
//.vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "G++ Build",
            "type": "shell",
            "command": "g++-14",
            "args": [
                "-std=c++17",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/build/${fileBasenameNoExtension}"
            ],
            "dependsOn": "Make Build Directory",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "reveal": "silent",
                "panel": "shared"
            },
            "problemMatcher": "$gcc"
        },
          {
            "label": "Clang++ Build",
            "type": "shell",
            "command": "clang++",
            "args": [
                "-std=c++17",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/build/${fileBasenameNoExtension}"
            ],
            "dependsOn": "Make Build Directory",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "reveal": "silent",
                "panel": "shared"
            },
            "problemMatcher": "$gcc"
        },
        {
            "label": "Make Build Directory",
            "type": "shell",
            "command": "mkdir",
            "args": ["-p", "${fileDirname}/build"],
            "group": "build"
        }
    ]
}
```

接着配置`.vscode/launch.json`，这个是debug配置，由于macOS配置gdb很麻烦，我们直接采用lldb进行配置。注意下面的preLaunchTask是可以用不同的build的，参考上面Tasks的设置，因为我要用gcc的万能头，所以用gcc工具链

```json
//.vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "C++ Debug (LLDB)",
            "type": "lldb",
            "request": "launch",
            "program": "${fileDirname}/build/${fileBasenameNoExtension}",
            "args": [],
            "cwd": "${fileDirname}",
            "preLaunchTask": "G++ Build",
            "internalConsoleOptions": "neverOpen"
        },
        
    ]
}
```

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

### macOS添加自定义头文件（万能头）


### 起因

环境前提，`macOS15.1`，`clang19.1.17`

最近学习acm的时候发现万能头文件`bits/stdc++.h`被提到的很频繁，于是尝试用了一下，发现找不到头文件，编译也出错。想了一下大概是`clang`不自带这个头文件。于是想以一个优雅一点的方式（不修改原本的库文件，而是选择模块化的新增库）添加支持。

### 做法

不想写乱七八糟的东西了，直接给出做法。

在你喜欢的位置新增目录，比如在你的家目录下使用

```shell
mkdir -p include/bits
cd include/bits
vim stdc++.h
```

然后再将以下文件粘贴进去

```c++
// C++ includes used for precompiling -*- C++ -*-

// Copyright (C) 2003-2014 Free Software Foundation, Inc.
//
// This file is part of the GNU ISO C++ Library.  This library is free
// software; you can redistribute it and/or modify it under the
// terms of the GNU General Public License as published by the
// Free Software Foundation; either version 3, or (at your option)
// any later version.

// This library is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.

// Under Section 7 of GPL version 3, you are granted additional
// permissions described in the GCC Runtime Library Exception, version
// 3.1, as published by the Free Software Foundation.

// You should have received a copy of the GNU General Public License and
// a copy of the GCC Runtime Library Exception along with this program;
// see the files COPYING3 and COPYING.RUNTIME respectively.  If not, see
// <http://www.gnu.org/licenses/>.

/** @file stdc++.h
 *  This is an implementation file for a precompiled header.
 */

// 17.4.1.2 Headers

// C
#ifndef _GLIBCXX_NO_ASSERT
#include <cassert>
#endif
#include <cctype>
#include <cerrno>
#include <cfloat>
#include <ciso646>
#include <climits>
#include <clocale>
#include <cmath>
#include <csetjmp>
#include <csignal>
#include <cstdarg>
#include <cstddef>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>

#if __cplusplus >= 201103L
#include <ccomplex>
#include <cfenv>
#include <cinttypes>
#include <cstdbool>
#include <cstdint>
#include <ctgmath>
#include <cwchar>
#include <cwctype>
#endif

// C++
#include <algorithm>
#include <bitset>
#include <complex>
#include <deque>
#include <exception>
#include <fstream>
#include <functional>
#include <iomanip>
#include <ios>
#include <iosfwd>
#include <iostream>
#include <istream>
#include <iterator>
#include <limits>
#include <list>
#include <locale>
#include <map>
#include <memory>
#include <new>
#include <numeric>
#include <ostream>
#include <queue>
#include <set>
#include <sstream>
#include <stack>
#include <stdexcept>
#include <streambuf>
#include <string>
#include <typeinfo>
#include <utility>
#include <valarray>
#include <vector>

#if __cplusplus >= 201103L
#include <array>
#include <atomic>
#include <chrono>
#include <condition_variable>
#include <forward_list>
#include <future>
#include <initializer_list>
#include <mutex>
#include <random>
#include <ratio>
#include <regex>
#include <scoped_allocator>
#include <system_error>
#include <thread>
#include <tuple>
#include <typeindex>
#include <type_traits>
#include <unordered_map>
#include <unordered_set>
#endif


```

保存完之后退出，记下你的`include`路径，比如我的就是`User/yu/include`

然后再将此路径添加到环境变量中，我使用`zsh`于是编辑`~/.zshrc`，将以下代码添加进去

```shell
export CPLUS_INCLUDE_PATH=/Users/yu/include:$CPLUS_INCLUDE_PATH
```

这样即可找到新增的库文件。

并且vscode每次进入的时候会自动加载环境变量，这个时候会把我们在zsh中添加的PATH给载入，所以如果用clangd做lsp，实际上vscode里面的语法高亮和头文件也是能work的。