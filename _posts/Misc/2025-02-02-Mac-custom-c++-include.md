---
title: Mac中添加自定义的C++库
date: 2025-02-02 16:24:50 +08:00
filename: 2025-02-02-Mac-custom-c++-include
categories:
  - Misc
tags:
  - C
  - macOS
dir: Misc
share: true
---

## 起因

环境前提，`macOS15.1`，`clang19.1.17`

最近学习acm的时候发现万能头文件`bits/stdc++.h`被提到的很频繁，于是尝试用了一下，发现找不到头文件，编译也出错。想了一下大概是`clang`不自带这个头文件。于是想以一个优雅一点的方式（不修改原本的库文件，而是选择模块化的新增库）添加支持。

## 做法

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