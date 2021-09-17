# Activity result

## 示例

```kotlin
const val IMAGE_MIME_TYPE = "image/*"

val processingPictures = registerForActivityResult(
    contract = ActivityResultContracts.StartActivityForResult()
) {
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

使用 `registerForActivityResult` 创建一个启动器，参数为启动需要的协议和拿到结果之后的回调，然后使用 `launch` 函数启动即可。

约等于是改写了被废弃的 `startActivityForResult` 方法，不过不再需要定义 request code，各种不同类型的事务也不再堆叠在一起。

## 协议

```kotlin
abstract class ActivityResultContract<I, O>
```

`ActivityResultContract` 是一个抽象类，它的范型参数分别为，我们启动的时需要指定的参数和在回调中我们能够获取到的数据。

一般我们不需要实现自己的协议类，比如这里使用的预置的通用的 `StartActivityForResult`，整体写法和之前还是很类似的。
你可以查看 [更多预置的协议](common-intents.md)

### 补充协议信息

大多数协议都将常用的参数作为范型参数，而非直接使用 Intent 作为参数。
如果有额外的信息需要补充，我们需要重写协议的 `createIntent` 方法来传递一些其他的内容到 intent 中。

```kotlin
contract = object : ActivityResultContracts.CreateDocument() {
    override fun createIntent(context: Context, input: String): Intent {
        return super.createIntent(context, input).apply {

        }
    }
}
```

## 其它

jetpack compose 中提供了对应的函数 `rememberLauncherForActivityResult` ，写法上完全一致。

## 参考链接

- [再见！onActivityResult！你好，Activity Results API！](https://segmentfault.com/a/1190000037601888)
