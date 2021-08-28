---
title: mac 补全计划
date: 2021-08-10T17:31:20+08:00
summary: 拿到一台新的 mac 需要做哪些设置呢
---

## 软件列表

### 系统补全

- [hackintool](https://github.com/headkaze/Hackintool) 便于替换 efi，修复睡眠
- [alt tab](https://github.com/lwouis/alt-tab-macos) 提高切换软件体验
- [mos](https://github.com/Caldis/Mos) 鼠标平滑
- [appcleaner](https://freemacsoft.net/appcleaner/) 卸载软件残留

### 必备工具

- [clashx](https://github.com/yichengchen/clashX) or [clashx pro](https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public) 代理软件，pro可以作为旁路由
- [sublime text](https://www.sublimetext.com/)
- [chrome](https://www.google.com/chrome/)

### 开发工具

- [android studio](https://developer.android.com/studio/archive)
- [idea](https://www.jetbrains.com/idea/download)
- [vscode](https://github.com/microsoft/vscode)
- [github desktop](https://desktop.github.com/)

### 其他

- [netnewswire](https://github.com/Ranchero-Software/NetNewsWire) rss 订阅器，支持 iCloud 同步，订阅 twitter
- [telegram](https://github.com/telegramdesktop/tdesktop) app store 的版本更加流畅。
- [logitech options](https://support.logi.com/hc/zh-cn/articles/360025297893) 或者 [直接下载 zip](https://download01.logi.com/web/ftp/pub/techsupport/options/Options_8.54.147.zip) （新版本会多安装一个 pop，注意安装老版本）

## 重置指南（nuc 8i5beh 限定）

我有比较严重的强迫症，时常会想要去重装系统，所以就有了下面设置系统的流程。

### 注意事项

**重装系统前需要退出设备和网页上的 apple 设备，否则会导致设备无法收到更新。**（原因是没有手动替换三码）

开启 hidpi时，第一项选择 `(2) Enable HIDPI (with EDID)`

### 显示器

切换缩放，设置完成 hidpi 重启后再调整回来。（可选，如果你的显示器默认可以开启 hidpi 的话）

### mos

1. 设置鼠标平滑
2. 开机自启和隐藏图标

### hackintool

1. 替换 efi
2. 修复睡眠

### clash x pro

1. 安装 clashx pro
2. 添加订阅
3. 设置为系统代理
4. 设置开机启动

### terminal

1. 终端修改字体大小
2. 下载 brew
3. 开启 hidpi

### 快捷键

1. 替换 capslock 为 control
2. 设置快捷键

### 访达

1. 文件按照名称排序
2. 隐藏桌面图标
3. 设置启动文件夹
4. 侧边栏隐藏标签，调整展示选项
5. 显示文件扩展名
6. 桌面图标按网格放置

### dock 和 菜单栏

1. 设置 dock 放大
2. 移除 dock 图标
3. 自动隐藏
4. 移除右上角软件图标

### other

1. 设置自动登入，
2. 屏幕常亮
3. 触发角设置左下为启动台，右下为任务管理
4. 设置日期时间格式
5. 关闭开机恢复软件
6. 重启
7. 登录 apple 账号

## 终端设置

### 设置代理

```sh
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

### 安装 brew

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 开启 hidpi

```sh
bash -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi.sh)"
```

### 终端 hostname 显示修正

（可选）**路由器设置自定义 dns 即可**，一般可选 8.8.8.8（谷歌，国外）和 114.114.114.114（国内），也有说国内实际连接不上谷歌的 dns，已经被污染了。

常见的出错情况为显示 bogon 或者 192，解决办法是手动指定 dns 而非使用路由器默认的 dns 服务，也就是 0.0.0.0。

通常情况下，路由器默认开启 dhcp 服务，自动分配 ip 地址的同时，为设备转发 dns。所以设置在路由器或者本机的网络设置里都可以，在本机设置的话，设备就无视路由器的 dns 转发。

> terminal 显示 hostname 之前会先根据本机 IP 做一次 rDNS 反向查询，就是通过 ip 地址查询 hostname，过程与 DNS 类似。
> rDNS 反向查询常用在 traceroute 以及反垃圾邮件技术中，terminal 显示查询到的 hostname，如果没有查询到，那么使用本机设置的 hostname。
> 本机 IP 通常是局域网 IP 地址（保留IP地址），一般是查不到的，所以 terminal 一般显示的本机设置的 hostname，比如dangchujiubugaixiafan's-macbook。
> 上面提到，局域网 IP 地址一般是查不到 hostname，是因为 ISP 提供商或者用户防火窗的屏蔽保留 IP 地址，因为保留 IP 地址在公网中没啥用，即便是没有被屏蔽掉，rDNS 服务器一般也会关闭响应保留IP地址的查询请求。凡事都有例外，rDNS 服务器对这种保留 IP 地址对查询一律返回 bogon。在 ipv4 对地址划分中，除了公网分配在用对 IP 地址外，其余保留 IP 地址统一叫做bogon space。

#### 参考

1. [Mac修复hostname被篡改为bogon](https://zhuanlan.zhihu.com/p/55827741)
2. [问：mac终端用户名不知何故变成了192](https://discussionschinese.apple.com/thread/76800?answerId=140246186322#140246186322)
3. [DNS 与路由器设置马克](https://zhuanlan.zhihu.com/p/105942315)

### [oh my zsh](https://github.com/ohmyzsh/ohmyzsh)

注意先让终端启用代理

安装本体

```sh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装插件

```sh
cd ~/.oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions.git
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
```

配置 zshrc

```sh
# fix pip command in zsh
alias pip='noglob pip'

# set proxy
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890

# nvm
export NVM_DIR="$HOME/.nvm"
  [ -s "/usr/local/opt/nvm/nvm.sh" ] && . "/usr/local/opt/nvm/nvm.sh"  # This loads nvm
  [ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && . "/usr/local/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion

# set plugins
plugins=(git zsh-autosuggestions zsh-syntax-highlighting sublime)

# set adb
export ANDROID_SDK_ROOT="/Users/hyoban/Library/Android/sdk"
export PATH="$ANDROID_SDK_ROOT/platform-tools:$PATH"
```

#### 主题

1. [Dracula](https://draculatheme.com/terminal) [直接下载](https://github.com/dracula/terminal-app/archive/master.zip)
2. 设置字体为 Monaco，字号为16

### git 配置

#### 安装 github desktop 

设置默认分支为 master

#### 设置全局的 git ignore

[添加设置](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files#configuring-ignored-files-for-all-repositories-on-your-computer)

    git config --global core.excludesfile ~/.gitignore_global

完成后的 .gitconfig 有如下内容

    [core]
    	excludesfile = /Users/<username>/.gitignore_global

添加需要排除的文件

这里有一份 [mac 的示例](https://github.com/github/gitignore/blob/master/Global/macOS.gitignore)

[参考](https://twitter.com/github/status/1423663917881077760)

## brew list

### [neofetch](https://github.com/dylanaraps/neofetch)

显示系统信息，不推荐 screenfetch，目前 macOS 11 识别有问题，参考 KittyKatt/screenFetch/issues/716

    brew install neofetch

### ~~[github cli](https://cli.github.com)~~

便于终端环境配置 github，mac 无需使用，Github desktop 即可

    brew install gh

### [go](https://github.com/golang/go)

    brew install go

### [ktlint](https://github.com/pinterest/ktlint)

配置 kotlin 代码格式，此外这会依赖于 openjdk 11，可以顺手配置 java 环境。

    brew install ktlint
    sudo ln -sfn $(brew --prefix)/opt/openjdk@11/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-11.jdk

### ~~gpg~~

目前主要用于 github commit 认证

    brew install gnupg
