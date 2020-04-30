---
title: Android 10 使用OkHttp4和Retrofit上传图片
date: 2020-4-30 12:00:00
categories:
- Android/开发
tags: Android
--- 


作者：Angki  
转载请注明

---
最近发现项目在Android 10下，上传图片报错

```
open failed: EACCES (Permission denied)
```
上网查了下，是由于Android 10的文件存储机制修改成了沙盒模式，应用不能直接访问除了沙盒文件和公共文件以外的文件，直接使用图片绝对地址上传图片会出错。最简单的解决办法就是在AndroidManifest.xml中添加

```
android:requestLegacyExternalStorage="true"
```
但是，这方式只是临时解决而已，Google也说明了在Android 11 中这种临时的解决办法也会失效。为了以后方便，直接就是用Google推荐的方式进行适配。

由于项目网络框架使用OkHttp4和Retrofit，只要把上传需要的MultipartBody对象构建出来就行了，之前的写法：

```
val file = File(图片的绝对地址)
//传输图片
val img = file.asRequestBody("image/png".toMediaType())
//MultipartBody
val request = MultipartBody.Builder()
    .addFormDataPart("file", "${TimeUtils.getNowMills()}.png", img)
    .build()
```
适配后的写法

```
var file: ByteArray? = null
contentResolver.openInputStream(图片的Uri).use {
    file = it?.readBytes()
}
file?.let {
    val img = it.toRequestBody("image/png".toMediaType())
    val request = MultipartBody.Builder()
        .addFormDataPart("file", "${TimeUtils.getNowMills()}.png", img)
        .build()
    mViewModel.uploadImage(request)
}
```

之前的写法和适配后的写法差别就是在于RequestBody的获取。在适配过程中，陷入了一个误区，就是要先获取File文件，然后使用OkHttp提供的方法来获取RequestBody。

```
fun File.asRequestBody(contentType: MediaType? = null): RequestBody 
```
然后就找方法来获取File，但是只找到了通过Uri获取Bitmap，我就在想着先获取Bitmap，然后把Bitmap转换成File文件。幸运的是，我看见了OkHttp提供的其他方法来获取RequestBody。

```
fun String.toRequestBody(contentType: MediaType? = null): RequestBody
fun ByteString.toRequestBody(contentType: MediaType? = null): RequestBody
fun ByteArray.toRequestBody(
    contentType: MediaType? = null,
    offset: Int = 0,
    byteCount: Int = size
    ): RequestBody
```
再结合在官网看见的获取文件的流方法：

```
// Open a specific media item using InputStream.
val resolver = applicationContext.contentResolver
resolver.openInputStream(content-uri).use { stream ->
    // Perform operations on "stream".
}
```
最后就有了适配的写法，测试了一下，没得问题。最后得出一个结论，我是菜狗...





