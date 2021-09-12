# 如何开始写一个 android 项目

## android studio 设置

1. 更新最新，请保持版本和插件为最新稳定版
2. 内存设置，设置到最大
3. sdk 下载
4. 代理配置

## idea 系快捷键

 快捷键                    | 功能           
------------------------|--------------
 cmd + shift + u        | 转换大小写        
 option + shift + click | 多光标          
 cmd + option + t       | 创建代码块供折叠     
 cmd + .                | 折叠           
 cmd + shift + + or -   | 快速折叠和展开全部代码块 

## spotless 和 ktlint

### 通过 ktlint 来设置 kotlin 代码风格

需要提前安装好 ktlint，在项目文件夹下运行

```sh
ktlint --android applyToIDEAProject
```

这只会影响当前 idea 项目，也是官方推荐的方法，你可以点击 [链接](https://github.com/pinterest/ktlint#-with-intellij-idea) 查看更多

### 设置 spotless

请手动查询当前的 spotless 和 ktlint 的最新版本，链接已放在后面

<script src="https://gist.github.com/hyoban/e18fc910f045e50e0fb7a37c39030f25.js"></script>

不要忘记移除 as 默认生成的 task

```gradle
task clean(type: Delete) {
    delete rootProject.buildDir
} 
```

### 配置 gradle

终端输入 `./gradlew app:spotlessApply`，按下 `cmd + enter` 来创建

### 参考

- [谷歌项目示例](https://github.com/android/android-dev-challenge-compose)
- [spotless](https://github.com/diffplug/spotless)
- [ktlint](https://github.com/pinterest/ktlint)

## 启用版本控制

我们需要配置好 git ignore 文件，可以参考前面的谷歌项目示例，也可以参考 github 维护的 [示例](https://github.com/github/gitignore/blob/master/Android.gitignore)

当然你可以排除你想要排除的，但是请确保别人 clone 下你的项目，可以马上跑起来

## 使用 r8 来压缩应用体积

我们只需要设置 minifyEnabled true 即可

不过需要注意的是，一些第三方库使用到了反射，我们需要保留这些代码不被优化掉。要保留名称字段，在 proguard-rules.pro 文件中添加一个保留规则 -keep:

1. litepal [ProGuard](https://github.com/guolindev/LitePal#proguard)
2. serialization [ProGuard](https://github.com/Kotlin/kotlinx.serialization#android)

### 参考

- [Shrink, obfuscate, and optimize your app](https://developer.android.google.cn/studio/build/shrink-code)
- [使用 R8 压缩您的应用](https://mp.weixin.qq.com/s/zDx-SdsqargT4JB6oMIrTw)
