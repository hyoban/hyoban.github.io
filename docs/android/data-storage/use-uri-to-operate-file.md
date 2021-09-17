# 通过 uri 来操作文件

## 写入

基本上都是使用 contentResolver api 来进行操作，使用 context 的 getContentResolver 方法来获取到。

```kotlin
context.contentResolver.openFileDescriptor(uri, "w")?.use {
    FileOutputStream(it.fileDescriptor).use { output ->
        output.write(bytearray)
    }
}
```

获取到 ParcelFileDescriptor 的 fileDescriptor 就可以构建文件输入输出流，包装 ParcelFileDescriptor 的主要作用是允许我们使用完之后可以关闭它。

构建的 FileOutputStream 允许我们使用字节流的方式来写入文件，比如我们可以写入图像。如果是字符流的话，我们需要使用 FileReader。

## 读取

```kotlin
context.contentResolver.openInputStream(uri).use {
    val reader = BufferedReader(InputStreamReader(it))
    reader.forEachLine {

    }
}
```

这里我们和上面的示例不同，展示按照字符流的方式来读取文件的操作。InputStreamReader 是字节流和字符流之间的桥梁，但是考虑到，我们每读入一个字符，就需要读入多个字节。我们需要通过构建 BufferedReader 来获取更好的读入效率，它会为我们处理好字符的缓冲。

forEachLine 是个好用的扩展函数，我们按行读入数据，当没有更多数据之后，它会为我们关闭掉使用的资源。

我们也可以使用 readText 这个函数，来一次性读入全部内容，请注意需要我们手动关闭。
