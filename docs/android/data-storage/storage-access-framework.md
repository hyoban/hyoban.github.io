# Storage Access Framework

本文不会介绍如何进行文件操作，只会介绍如何拿到文件或者文件夹的 uri，后续请参考 [通过 uri 来操作文件](use-uri-to-operate-file.md)

> 使用场景

需要保存非私密文档数据（而非媒体文件）到手机上，并且希望数据在软件卸载之后仍然存在。

> 是否需要申请权限

不需要。因为确定读写的文件位置都是用户自己选定的，系统授予对应部分的访问权限。

> 其它程序是否可以访问

可以，通过 SAF 或者程序本身拥有文件访问权限

## 创建文件

使用 [Activity result API](../intent/activity-result.md)，选择 `CreateDocument` 这个协议（对应 ACTION_CREATE_DOCUMENT）。

在 `launch` 函数中，我们可以设置一个预置的文件名称，用户可以稍后自由修改。
如果存在同名文件，不会覆盖已有的文件，系统会自动添加数字后缀。

在 `onResult` 的回调中我们可以拿到用户选择的 uri。
请注意可能为空，因为用户可能取消了选择的操作。之后我们可以对该 uri 进行读写操作。

此外，你可以 [重写协议的 `createIntent` 方法](../../intent/activity-result/#补充协议信息)，通过设置 `type` 指定创建文件的 MIME 类型，或者是指定 `EXTRA_INITIAL_URI` 来打开文件选择器时显示的文件路径的 uri。
需要注意的是，这里的 uri 需要是 [经过授权的文件夹路径](https://stackoverflow.com/a/54099021/15548365)。

```kotlin
type = "application/json"
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    putExtra(DocumentsContract.EXTRA_INITIAL_URI, uri)
}
```

## 打开文件

1. 同上，将协议改为 OpenDocument（对应的 intent action 为 ACTION_OPEN_DOCUMENT）。
2. launch 函数中我们传递字符串数组作为参数，字符串为我们限制用户可以选择的 mime 类型。
3. 拿到 uri，判空，然后就可以进行读写操作。

我们同样的也可以通过 EXTRA_INITIAL_URI 来设置文件选择器打开时的初始目录。

需要注意的是，用户无法通过文件选择器打开 `Android/data/` 和 `Android/obb/` 目录以及其子目录中的文件。

## 获取文件夹的读写权限

通过用户选择，程序可以获得某个人文件夹的访问权限，程序无法访问除此之外的文件。

1. 使用 OpenDocumentTree 协议来选择文件夹（对应 ACTION_OPEN_DOCUMENT_TREE）。
2. launch 函数可以传递指定文件选择器的起始位置的 uri。
3. 拿到可以访问的文件夹的 uri，判空。

除了前面提到的目录，用户也不可以选择存储的根目录，下载目录[等文件夹](https://developer.android.com/training/data-storage/shared/documents-files#document-tree-access-restrictions)。

### 获取永久权限

上个步骤中我们就可以对拿到授权的文件夹内的文件进行操作，但是用户设备重启之后我们将失去该文件夹访问权限。
我们可以按照如下的方法来获取永久的访问权限。
但是请注意，如果用户手动删除或者移动了该文件夹的话，程序还是会失去访问权限。

```kotlin
val takeFlags = Intent.FLAG_GRANT_READ_URI_PERMISSION or Intent.FLAG_GRANT_WRITE_URI_PERMISSION
context.contentResolver.takePersistableUriPermission(uri, takeFlags)
```

此外，你应该需要保存下 uri 的值，以便以后进行文件操作。而如下的操作可以获取便于展示的文件目录。

```kotlin
uri.lastPathSegment?.removePrefix("primary:")
```

## 在被授权的文件夹下进行操作

首先需要添加依赖，`implementation "androidx.documentfile:documentfile:1.0.1"`

> 获取当前根目录

```kotlin
val home = DocumentFile.fromTreeUri(context, Uri.parse(uriString))
```

> 判断目录下有无对应文件

```kotlin
home?.findFile(displayName)
```

参数为文件名（含后缀），如果存在则返回 DocumentFile，否则为空。通过 getUri 函数获取到 uri 后就可以进行读写操作了。

> 创建新的文件

```kotlin
home?.createFile(mimeType, displayName)
```

第一个参数为文件的 mime 类型，第二个参数为文件名（不含后缀）。

还有其它的函数可用，参考文档即可。

> 一些坑

当你通过 documentFile 执行删除操作之后，就无法通过它的 uri 来进行文件操作了，会报错。
因为此时系统无法判断该 uri 是否位于你可以进行操作的目录下，尽管它实际上是的。
这里和获取文件夹权限后，文件夹的更改会导致权限丢失的情况是一致的。
