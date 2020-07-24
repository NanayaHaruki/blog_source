---
title: 导航栏的去除和显示（不是隐藏）
date: 2020-06-29 10:04:27
tags: 
- Android
- UI
categories: Android
---

### 问题：
搜了很多文章，控制导航栏是这么干的
```
        window.decorView.systemUiVisibility =
        View.SYSTEM_UI_FLAG_FULLSCREEN or
        View.SYSTEM_UI_FLAG_HIDE_NAVIGATION  or
                View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION or
        View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
```
这种只是隐藏了，触摸屏幕还是能划出来的，而且点击`EditText`也会因为输入法的弹出而与导航栏联动。
### 解决：

在Android源码`PhoneWindowManager.java`中可以看到这么一段
```
        String navBarOverride = SystemProperties.get("qemu.hw.mainkeys");
        if ("1".equals(navBarOverride)) {
            mHasNavigationBar = false;
        } else if ("0".equals(navBarOverride)) {
            mHasNavigationBar = true;
        }
```
所以在`system/build.prop`内增加一行`qemu.hw.mainkeys=1`，即可去掉导航栏。改为0即可显示导航栏。
android没有vim，编辑文件我是pull到电脑中改完了再push进去的。

`adb pull /system/build.prop d:/test/build.prop`  
`adb push d:/test/build.prop /system/build.prop `  