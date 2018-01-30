# 视频格式判断

最近贵司的项目需用对用户上传的文件进行校验，其中视频格式判断尤为重要

本文旨在详解格式判断的具体做法，而不是像网上人云亦云的mp4格式解释，最后会附上Python代码

工欲善其事必先利其器：

* sublime mac上的轻量级编辑器，官网下载就好
* Any Video Converter Pro，mac上视频格式转换的工具，[/下载链接](/下载链接 "https://www.jianshu.com/p/4ed2917e79cf")
* 网页版ASCII 在线转换器，[/下载链接](/下载链接 "http://www.ab126.com/goju/1711.html")
* 视频demo，什么格式都无所谓了

关于视频格式，有两个概念很容易被混淆

编码方式，举个例子H.264，通过一些方法来压缩视频方便传输的，[/详情](/详情 "http://simplecodesky.com/2016/11/15/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA%E7%90%86%E8%A7%A3%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81H264%E7%BB%93%E6%9E%84/")

视频封装，举个例子MP4，我们常见的视频文件后缀都是此类，

```
               同一种编码方式也可能有很多视频封装格式，这是两个概念，并不冲突
```

可能有人会说，利用文件后缀可以判断文件类型。

但是，后缀很容易被改变，所以这显然不可行！

所以，我们根据文件头来判断视频的真实格式，那么问题来了，什么是文件头？

网上有很多解释，如果不是写论文，我们用不到那些概念，对于程序员来说，告诉我是哪些二进制数字就可以了，对吧！

### 第一步

将你的视频demo拖进sublime，即可观察到十六进制数字，就像这样

![](/assets/import.png)

文件头就是前面的几个数字，具体哪几个数字，不同的格式不同。上图这种mp4格式文件，前四个字节表示整个[/ftyp](/ftyp "http://blog.csdn.net/yu\_yuan\_1314/article/details/9366703")的长度，第8个到第12个字节就是视频格式啦，这很关键！不过也有例外，wmv和flv就都不是，不过这些表示视频格式的字节一定存在。

### 第二步

将第8-12个字节用ASCII 在线转换器翻译一下，你就会发现一些神奇的事情，比如你试下上图中的69736f6d，[/对照表格查看](/对照表格查看 "http://www.ftyps.com/\#7")，[/表格二](/表格二 "http://www.cnblogs.com/WangAoBo/p/6366211.html")，注意，有些视频格式并不是这几个字节，也许你还需要Google一下才知道。

### 第三步

利用视频格式转换工具将demo转换成其他视频格式，重复上面两步操作，你就可以搞清楚各种文件格式的奥秘了，很简单吧！

Python代码如下：

```
# coding: utf-8
# 利用文件头判断文件类型
import struct


def typelist():
    return {
        '000001BA': 'MPEG',
        '000001B3': 'MPEG',
        '41564920': 'avi',
        '6D6F6F76': 'mov'
    }


def bytes2hex(bytes):
    num = len(bytes)
    hexstr = u''
    for i in range(num):
        t = u'%x' % bytes[i]
        if len(t) % 2:
            hexstr += u'0'
        hexstr += t
    return hexstr.upper()


def filetype(filename):
    binfile = open(filename, 'rb')
    tl = typelist()
    ftype = 'unknown'
    for hcode in tl.keys():
        numOfBytes = len(hcode)/2
        binfile.seek(0)
        hbytes = struct.unpack_from('B'*numOfBytes, binfile.read(numOfBytes))
        f_hcode = bytes2hex(hbytes)
        if f_hcode == hcode:
            ftype = tl[hcode]
            break
    binfile.close()
    return ftype

if __name__ == '__main__':
    print filetype('你的文件路径')
```

这个代码不能直接使用，改变一下输入输出，以及读取字节的位置即可！

