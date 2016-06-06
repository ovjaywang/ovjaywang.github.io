---
title: Conversion to Dalvik format failed with error 1解决方法：
categories:
  - 代码狗
  - 学习log
tags:
  - Android
id: 212
date: 2013-05-25 00:15:52
---

Android 中出现Conversion to Dalvik format failed with error 1这个错误但是却没有错误提示...90%的原因是包的冲突 
检查一下是否有版本不同的jar包
第一种情况包导入错误.点击工程-->build path-->libraries-->选中android1.x 或者android2.x ，点击remove。
然后再点击add library-->User Library -->next-->User Libraries-->new 你取一个名字 比如android2.1 点击OK，
选中android2.1-->add jars-->\android-sdk-windows\platforms\android-7\android.jar 点击打开，点击ok-->finish.
第二种情况签名时没有成功。签名：java -jar signapk.jar platform.x509.pem platform.pk8 e:huaworkspace\hua\bin\hua.apk e:huaworkspace\hua\bin\hua_signaed.apk ，
如果hua_signaed.apk签名失败，那么请到你的工作目录中将hua_signaed.apk delete掉。
第三种情况包冲突，请到工程目录下将相同的包删除，重新导入一个，这一点和第一种情况类似，不过这是针对其他包，不是android包 