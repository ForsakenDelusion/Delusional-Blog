---
title: 记一次macOS配置
date: 2024-12-06 11:25:41 +08:00
filename: 2024-12-06-macOS-configuration
categories:
  - Misc
tags:
  - macOS
  - Environment
dir: Misc
share: true
---
参考[这篇文章](https://github.com/xiaolai/apple-computer-literacy/blob/main/start-from-terminal.md)，并结合自身经验进行配置

首先安装`Xcode Command Line Tools`

```bash
xcode-select --install
```

### 配置`brew`

此时还没有代理，装不了一些国外源的软件，所以我们先来配置`brew`，安装一些必备的软件

```bash
# 使用国内源安装
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

**部分代理软件已从`brew`上移除**，所以我们使用浏览器搜索并下载，这里我使用`clashx/pro`。接下来，我们来美化`zsh`

### 美化`ZSH`

```
wget https://gitee.com/pocmon/ohmyzsh/raw/master/tools/install.sh  chmod +x install.sh ./install.sh
```

现在，`ohmyzsh`就安装上去了
#### 配置终端代理

为了方便后续的步骤，我们开始配置终端代理

```bash
echo '
# >>>终端配置代理 START >>>
alias proxy="
    export http_proxy=socks5://127.0.0.1:7890;
    export https_proxy=socks5://127.0.0.1:7890;
    export all_proxy=socks5://127.0.0.1:7890;
    export no_proxy=socks5://127.0.0.1:7890;
    export HTTP_PROXY=socks5://127.0.0.1:7890;
    export HTTPS_PROXY=socks5://127.0.0.1:7890;
    export ALL_PROXY=socks5://127.0.0.1:7890;
    export NO_PROXY=socks5://127.0.0.1:7890;"
alias unproxy="
    unset http_proxy;
    unset https_proxy;
    unset all_proxy;
    unset no_proxy;
    unset HTTP_PROXY;
    unset HTTPS_PROXY;
    unset ALL_PROXY;
    unset NO_PROXY"

proxy
# <<< 终端配置代理 END <<<
' >> ~/.zshrc
```

我们开启代理，并打开局域网连接，之后再打开终端，就会自动通过代理访问终端。同时，我们也可以通过`unproxy`命令关闭终端代理，`proxy`打开终端代理

#### 安装p10k主题

首先要装上依赖字体

[安装文档](https://github.com/ryanoasis/nerd-fonts#font-installation)

不过我们有`brew`，可以直接`brew`下来，配置完代理，下载速度便会快很多。

```bash
brew search nerd # 查看有什么字体可以安装，我这里选择Hack Nerd
brew install font-hack-nerd-font
```

接下来我们安装`p10k`

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

在`~/.zshrc`中修改

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

执行`source ~/.zshrc`即可进入p10k配置，根据提示完成配置

#### zsh插件配置

下面是`zsh`插件

```bash
// 克隆两个插件
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

```bash
// 配置插件
plugins=(git zsh-syntax-highlighting zsh-autosuggestions z)
```

至此，我们的美化工作已经结束。

### 开启`HIDPI`（可选）

如果你和我一样也是2K屏的话，这个步骤要尽快完成，不然眼睛就要瞎了。
终端运行以下命令，根据提示完成配置。

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi.sh)"
```

### 开启电量控制（可选）

我们使用`batt`管控电量，手动控制充电与否，具体可以看[项目描述]((https://github.com/charlie0129/batt?tab=readme-ov-file#limit-battery-charge))

```bash
brew install batt
sudo brew services start batt
```

配置`batt`

```bash
batt limit 80
batt magsafe-led enable
batt disable-charging-pre-sleep enable
```

### 安装软件

```bash
# command line tools
brew install git	# MacOS 自带的 Apple Git 也不是不能用，但，替换掉已经成了习惯
brew install wget	# 比 curl 方便一点的下载工具
brew install tree	# 用来查询目录的树状结构
brew install pyenv # 管理python版本
brew install thefuck

# gui
brew install mos # 优化鼠标滚轮
brew install visual-studio-code # 微软出品的代码编辑器（基于 Google Atom）
brew install google-chrome
brew install microsoft-edge
brew install onedrive
brew install obsidian
brew instsall qq
brew install wechat
brew install discord
brew install neteasemusic
brew install easydict # 翻译软件
brew install orbstack # docker desktop上位替换
brew install chatbox # ai
brew install chatgpt
brew install iterm2
brew install onedriver
```

大概要下载这么多，其中`pyenv`安装完了之后，要将以下内容添加到`~/.zshrc`

```bash
# >>> pyenv配置 START >>>
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
# <<< pyenv配置 END <<<
```

同样的`thefuck`也需要进行配置

```bash
# the fuck配置
eval $(thefuck --alias)
```

接着，我们配置`java`环境，先去[azul官网](https://www.azul.com/downloads/?version=java-8-lts&os=macos&architecture=arm-64-bit&package=jdk#zulu)下载`java8`和`java21`，两个常用版本的`dmg`文件进行安装。

安装完了之后，将以下内容添加到`~/.zshrc`

```bash
# >>> Java环境配置 START >>>
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-21.jdk/Contents/Home # 优先使用jdk21
# export CPPFLAGS="-I/opt/homebrew/opt/openjdk@21/include" # 使得编译时能找到jdk21的头文件

function jdk8() {
    # 设置 JDK 8
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
    echo "Switched to JDK8"
}

