---
title: 算法竞赛ACM等VSCODE配置
date: 2025-03-25 22:25:49 +08:00
filename: 2025-03-25-vscode-configuration-of-acm
categories:
  - Misc
tags:
  - ACM
dir: Misc
share: true
---
仅作为个人记录。

主要使用Clangd，Codelldb，clang-format。

以下是debug设置。


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
            "terminal": "integrated",
            "preLaunchTask": "C++ Build",
            "internalConsoleOptions": "neverOpen"
        },
        {
            "name": "C++ Debug with Input File",
            "type": "lldb",
            "request": "launch",
            "program": "${fileDirname}/build/${fileBasenameNoExtension}",
            "args": ["<", "${fileDirname}/input.txt"],
            "cwd": "${fileDirname}",
            "terminal": "integrated", 
            "preLaunchTask": "C++ Build",
            "internalConsoleOptions": "neverOpen"
        },
        {
            "name": "C++ Debug with Input/Output Files",
            "type": "lldb",
            "request": "launch",
            "program": "${fileDirname}/build/${fileBasenameNoExtension}",
            "args": ["<", "${fileDirname}/input.txt", ">", "${fileDirname}/output.txt"],
            "cwd": "${fileDirname}",
            "terminal": "integrated",
            "preLaunchTask": "C++ Build",
            "internalConsoleOptions": "neverOpen"
        }
    ]
}
```

```json
//.vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "C++ Build",
            "type": "shell",
            "command": "clang++",
            "args": [
                "-std=c++17",
                "-stdlib=libc++",
                "-g",
                "-Wall",
                "-Wextra",
                "${file}",
                "-o",
                "${fileDirname}/build/${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "reveal": "silent",
                "panel": "shared"
            },
            "problemMatcher": "$gcc"
        }
    ]
}
```

```json
//.vscode/setting.json
{
    "code-runner.executorMap": {

        "javascript": "node",
        "java": "cd $dir && javac $fileName && java $fileNameWithoutExt",
        "c": "cd $dir && gcc $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "zig": "zig run",
        "cpp": "cd $dir && clang++ $fileName -o $dir/build/$fileNameWithoutExt && $dir/build/$fileNameWithoutExt",
        "objective-c": "cd $dir && gcc -framework Cocoa $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "php": "php",
        "python": "python -u",
        "perl": "perl",
        "perl6": "perl6",
        "ruby": "ruby",
        "go": "go run",
        "lua": "lua",
        "groovy": "groovy",
        "powershell": "powershell -ExecutionPolicy ByPass -File",
        "bat": "cmd /c",
        "shellscript": "bash",
        "fsharp": "fsi",
        "csharp": "scriptcs",
        "vbscript": "cscript //Nologo",
        "typescript": "ts-node",
        "coffeescript": "coffee",
        "scala": "scala",
        "swift": "swift",
        "julia": "julia",
        "crystal": "crystal",
        "ocaml": "ocaml",
        "r": "Rscript",
        "applescript": "osascript",
        "clojure": "lein exec",
        "haxe": "haxe --cwd $dirWithoutTrailingSlash --run $fileNameWithoutExt",
        "rust": "cd $dir && rustc $fileName && $dir$fileNameWithoutExt",
        "racket": "racket",
        "scheme": "csi -script",
        "ahk": "autohotkey",
        "autoit": "autoit3",
        "dart": "dart",
        "pascal": "cd $dir && fpc $fileName && $dir$fileNameWithoutExt",
        "d": "cd $dir && dmd $fileName && $dir$fileNameWithoutExt",
        "haskell": "runghc",
        "nim": "nim compile --verbosity:0 --hints:off --run",
        "lisp": "sbcl --script",
        "kit": "kitc --run",
        "v": "v run",
        "sass": "sass --style expanded",
        "scss": "scss --style expanded",
        "less": "cd $dir && lessc $fileName $fileNameWithoutExt.css",
        "FortranFreeForm": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "fortran-modern": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "fortran_fixed-form": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "fortran": "cd $dir && gfortran $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "sml": "cd $dir && sml $fileName",
        "mojo": "mojo run",
        "erlang": "escript",
        "spwn": "spwn build",
        "pkl": "cd $dir && pkl eval -f yaml $fileName -o $fileNameWithoutExt.yaml",
        "gleam": "gleam run -m $fileNameWithoutExt"
    }
}
```

```
//.clang-format
# 风格：Google, LLVM, Chromium, Mozilla, WebKit, Microsoft, GUN
BasedOnStyle: Google
# 语言: None, Cpp, Java, JavaScript, ObjC, Proto, TableGen, TextProto
Language: Cpp
# 标准: Cpp03, Cpp11, Auto
Standard: Auto
# 去除C++11的列表初始化的大括号{后和}前的空格
Cpp11BracedListStyle: true
# 访问说明符(public、private等)的偏移
AccessModifierOffset: -4
# 开括号(开圆括号、开尖括号、开方括号)后的对齐: Align, DontAlign, AlwaysBreak(总是在开括号后换行)
AlignAfterOpenBracket: AlwaysBreak
# 水平对齐二元和三元表达式的操作数
AlignOperands: false
# 允许函数声明的所有参数在放在下一行
AllowAllParametersOfDeclarationOnNextLine: false
# 允许短的块放在同一行
AllowShortBlocksOnASingleLine: false
# 允许短的case标签放在同一行
AllowShortCaseLabelsOnASingleLine: false
# 允许短的函数放在同一行: None, InlineOnly(定义在类中), Empty(空函数), Inline(定义在类中，空函数), All
AllowShortFunctionsOnASingleLine: Empty
# 允许短的if语句保持在同一行
AllowShortIfStatementsOnASingleLine: false
# 允许短的循环保持在同一行
AllowShortLoopsOnASingleLine: false
# 总是在返回类型后换行: None, All, TopLevel(顶级函数，不包括在类中的函数),  # AllDefinitions(所有的定义，不包括声明), TopLevelDefinitions(所有的顶级函数的定义)
AlwaysBreakAfterReturnType: None
# 总是在template声明后换行
AlwaysBreakTemplateDeclarations: true
# false表示函数实参要么都在同一行，要么都各自一行
BinPackArguments: false
# false表示所有形参要么都在同一行，要么都各自一行
BinPackParameters: false
# 构造函数的初始化列表换行
BreakConstructorInitializers: AfterColon
# 每行字符的限制，0表示没有限制
ColumnLimit: 120
# 构造函数的初始化列表的缩进宽度
ConstructorInitializerIndentWidth: 8
# 延续的行的缩进宽度
ContinuationIndentWidth: 4
# 继承最常用的指针和引用的对齐方式
DerivePointerAlignment: true
# 为命名空间添加缺失的命名空间结束注释
FixNamespaceComments: true
# 缩进case标签
IndentCaseLabels: false
# 使用tab字符: Never, ForIndentation, ForContinuationAndIndentation, Always
UseTab: Never
# 缩进宽度
IndentWidth: 4
# 连续空行的最大数量
MaxEmptyLinesToKeep: 1
# 命名空间的缩进: None, Inner(缩进嵌套的命名空间中的内容), All
NamespaceIndentation: None
PenaltyBreakAssignment: 2
# 在call(后对函数调用换行的penalty
PenaltyBreakBeforeFirstCallParameter: 1
# 在一个注释中引入换行的penalty
PenaltyBreakComment: 500
# 第一次在<<前换行的penalty
PenaltyBreakFirstLessLess: 120
# 在一个字符串字面量中引入换行的penalty
PenaltyBreakString: 1000
# 对于每个在行字符数限制之外的字符的penalty
PenaltyExcessCharacter: 1000000
# 将函数的返回类型放到它自己的行的penalty
PenaltyReturnTypeOnItsOwnLine: 400
# 指针和引用的对齐: Left, Right, Middle
PointerAlignment: Right
# 允许排序#include
SortIncludes: false
# 连续宏定义的值对齐
AlignConsecutiveMacros: true
```

```
//.gitignore
*

!*.cpp
!.vscode
```