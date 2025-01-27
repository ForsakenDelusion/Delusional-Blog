---
title: 记一次macOS配置
date: 2024-12-06 11:25:41 +08:00
filename: 2024-12-06-macOS-configuration
categories:
  - misc
tags:
  - macOS
  - Environment
dir: misc
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

至此，我们的配置便可以告一段落了。一些`brew`中没有的软件可以去[这个网站](https://www.macbed.com/)下载






