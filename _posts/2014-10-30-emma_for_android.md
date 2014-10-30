---
layout: post
title: 使用EMMA获取Android测试覆盖率
category: technology
---
[EMMA](http://emma.sourceforge.net/)是一个Java代码测试覆盖率获取工具。尝试了一种使用EMMA获取Android测试覆盖率的方法，参考使用了
[DynoDroid](http://www.cercs.gatech.edu/tech-reports/tr2012/git-cercs-12-09.pdf)提供的方法，其原理是使用插桩与`BroadcastReceiver`，使得插桩后打包签名而成的APK运行时每次操作均发送信息给`BroadcastReceiver`，`BroadcastReceiver`中负责将覆盖率信息写到SD卡的名为`coverage.ec`的文件中。其一大优点为全程无需修改原APK的源码。

## Pre ##
假定APK所在包为`net.clasnake.project`，工程主目录为`/folder`。

## Step 1 插桩 ##

首先下载[EmmaInstrument.rar]({{site:url}}/assets/code/EmmaInstrument.rar)，解压后包含四个文件：


- EmmaInsrumentation.java
- FinishListener.java
- InstrumentedActivity.java
- SMSInstrumentedReceiver.java

将文件夹EmmaInstrument复制到/folder/src下。
将上述四个java文件的包名修改为`net.clasnake.project.EmmaInstrument`，并令InstrumentActivity继承自项目的主Activity。
然后修改AndroidManifest.xml，加入SMSInstrumentedReceiver、EmmaInstrumentationActivity：

```xml
<receiver android:name="net.clasnake.project.EmmaInstrument.SMSInstrumentedReceiver">
	<intent-filter>
		<action android:name="edu.gatech.m3.emma.COLLECT_COVERAGE"/>
	</intent-filter>
</receiver>
<activity android:label="EmmaInstrumentationActivity" android:name="net.clasnake.project.EmmaInstrument.InstrumentedActivity"/>
```

加入插桩标签，并允许写SD卡权限:

```xml
<instrumentation android:handleProfiling="true" android:label="EmmaInstrumentation" android:name="net.clasnake.project.EmmaInstrument.EmmaInstrumentation" android:targetPackage="net.clasnake.project"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
## Step 2 重编译、安装 ##

连接设备至adb，首先`android update project`更新项目，生成build.xml，以便使用ant。

然后编译插桩版本：`ant instrument`。

安装：`ant installi`。

## Step 3 测试 ##

启动插桩版本：
    `adb shell am instrument net.clasnake.project/net.clasnake.project.EmmaInstrument.EmmaInstrumentation`

进行测试，结束后使用后退键退出应用。

从设备中得到coverage.ec：`adb pull /mnt/sdcard/coverage.ec`

从/folder/bin中得到coverage.em，该文件中包含了待测APP的结构信息，将其与coverage.ec放置同一目录下，然后生成覆盖率报告：
`java -cp ~/adt/sdk/tools/lib/emma.jar emma report -r html -in coverage.em,coverage.ec`。

在同目录下的coverage文件夹下生成覆盖率报告：

![]({{site:url}}/assets/images/posts/2014-10-30-emma_for_android/emma.jpg)

### 参考引用 ###

- [EMMA](http://emma.sourceforge.net/)
- [DynoDroid](http://www.cercs.gatech.edu/tech-reports/tr2012/git-cercs-12-09.pdf)