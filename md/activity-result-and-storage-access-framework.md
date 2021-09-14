# 从 activity result 到适配 SAF

## Activity result api

### 介绍

在 onActivityResult 中处理各种事情（比如打开系统软件，申请权限等）实在是很不优雅，google 给我们带来了全新的 api。我们来看一段示例代码：

```kotlin
const val IMAGE_MIME_TYPE = "image/*"

val processingPictures =
    registerForActivityResult(ActivityResultContracts.StartActivityForResult()) {
        if (it.resultCode == RESULT_OK) {
            it.data?.data.let { uri ->
                activity?.let {
                    binding.preview.setImageBitmap(
                        BitmapFactory.decodeStream(it.contentResolver.openInputStream(uri))
                    )
                }
            }
        }
    }

fun selectImage() {
    val intent = Intent(Intent.ACTION_GET_CONTENT).apply {
        type = IMAGE_MIME_TYPE
    }
    processingPictures.launch(intent)
}
```

我们使用 registerForActivityResult 创建一个启动器，参数为启动需要的协议，然后使用 launch 函数启动即可。
这里等于是改写了被废弃的 startActivityForResult 方法。

```kotlin
abstract class ActivityResultContract<I, O>
```

ActivityResultContract 是一个抽象类，它的范型参数分别为，我们启动的时需要指定的参数和在回调中我们能够获取到的数据。
一般我们不需要实现自己的协议类，这里使用的预置的通用的 StartActivityForResult，整体写法和之前还是很类似的。

### 示例 1 启动文件选择器获取照片

```kotlin
val processingPictures = registerForActivityResult(ActivityResultContracts.GetContent()) { uri ->}

fun selectImage() {
    processingPictures.launch("image/*")
}
```

将 Contract 指定为 GetContent 以从文件管理器中获取内容

```kotlin
class GetContent extends ActivityResultContract<String, Uri>
```

我们需要在 launch 函数中指定选择文件的类型，这里指定为 `image/*` 表示过滤掉非图片，在回调中拿到我们就可以拿到图片的 uri。

### 示例 2 从相机拍摄图片

将协议指定为 TakePicture，launch 函数中传递将要保存图片地址的 uri。
请注意回调中我们只能拿到是否拍照成功的数据，所以我们一般需要记录下此刻传递的 uri 便于使用拍摄的照片。

对于回调中的处理是这样的，我们可以拿到是否拍摄成功的信息，如果拍摄成功，之前记录下的 uri 就能加载出图片了。

```kotlin
val filename = "${System.currentTimeMillis()}.jpg"

val imageUri =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        MediaStore.Images.Media.getContentUri(
            MediaStore.VOLUME_EXTERNAL_PRIMARY
        )
    } else {
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI
    }

val imageDetails = ContentValues().apply {
    put(MediaStore.Images.Media.DISPLAY_NAME, filename)
}

context.contentResolver.insert(imageUri, imageDetails).let {
    takenPictureUri = it
    launcherFromCamera.launch(takenPictureUri)
}
```

对于传递的 uri，我们需要先自己构建出，这里我们指定出文件名并填充即可。

### jetpack compose 中对应的写法

compose 没办法和 activity 直接打交道，但实际上在 compose 中提供了对应的函数 rememberLauncherForActivityResult ，写法上完全一致。

## 访问媒体资源

### 查询文件信息

通过 contentResolver 按需获取到对应的 Cursor

我们已经拿到 uri 的话并且只希望查询对应的，可以直接传递 uri 作为第一个参数，否则传递 `MediaStore.Images.Media.EXTERNAL_CONTENT_URI` 来查询全部数据（这里以查询图像为示例）

第二个参数传递我们希望查询的数据类型的数组（这里假设我们只需要拿到文件名）

第三四个参数对应查询语句和语句中的变量，第五个参数为排序方式。只是查询对应文件的话，我们就留为空即可

```kotlin
val cursor = resolver.query(
    uri,
    arrayOf(MediaStore.Images.Media.DISPLAY_NAME),
    null,
    null,
    null
)
```

