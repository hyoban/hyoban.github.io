# 序列化的处理技巧

## 和后端勾心斗角

### 传递给你非标准格式 json

如果后台直接设置响应体格式为 string 的话，就会报错 `json document was not fully consumed`。

> 参考 [json document was not fully consumed](https://github.com/square/retrofit/issues/3004#issuecomment-514671419)

引入库
```
implementation "com.squareup.retrofit2:converter-scalars:2.5.0"
```

添加在你使用的序列化库之前
```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(ScalarsConverterFactory.create())
        .addConverterFactory(GsonConverterFactory.create())
        .build();
```

### post 请求的请求体格式需要动态指定

如果是固定格式的请求很好处理，直接构建对应类型的 data class 作为参数即可。
而请求体格式不确定的话就麻烦一点了，我们只能指定参数为 RequestBody。

> 参考 [How to POST raw whole JSON in the body of a Retrofit request?](https://stackoverflow.com/a/36821182)

```kt
fun createJsonRequestBody(vararg params: Pair<String, String>) =
        RequestBody.create(
            okhttp3.MediaType.parse("application/json; charset=utf-8"), 
            JSONObject(mapOf(*params)).toString())
```

调用的时候传递多个 pair 即可。

## 放弃 Gson

我放弃和 kotlin 一起使用 Gson，主要有以下两点：

1. 导致 kotlin 的空判断失效
2. 极其受限制的情况下才能添加默认值

> 参考 [Gson 和 Kotlin data class 的避坑指南](https://juejin.cn/post/6908391430977224718)

## 拥抱 kotlin serialization

这个库有很多我觉得应该默认的设置需要自己来设置。

### 解析数组起始的 json 内容

serialization 默认时不支持解析的，需要我们自定义 serializer。考虑到这是个比较常见的需求，kotlin 内置了这个工具。

需要注意的是，在使用 data class 定义数据格式时，只需要定义数据中每个元素的格式即可。
在 retrofit 部分，我们指定返回数据类型为对应的 list。然后将 `ListSerializer(Type.serializer()) ` 传递到 Json 的 builder 中，即可自动解析。

> 参考 [kotlinx.serialization serializers](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/serializers.md#constructing-collection-serializers)

### 不全部解析 json 内容

在 Json 的 builder 里面进行配置。

```kt
Json {
    ignoreUnknownKeys = true
}
```

### 忽略字段

添加 `@Transient` 注解到需要忽略的字段

### 和 retrofit 配合

我们可以自己去 GitHub 上找一些开源的实现，比如 [Kotlin Serialization Converter](https://github.com/JakeWharton/retrofit2-kotlinx-serialization-converter)
