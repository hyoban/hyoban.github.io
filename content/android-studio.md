---
title: 开始写一个 android 项目
date: 2021-07-27T18:11:20+08:00
summary: 关于 Android studio 的配置
---

## android studio 设置

1. 更新最新，请保持版本和插件为最新稳定版
2. 内存设置，设置到最大
3. sdk 下载
4. 代理配置
5. 字体为 16 号

## spotless 和 ktlint

### 通过 ktlint 来设置 kotlin 代码风格

需要提前安装好 ktlint，在项目文件夹下运行

```sh
ktlint --android applyToIDEAProject
```

这只会影响当前 idea 项目，也是官方推荐的方法，你可以点击 [链接](https://github.com/pinterest/ktlint#-with-intellij-idea) 查看更多。

### 设置 spotless

请手动查询当前最新版本，链接已放在后面

gradle 

```gradle
plugins {
    id 'com.diffplug.spotless' version '5.14.2'
}

subprojects {
    apply plugin: 'com.diffplug.spotless'
    spotless {
        kotlin {
            target '**/*.kt'
            targetExclude("$buildDir/**/*.kt")
            targetExclude('bin/**/*.kt')

            ktlint("0.42.1")
//            licenseHeaderFile rootProject.file("spotless/copyright.kt")
        }
    }
}
```

kts

```kt
plugins {
    id("com.diffplug.spotless") version "5.14.2"
}

subprojects {
    apply(plugin = "com.diffplug.spotless")
    spotless {
        kotlin {
            target("**/*.kt")
            targetExclude("$buildDir/**/*.kt")
            targetExclude("bin/**/*.kt")

            ktlint("0.42.1")
//            licenseHeaderFile rootProject.file("spotless/copyright.kt")
        }
    }
}
```

移除 as 默认生成的 task

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
