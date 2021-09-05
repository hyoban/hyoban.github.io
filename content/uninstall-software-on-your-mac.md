---
title: 如何相对干净地卸载 mac 上的软件
date: 2021-08-26T22:41:20+08:00
summary: 强迫症福音
---

目前总结了三种方案

## 软件自带的卸载器

很多软件的安装包里就附有有卸载工具（如 virtual box），或者安装完成之后会有卸载程序（罗技的键盘驱动），这是比较理想的卸载方式。

<img width="600" alt="pgp suite 的安装包" src="https://user-images.githubusercontent.com/38493346/130984363-af72fa36-9ffb-4e4a-90f1-ade2dde61061.png">

## 🌟 [appcleaner](https://freemacsoft.net/appcleaner/)

它可以帮你自动的检测需要删除的文件，还可以开启自动检测模式，当你移动软件到🗑️里时，就会提示你删除软件附带的文件。

<img width="600" alt="使用 appcleaner 尝试卸载 vscode" src="https://user-images.githubusercontent.com/38493346/130984852-5dc827a7-4579-46a1-ae10-03a47963e8ed.png">

最为推荐。

## [brew](https://brew.sh/)

关于如何安装 brew 的细节，这里不多赘述。首先为 brew 加上 cask 的源：

    brew tap Homebrew/homebrew-cask

然后就可以使用 brew 来卸载软件了，即使不是用 brew 安装的。

    brew rm -f --zap --cask <app name>

-f 表示强制执行，不管是否安装（我们只是想要使用它提供的脚本），zap 可以理解为强力清理。

<img width="600" alt="在终端使用 brew 执行卸载脚本" src="https://user-images.githubusercontent.com/38493346/130985443-5a2e834f-bac4-43d8-bc56-233ed05b32b9.png">
