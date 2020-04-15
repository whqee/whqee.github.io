---
layout: post
title: 编译你的第一个安卓APP（使用命令行，而不是Android Studio）
date: 2020-04-09 21:00:00 +800
description: 搭建环境并编译你的第一个安卓APP（使用命令行，而不是Android Studio）                                                                                  # Add post description (optional)
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
categories: [软、硬件编程, Android]
tags: [Android] # add tag
seo:
  date_modified: 2020-04-10 08:55:26 +0800
---

笔者环境：Linux  
参考资料：安卓官网  

## 一、下载工具，准备开发环境
（先安装好java开发环境，比如open-jdk）
到 https://developer.android.google.cn/studio 下载command line tools：  
![](/post_images/android-app-development/1.png)  

在任意位置新建一个文件夹，任意名称，我命名为android_sdk，把下载的commandlinetools解压到这里。  
查看usage: `tools/bin/sdkmanager --help`  
列出可安装的内容:  
```
[whqee@whqee android_sdk]$ tools/bin/sdkmanager --sdk_root=. --list
Installed packages:=====================] 100% Computing updates...             
  Path    | Version | Description         | Location
  ------- | ------- | -------             | ------- 
  tools   | 1.0.0   | Android SDK Tools 1 | tools/  

Available Packages:                                                       
  中间省略...                                               
Available Updates:
  ID      | Installed | Available
  ------- | -------   | -------  

[whqee@whqee android_sdk]$ 
```

可以看到一大页的列表,为了方便输入命令，起个别名：  
`alias sdkmanager="tools/bin/sdkmanager --sdk_root=. "`  

安装android29平台和工具链,当然你也可以安装其它的:  
`sdkmanager "platforms;android-29" "build-tools;29.0.3"`  

文件夹现状:
```
[whqee@whqee android_sdk]$ tree -L 2
.
├── build-tools
│   └── 29.0.3
├── licenses
│   └── android-sdk-license
├── platforms
│   └── android-29
└── tools
    ├── bin
    ├── lib
    ├── NOTICE.txt
    ├── package.xml
    └── source.properties

8 directories, 4 files
```  

## 二、开始构建第一个安卓APP

### 1、构建前的准备  
在任意地方新建一个文件夹，比如我新建一个名为“hello_apk”的文件夹，进入文件夹。  
新建一个res文件夹(resource)，里面分别新建layout(UI布局)和values(变量)文件夹，并新建两个文件，如下：  
```
[whqee@whqee hello_apk]$ tree res
res
├── layout
│   └── helloUI.xml
└── values
    └── strings.xml

2 directories, 2 files
```
其中，helloUI.xml：  
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools"
   android:layout_width="match_parent"
   android:layout_height="match_parent" >
   
   <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_centerHorizontal="true"
      android:layout_centerVertical="true"
      android:text="@string/hello_msg"
      tools:context=".hello" />

  <Button
          android:id="@+id/startbutton"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_below="@+id/time_display_box"
          android:layout_centerHorizontal="true"
          android:layout_marginTop="41dp"
          android:text="@string/start" />

</RelativeLayout>
```
strings.xml：  
```
<resources>
   <string name="app_name">APP Hello Android</string>
   <string name="hello_msg">Hello Android!</string>
   <string name="menu_settings">Settings</string>
   <string name="title_activity_main">MainActivity</string>
   <string name="start">Start</string>
</resources>
```
根目录新建一个hello_android_manifest.xml文件，内容如下：  
```
<?xml version='1.0'?>
<manifest xmlns:a='http://schemas.android.com/apk/res/android' package='demo.demo' a:versionCode='1' a:versionName='1'>
    <application a:label='A Hello Android'>
        <activity a:name='demo.demo.hello'>
             <intent-filter>
                <category a:name='android.intent.category.LAUNCHER'/>
                <action a:name='android.intent.action.MAIN'/>
             </intent-filter>
        </activity>
    </application>
</manifest>
```
新建java/demo文件夹路径（省得编译后的临时.class文件在根目录显得乱），里面新建一个hello.java文件：  
```
package demo.demo

