# 使用 github 作为知识的内化和输出平台

## markdown 支持相关

首先我想介绍一下 github 对于在线编写 markdown 的支持。

1. 预览样式（也就是你现在看到的样式）很好看
2. 自带图床。只需要将图片拖拽到编辑器中即可上传图片，就会生成对应的 img 标签。
3. 简洁的快捷键支持。比如可以使用 cmd + k 快速插入链接
4. vscode 在线版。任意仓库，只需要按下 . 即可启动在线版的 vscode，并且通过 github 账户同步你的 vscode 设置。

## 内化

这部分是比较简单的，只需要开通仓库的 issues 即可，使用示例可以参考本仓库的 issues。

总体来说，它以下的优点使我选择了它。

1. 出色的 markdown 支持。
2. 双向链接，只需使用 # 即可链接其他的 issue，同时被链接到的位置会提示链接。
3. 标签支持。
4. 控制是否归档，只需要 close ，当前 issue 即可。

## 输出

在这个仓库中，我使用 hugo 配合 github action 以及 github pages，搭建了一个完全自动化的在线博客。每次编辑完仓库的文章之后它就会自动帮我生成对应的站点，我无需管理网站的状态。

同时，它具有最完美的编辑体验。

1. 完全在线化，无需配置本地环境，或是使用 typora 之类的 markdown 编辑器。
2. 接上条，编辑完成之后可以直接保存在线提交，免去本地手动提交，push 到 github 的过程。
3. 自带版本控制，我们文章内容何时做过怎样的修改一目了然

### hugo

所有文章放在 content 目录下，static 文件夹目前只用于存放站点图标。

主题为 cayman，我自己稍作了修改，主要是首页按照日期分组展示，具体实现如下：

```
{{ define "main" }} 
{{ $pages := where site.RegularPages "Type" "in" site.Params.mainSections }} 
{{ range $pages.GroupByDate "2006" }}
<div>
  <h2>{{ .Key }}</h2>
  <ul>
    {{ range .Pages}}
    <li>
      {{ .Date.Format "Jan 2" }} <a href="{{ .Permalink }}">{{ .Title }}</a>
    </li>
    {{ end }} 
  </ul>
</div>
{{ end }}
{{ end }}
```

对于每篇文章来说，可以控制显示 toc 和 评论，使用 showToc 和 showComment 属性。

### 部署到 github pages

参考 [Build Hugo With GitHub Action](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)

在对应位置创建对应文件即可，不过我稍作了修改

1. 我的主分支是 master，所以移除了 `if: github.ref == 'refs/heads/main'`
2. publish_dir 无需手动指定，移除了 `publish_dir: ./public`

## 关于本仓库需要注意的点

你可能会想 fork 我的仓库，快速搭建自己的学习平台，这里我有些需要告诉你的。

#### 集成评论

在 config.yaml 文件中可以看到，集成了 susdis 评论，你需要换到自己的 id。

#### 新建文章

在 post 目录下新建 md 文件即可。你需要手动指定 title 和 date。

#### 修改站点图标

修改 `static/favicon.ico` 文件即可。