然后我们就可以开始查询了，我们先缓存下数据库中对应的行，然后获取对应行的值。这里我们确定查询对应的文件信息，如果是获取多个文件数据的话，就改为 `while (cursor.moveToNext())` 即可，持续获取数据

```kotlin
cursor?.use {
    val columnIndex =
        cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME)
    if (cursor.moveToFirst()) {
        if (columnIndex > -1) {
            fileName = cursor.getString(columnIndex)
        }
    }
}
```

如果我们是在无 uri 的情况下，查询所需的多个文件，那我们应该需要拿到文件的 uri 。要实现这个操作，我们需要先查询到 id ，然后将 id 附加到内容 uri 来获取到文件的 uri

```kotlin
val contentUri = ContentUris.withAppendedId(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    id
)
```

> 在应用中执行此类查询时，请注意以下几点：
>
> 1. 在工作线程中调用 query() 方法。
> 2. 缓存列索引，以免每次处理查询结果中的行时都需要调用 getColumnIndexOrThrow()。
> 3. 将 ID 附加到内容 URI，如代码段所示。
> 4. 搭载 Android 10 及更高版本的设备需要在 MediaStore API 中定义的列名称。如果应用中的某个依赖库需要 API 中未定义的列名称（例如 "MimeType"），请使用 CursorWrapper 在应用的进程中动态转换列名称。

## 文档和其它文件（Storage Access Framework）

本文不会介绍如何进行文件操作，只会介绍如何拿到文件或者文件夹的 uri。

> 使用场景

需要保存非私密文档数据（而非媒体文件）到手机上，并且希望数据在软件卸载之后仍然存在。

> 是否需要申请权限

不需要。因为确定读写的文件位置都是用户自己选定的，系统授予软件该用户选的文件或者文件夹的访问权限。

> 其它程序是否可以访问

可以，通过 SAF 或者程序本身拥有文件访问权限

额外的一个想法，关于软件自动备份数据的情况，该数据并非共享给其它软件使用，但是需要在软件卸载之后保持。
这种情景似乎并不符合共享存储的设计，但是在不请求文件访问权限的情况下，似乎只有这一种方式能实现。

### 创建文件

使用 Activity result api，选择 CreateDocument 这个协议（对应的 intent action 为 ACTION_CREATE_DOCUMENT）。
在 launch 函数中，我们可以设置一个预置的文件名称，用户可以稍后自由修改。
不过我们不能覆盖已有的文件，如果存在同名文件，系统会自动添加数字后缀。
在 onResult 的回调中我们可以拿到用户选择的 uri。
请注意**可能为空**，因为用户可能取消了选择的操作。之后我们可以对该 uri 进行读写操作。

