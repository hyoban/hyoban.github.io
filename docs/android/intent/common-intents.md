# 常用的 Intent 示例

## 从相机拍摄图片

将协议指定为 `TakePicture`，`launch` 函数中传递将要保存图片地址的 uri。
请注意回调中只能拿到是否拍照成功的数据，所以一般需要记录下此刻传递的 uri 便于使用拍摄的照片。

对于传递的 uri，需要先构建，这里指定出文件名并填充。

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

对于回调，可以拿到是否拍摄成功的信息，如果拍摄成功，之前记录下的 uri 就能进行操作了，比如展示图片。

## 参考链接

- [AndroidResultContracts.TakePicture() returns Boolean instead of Bitmap](https://stackoverflow.com/a/66993181/15548365))