function jdk21() {
    # 设置 JDK 21
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-21.jdk/Contents/Home
    echo "Switched to JDK21"
}
# <<< Java环境配置 END <<<
```

然后去下载`nvm`管理`nodejs`环境，因为`nvm`不推荐`homebrew`管理，所以我们去[官方仓库](https://github.com/nvm-sh/nvm)下载

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

使用这条命令，结束之后在`~/.zshrc`中加入以下配置

```bash
# >>> nvm配置 START >>>
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
# <<< nvm配置 END <<<
```

### 快捷的在文件夹下打开终端（可选）

打开Mac上的自动操作app，在合适位置新建文稿，并创建一个应用程序，在左侧`资源库->实用工具`中选择`运行AppleScript`，然后将以下内容粘贴进去（需要提前安装`Iterm2`，如果想用默认的`terminal`，可以稍作修改）

```
on run {input, parameters}

tell application "Finder"

set pathList to (quoted form of POSIX path of (folder of the front window as alias))

set command to "clear; cd " & pathList

end tell

tell application "System Events"

# some versions might identify as "iTerm2" instead of "iTerm"

set isRunning to (exists (processes where name is "iTerm")) or (exists (processes where name is "iTerm2"))

end tell

# 如果想用terminal，就把这里改成terminal
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

接下来，我们保存这个软件，打开访达找到这个软件所在位置，按住`cmd`并用鼠标将其拖拽到访达上面的快捷栏处，就可以实现快捷的在文件夹目录打开终端了。

最后给出我的`zshrc`配置

```
# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
# Initialization code that may require console input (password prompts, [y/n]
# confirmations, etc.) must go above this block; everything else may go below.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:$HOME/.local/bin:/usr/local/bin:$PATH

# Path to your Oh My Zsh installation.
export ZSH="$HOME/.oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time Oh My Zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="powerlevel10k/powerlevel10k"

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Uncomment the following line to use case-sensitive completion.
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment one of the following lines to change the auto-update behavior
# zstyle ':omz:update' mode disabled  # disable automatic updates
# zstyle ':omz:update' mode auto      # update automatically without asking
# zstyle ':omz:update' mode reminder  # just remind me to update when it's time

# Uncomment the following line to change how often to auto-update (in days).
# zstyle ':omz:update' frequency 13

# Uncomment the following line if pasting URLs and other text is messed up.
# DISABLE_MAGIC_FUNCTIONS="true"

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# You can also set it to another string to have that shown instead of the default red dots.
# e.g. COMPLETION_WAITING_DOTS="%F{yellow}waiting...%f"
# Caution: this setting can cause issues with multiline prompts in zsh < 5.7.1 (see #5765)
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git zsh-syntax-highlighting zsh-autosuggestions z)

source $ZSH/oh-my-zsh.sh

# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by Oh My Zsh libs,
# plugins, and themes. Aliases can be placed here, though Oh My Zsh
# users are encouraged to define aliases within a top-level file in
# the $ZSH_CUSTOM folder, with .zsh extension. Examples:
# - $ZSH_CUSTOM/aliases.zsh
# - $ZSH_CUSTOM/macos.zsh
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"

# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh

# --- 个人配置 ---


# the fuck配置
eval $(thefuck --alias)

# >>> Java环境配置 START >>>
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-21.jdk/Contents/Home # 优先使用jdk21
# export CPPFLAGS="-I/opt/homebrew/opt/openjdk@21/include" # 使得编译时能找到jdk21的头文件

function jdk8() {
    # 设置 JDK 8
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
    echo "Switched to JDK8"
}

function jdk21() {
    # 设置 JDK 21
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-21.jdk/Contents/Home
    echo "Switched to JDK21"
}
# <<< Java环境配置 END <<<

# >>>终端配置代理 START <<<
alias proxy="
    export http_proxy=socks5://127.0.0.1:7890;
    export https_proxy=socks5://127.0.0.1:7890;
    export all_proxy=socks5://127.0.0.1:7890;
    export no_proxy=socks5://127.0.0.1:7890;
    export HTTP_PROXY=socks5://127.0.0.1:7890;
    export HTTPS_PROXY=socks5://127.0.0.1:7890;
    export ALL_PROXY=socks5://127.0.0.1:7890;
    export NO_PROXY=socks5://127.0.0.1:7890;"
alias unproxy="
    unset http_proxy;
    unset https_proxy;
    unset all_proxy;
    unset no_proxy;
    unset HTTP_PROXY;
    unset HTTPS_PROXY;
    unset ALL_PROXY;
    unset NO_PROXY"

proxy
# <<< 终端配置代理 END <<<

# >>> pyenv配置 START >>>
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
# <<< pyenv配置 END <<<

# >>> nvm配置 START >>>
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
# <<< nvm配置 END <<<

# >>> llvm配置 START >>>
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
# homebrew clang, llvm
export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
# 个人新增头文件
export CPLUS_INCLUDE_PATH=/Users/yu/Documents/studyAbout/codeLearn/include:$CPLUS_INCLUDE_PATH
# 切换到 Homebrew 安装的 LLVM 工具链
function bclang() {
    export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
    # homebrew clang, llvm
    export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
    export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"

    echo "Switched to Homebrew LLVM Clang"
}

# 切换回 macOS 系统自带的 Clang
function dclang() {
    # 移除 Homebrew LLVM 路径
    export PATH=$(echo $PATH | sed 's|/opt/homebrew/opt/llvm/bin:||')
    # 清除 LLVM 相关的环境变量
    unset LDFLAGS
    unset CPPFLAGS
    echo "Switched to System Clang"
}
# <<< llvm配置 END <


# <<< 个人配置 <<<
```

至此，我们的配置便可以告一段落了。一些`brew`中没有的软件可以去[这个网站](https://www.macbed.com/)下载