此外，你可能想要指定创建文件的 MIME 类型，或者是打开文件选择器时显示的文件路径的 uri。
我们可以重写 CreateDocument 协议的 createIntent 这个函数来指定。
需要注意的是，这里的 uri 需要是[经过授权的文件夹路径](https://stackoverflow.com/a/54099021/15548365)。

```kt
contract = object : ActivityResultContracts.CreateDocument() {
    override fun createIntent(context: Context, input: String): Intent {
        return super.createIntent(context, input).apply {
            type = "application/json"
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                putExtra(DocumentsContract.EXTRA_INITIAL_URI, uri)
            }
        }
    }
}
```

事实上，我们都可以重写协议的 createIntent 方法来传递一些其他的内容到 intent 中。

### 打开文件

1. 同上，将协议改为 OpenDocument（对应的 intent action 为 ACTION_OPEN_DOCUMENT）。
2. launch 函数中我们传递字符串数组作为参数，字符串为我们限制用户可以选择的 mime 类型。
3. 拿到 uri，判空，然后就可以进行读写操作。

我们同样的也可以通过 EXTRA_INITIAL_URI 来设置文件选择器打开时的初始目录。

需要注意的是，用户无法通过文件选择器打开 `Android/data/` 和 `Android/obb/` 目录以及其子目录中的文件。

### 获取文件夹的读写权限

通过用户选择，程序可以获得某个人文件夹的访问权限，程序无法访问除此之外的文件。

1. 使用 OpenDocumentTree 协议来选择文件夹（对应 ACTION_OPEN_DOCUMENT_TREE）。
2. launch 函数可以传递指定文件选择器的起始位置的 uri。
3. 拿到可以访问的文件夹的 uri，判空。

除了前面提到的目录，用户也不可以选择存储的根目录，下载目录[等文件夹](https://developer.android.com/training/data-storage/shared/documents-files#document-tree-access-restrictions)。

#### 获取永久权限

上个步骤中我们就可以对拿到授权的文件夹内的文件进行操作，但是用户设备重启之后我们将失去该文件夹访问权限。
我们可以按照如下的方法来获取永久的访问权限。
但是请注意，如果用户手动删除或者移动了该文件夹的话，程序还是会失去访问权限。

```
val takeFlags = Intent.FLAG_GRANT_READ_URI_PERMISSION or Intent.FLAG_GRANT_WRITE_URI_PERMISSION
context.contentResolver.takePersistableUriPermission(uri, takeFlags)
```

此外，你应该需要保存下 uri 的值，以便以后进行文件操作。而如下的操作可以获取便于展示的文件目录。

```
uri.lastPathSegment?.removePrefix("primary:")
```

### 在被授权的文件夹下进行操作

首先需要添加依赖，`implementation "androidx.documentfile:documentfile:1.0.1"`

> 获取当前根目录

```kt
val home = DocumentFile.fromTreeUri(context, Uri.parse(uriString))
```

> 判断目录下有无对应文件

```
home?.findFile(displayName)
```

参数为文件名（含后缀），如果存在则返回 DocumentFile，否则为空。通过 getUri 函数获取到 uri 后就可以进行读写操作了。

> 创建新的文件

```
home?.createFile(mimeType, displayName)
```

第一个参数为文件的 mime 类型，第二个参数为文件名（不含后缀）。

还有其它的函数可用，参考文档即可。

> 一些坑

当你通过 documentFile 执行删除操作之后，就无法通过它的 uri 来进行文件操作了，会报错。
因为此时系统无法判断该 uri 是否位于你可以进行操作的目录下，尽管它实际上是的。
这里和获取文件夹权限后，文件夹的更改会导致权限丢失的情况是一致的。

## 通过 uri 来读写文件

### 写入

基本上都是使用 contentResolver api 来进行操作，使用 context 的 getContentResolver 方法来获取到。

```
context.contentResolver.openFileDescriptor(uri, "w")?.use {
    FileOutputStream(it.fileDescriptor).use { output ->
        output.write(bytearray)
    }
}
```

获取到 ParcelFileDescriptor 的 fileDescriptor 就可以构建文件输入输出流，包装 ParcelFileDescriptor 的主要作用是允许我们使用完之后可以关闭它。

构建的 FileOutputStream 允许我们使用字节流的方式来写入文件，比如我们可以写入图像。如果是字符流的话，我们需要使用 FileReader。

### 读取

```
context.contentResolver.openInputStream(uri).use {
    val reader = BufferedReader(InputStreamReader(it))
    reader.forEachLine {

    }
}
```

这里我们和上面的示例不同，展示按照字符流的方式来读取文件的操作。InputStreamReader 是字节流和字符流之间的桥梁，但是考虑到，我们每读入一个字符，就需要读入多个字节。我们需要通过构建 BufferedReader 来获取更好的读入效率，它会为我们处理好字符的缓冲。

forEachLine 是个好用的扩展函数，我们按行读入数据，当没有更多数据之后，它会为我们关闭掉使用的资源。

我们也可以使用 readText 这个函数，来一次性读入全部内容，请注意需要我们手动关闭。

## 参考

- [再见！onActivityResult！你好，Activity Results API！](https://segmentfault.com/a/1190000037601888)
- [AndroidResultContracts.TakePicture() returns Boolean instead of Bitmap](https://stackoverflow.com/questions/63403425/androidresultcontracts-takepicture-returns-boolean-instead-of-bitmap/66993181#66993181)
- [Android 通过 URI 获取文件路径](https://juejin.cn/post/6844903545385451528)
- [Access documents and other files from shared storage](https://developer.android.com/training/data-storage/shared/documents-files)
