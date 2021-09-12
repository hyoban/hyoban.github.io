---
title: python 虚拟环境
date: 2021-06-15T17:31:20+08:00
summary: 隔离项目和系统的 python 环境
---

macOS 中默认安装了 python，但是是 2.x 的版本，通常我们需要手动安装 python3。
安装之后，我们需要使用 python3 和 pip3 对应的命令，而且把包全安装到系统环境下，好像也不是很优雅。

## 虚拟环境

创建一个虚拟环境，-m 参数表示在 sys.path 中搜索指定模块，并以 `__main__` 模块执行其内容。

    python3 -m venv venv

激活虚拟环境(on macos)

    source venv/bin/activate

## pip 管理包

以下 <pkn> 表示 package name。

安装

    pip install [--upgrade] <pkn> [= 1.0.0]

卸载

    pip uninstall <pkn>

显示详情

    pip show <pkn>

显示所有包

    pip list

生成依赖安装文件

    pip freeze > requirements.txt

一键安装

    pip install -r requirements.txt

你可以在 pip 文档中查看更多信息。

## 参考

- [命令行与环境](https://docs.python.org/zh-cn/3/using/cmdline.html)
- [虚拟环境和包](https://docs.python.org/zh-cn/3/tutorial/venv.html)
- [pip 文档](https://pip.pypa.io/en/stable/)