import android.app.Activity;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.view.View;

public class hello extends Activity {
   @Override
   protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.helloUI);

	  Button startButton = (Button) findViewById(R.id.startbutton);
	  startButton.setOnClickListener(new View.OnClickListener() {
		  public void onClick(View view) {
			  try {

			  } catch (NumberFormatException e) {

			  }
		  }
	  });
   }
}
```
然后所有文件路径看起来是这样的了：  
```
[whqee@whqee hello_apk]$ tree
.
├── hello_android_manifest.xml
├── java
│   └── demo
│       └── hello.java
└── res
    ├── layout
    │   └── helloUI.xml
    └── values
        └── strings.xml

5 directories, 4 files
```

### 2、开始构建APK  
导出build-tools的路径到全局环境变量：  
`export PATH=~/Development/Android/android_sdk/build-tools/29.0.3/:$PATH`

声明路径变量，方便引用android.jar：  
`PLATFORM=~/Development/Android/android_sdk/platforms/android-29/android.jar`

新建build文件夹，  
(1)先把res里的资源编译成.flat文件：  
```shell
aapt2 compile res/*/* -o build/
``` 

(2)把.flat文件链接成.apk，并生成R.java：  
```
aapt2 link -o unsigned.demo.apk -I $PLATFORM build/* --manifest hello_android_manifest.xml --java build`  
```

(3)编译java文件成.class（现状只有R.java和hello.java）：  
```
javac `find . -name "*.java"` -classpath $PLATFORM
```

(4)使用d8把.class编译成一个.dex  
```
d8 --lib $PLATFORM --release --output build/ build/demo/demo/R*.class
```
(5)前面打包的apk里只有UI、变量等资源，现在把程序逻辑打包进去：
```
aapt add unsigned.demo.apk build/classes.dex
```
(6)将.apk后缀的zip压缩包内容按4字节对齐(非必要，减少运行后RAM占用量)：
```
zipalign -p 4 unsigned.demo.apk unsigned.aligned.demo.apk
```
(7)使用keytool生成证书/导入证书/生成密钥/导入密钥，查看帮助：
```
[whqee@whqee hello_apk]$ keytool --help
密钥和证书管理工具

命令:

 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法
[whqee@whqee hello_apk]$ 
```
生成一个密钥对，alias不填默认为“mykey”：
```
[whqee@whqee hello_apk]$ keytool -genkeypair -validity 10000 -keystore hello.keystore -keyalg RSA -alias whqee.cn -storetype pkcs12
输入密钥库口令:  
再次输入新口令: 
您的名字与姓氏是什么?
  [Unknown]:  whqee.cn
您的组织单位名称是什么?
  [Unknown]:  whqee.cn
您的组织名称是什么?
  [Unknown]:  whqee.cn
您所在的城市或区域名称是什么?
  [Unknown]:  
您所在的省/市/自治区名称是什么?
  [Unknown]:  
该单位的双字母国家/地区代码是什么?
  [Unknown]:  
CN=whqee.cn, OU=whqee.cn, O=whqee.cn, L=Unknown, ST=Unknown, C=Unknown是否正确?
  [否]:  y

[whqee@whqee hello_apk]$ 
```
查看这个密钥对（公钥+私钥）：
```
[whqee@whqee hello_apk]$ keytool -list -v -keystore hello.keystore
输入密钥库口令:  
密钥库类型: PKCS12
密钥库提供方: SUN

您的密钥库包含 1 个条目

别名: whqee.cn
创建日期: 2020-4-10
条目类型: PrivateKeyEntry
证书链长度: 1
证书[1]:
所有者: CN=whqee.cn, OU=whqee.cn, O=whqee.cn, L=Unknown, ST=Unknown, C=Unknown
发布者: CN=whqee.cn, OU=whqee.cn, O=whqee.cn, L=Unknown, ST=Unknown, C=Unknown
序列号: 4b7bc328
有效期为 Fri Apr 10 02:29:40 CST 2020 至 Tue Aug 27 02:29:40 CST 2047
证书指纹:
         MD5:  CD:C8:74:B0:3A:CC:43:56:B0:72:F9:15:F5:3C:06:28
         SHA1: 55:E2:7A:BB:65:53:BD:C7:09:E4:D1:59:22:3F:E9:16:A1:2D:CF:11
         SHA256: EB:A8:A4:DE:CD:E5:E0:9D:00:55:09:6E:4B:F1:8B:E5:28:8E:EF:3A:07:13:81:6B:FD:BA:3F:78:F2:29:3D:AF
签名算法名称: SHA256withRSA
主体公共密钥算法: 2048 位 RSA 密钥
版本: 3

扩展: 
（省略...）
```
导出密钥证书（只包含公钥）：
```
[whqee@whqee hello_apk]$ keytool -exportcert -keystore hello.keystore -alias whqee.cn -file hello.crt
输入密钥库口令:  
存储在文件 <hello.crt> 中的证书
[whqee@whqee hello_apk]$ 
```
(8)使用密钥签名apk：
```
[whqee@whqee hello_apk]$ apksigner sign --ks hello.keystore --out hello-release.apk unsigned.aligned.demo.apk 
Keystore password for signer #1: 
[whqee@whqee hello_apk]$ 
```
完成。  
>附合集：

```
export PATH=~/Development/Android/android_sdk/build-tools/26.0.3/:$PATH
PLATFORM="/opt/android-sdk/platforms/android-29/android.jar"
rm -r build
rm my-release-key.jks demo-release.apk unsigned.aligned.demo.apk unsigned.demo.apk
mkdir -p build
aapt2 compile res/*/* -o build/
aapt2 link -o unsigned.demo.apk -I $PLATFORM build/* --manifest hello_android_manifest.xml --java build
javac `find . -name "*.java"` -classpath $PLATFORM 
d8 --lib $PLATFORM --release --output build/ build/demo/demo/R*.class 
aapt add unsigned.demo.apk build/classes.dex 
zipalign -p 4 unsigned.demo.apk unsigned.aligned.demo.apk
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -validity 10000 -alias whqee.cn -storetype pkcs12
keytool -list -v -keystore my-release-key.jks 
apksigner sign --ks my-release-key.jks --out demo-release.apk unsigned.aligned.demo.apk
```
>另一种方式：
```
#!/bin/bash
export PATH=~/Development/Android/android_sdk/build-tools/26.0.3/:$PATH 
set -e

AAPT="~/Development/Android/android_sdk/build-tools/26.0.3/aapt"
DX="~/Development/Android/android_sdk/build-tools/26.0.3/dx"
ZIPALIGN="~/Development/Android/android_sdk/build-tools/26.0.3/zipalign"
APKSIGNER="~/Development/Android/android_sdk/build-tools/26.0.3/apksigner"
PLATFORM="/home/whqee/Development/Android/android_sdk/platforms/android-26/android.jar"
 
echo "Cleaning..."
rm -rf build
# rm my-release-key.jks *.apk 

echo "Generating R.java file..."
mkdir -p build
aapt p -f -m -J build -M AndroidManifest.xml -S res -I $PLATFORM

echo "Compiling..."
javac -d build -classpath build/demo/demo -bootclasspath $PLATFORM -source 1.7 -target 1.7 java/demo/hello.java
javac -d build -classpath build/demo/demo -bootclasspath $PLATFORM -source 1.7 -target 1.7 build/demo/demo/R.java
 
echo "Translating in Dalvik bytecode..."
dx --dex --output=classes.dex build
 
echo "Making APK..."
aapt package -f -m -F build/hello.unaligned.apk -M AndroidManifest.xml -S res -I $PLATFORM
aapt add build/hello.unaligned.apk build/classes.dex
 
echo "Aligning and signing APK..."
zipalign -f 4 build/hello.unaligned.apk build/hello.apk
# keytool -genkeypair -validity 365 -keystore mykey.keystore -keyalg RSA -keysize 2048
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias
apksigner sign --ks mykey.keystore build/hello.apk
```
