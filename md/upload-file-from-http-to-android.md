# 文件上传

## http 中的 multipartForm

<img width="600" alt="http demo" src="https://user-images.githubusercontent.com/38493346/125196885-217cea00-e28e-11eb-9ae6-ed5af0e0a891.jpg">

上图是一个简单的文件上传请求示例，该请求上传了两个文件和一个字符串数据。

http 协议规定使用 multipart/form-data 格式来上传文件。
和 form-data 格式中的参数被编码到 URL 字符串不一样的是，表单参数（包括文件内容）通过多部分的文档被包含在请求的 body 中发送。

通过分割符，我们可以在 body 中存放多个内容，可以是普通键值对数据，如第三个参数；也可以是二进制文件，如前两个图片。
文件通过 content-type 来指定类型，此外还需要指定 filename，而具体内容则是二进制文件。

需要注意的是，在请求中的 content-type 所指定的分割符，在实际内容中我们需要**在每个分割符开头以及 body 结尾多加上--**

## 客户端实现

我们使用 retrofit 来定义接口。

```kt
interface Api {
    @Multipart
    @POST
    suspend fun uploadFile(
        @Part file: MultipartBody.Part,
        @Part("token") token: RequestBody,
    )
}
```

使用 Multipart 注解表示请求体的内容格式为 multipart/form-data。
请注意，这时我们每个参数就都要加上 Part 的注解了，表示每个参数是多部分的一部分。

对于参数为普通字符串的情况，我们指定为 RequestBody 类型即可。对于文件，就需要 MultipartBody.Part。

关于Part 注解中的参数，如果类型是 MultipartBody.Part 内容将被忽略，而如果类型为 RequestBody 该值将直接与其内容类型一起使用。

上面接口的调用实现如下

```kt
suspend fun uploadFile(api: Api, inputStream: InputStream, fileName: String, token: String) = flow {
    val requestBody = RequestBody.create(MediaType.parse("multipart/form-data"), inputStream.readBytes())
    val fileInBody = MultipartBody.Part.createFormData("file", fileName, requestBody)
    val tokenInBody = RequestBody.create(MediaType.parse("multipart/form-data"), token)
    emit(api.uploadFile(fileInBody, tokenInBody))
}
```

对于文件类型， createFormData 需要我们传递一个 RequestBody 类型的对象，也就是上传的文件内容。
第一个参数表示后端接口中定义的接收者，即 http 请求中的 name；第二个参数为文件名称 filename。

```kt
Part createFormData(String name, @Nullable String filename, RequestBody body)
```

我们使用 create 方法来创建 formData，这里我们有两种方式。一种是通过输入流将文件读到 byte 数组，一种是直接传递文件。

```kt
RequestBody create(final @Nullable MediaType contentType, final byte[] content)
RequestBody create(final @Nullable MediaType contentType, final File file)
```

至于普通字符串，则没什么可说的。注意我们需要将请求体的格式都指定为 multipart/form-data。

## 其他的写法

okhttp 为我们提供了一些 kotlin 的扩展函数，可以使写法更简洁。
比如创建 MIME 类型，`"image/png".toMediaType()`；在比如创建请求体，`File("docs/images/logo-square.png").asRequestBody(MEDIA_TYPE_PNG)`。
你可以在参考示例的链接中了解更多。

## 参考

- [How does HTTP file upload work?](https://stackoverflow.com/a/8660740)
- [How to Upload Image file in Retrofit 2](https://stackoverflow.com/questions/39953457)
- [okhttp recipes](https://square.github.io/okhttp/recipes/)
