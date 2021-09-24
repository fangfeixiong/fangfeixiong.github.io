---
layout: single
title:  "flutter-unity-view-widget使用问题和解决办法汇总"
date:   2021-09-24
categories: flutter, unity3d
toc: true
---

### 微信小程序分享到Android朋友圈的Beta版功能的使用问题

> #### **解决办法**

- 找到Build.cs文件，路径如下:
 Assets > FlutterUnityIntegration > Editor > Build.cs.

> 对于iOS,找到BuildIOS()函数， 按如下方式进行修改:

```Swift
private static void BuildIOS(String path)
{
        if (Directory.Exists(path))
            Directory.Delete(path, true);

        EditorUserBuildSettings.iOSBuildConfigType = iOSBuildType.Release;

        var options = BuildOptions.AcceptExternalModificationsToPlayer;
        var report = BuildPipeline.BuildPlayer(
            GetEnabledScenes(),
            path,
            BuildTarget.iOS,
            options
        );
......
```
>- 将这一行代码
```Swift
var options = BuildOptions.AcceptExternalModificationsToPlayer;
```
>- 替换为下面的代码
```Swift
var options = BuildOptions.AllowDebugging;
```
> 替换后如下所示

```Swift
private static void BuildIOS(String path)
{
        if (Directory.Exists(path))
            Directory.Delete(path, true);

        EditorUserBuildSettings.iOSBuildConfigType = iOSBuildType.Release;
        //修改了这里
        var options = BuildOptions.AllowDebugging;

        var report = BuildPipeline.BuildPlayer(
            GetEnabledScenes(),
            path,
            BuildTarget.iOS,
            options
        );

......
```


> 对于Android,找到DoBuildAndroid()函数， 按如下方式进行修改:

```Kotlin
public static void DoBuildAndroid(String buildPath, bool isPlugin)
{
        if (Directory.Exists(apkPath))
            Directory.Delete(apkPath, true);

        if (Directory.Exists(androidExportPath))
            Directory.Delete(androidExportPath, true);

        EditorUserBuildSettings.androidBuildSystem = AndroidBuildSystem.Gradle;

        var options = BuildOptions.AcceptExternalModificationsToPlayer;
        var report = BuildPipeline.BuildPlayer(
            GetEnabledScenes(),
            apkPath,
            BuildTarget.Android,
            options
        );

......
```
- 将这一行代码
```kotlin
var options = BuildOptions.AcceptExternalModificationsToPlayer;
```
- 替换为下面的代码
```kotlin
var options = BuildOptions.AllowDebugging;
EditorUserBuildSettings.exportAsGoogleAndroidProject = true;

```

> 修改后:
```kotlin
public static void DoBuildAndroid(String buildPath, bool isPlugin)
{
        if (Directory.Exists(apkPath))
            Directory.Delete(apkPath, true);

        if (Directory.Exists(androidExportPath))
            Directory.Delete(androidExportPath, true);

        EditorUserBuildSettings.androidBuildSystem = AndroidBuildSystem.Gradle;

        //修改了这里
        var options = BuildOptions.AllowDebugging;
        EditorUserBuildSettings.exportAsGoogleAndroidProject = true;

        var report = BuildPipeline.BuildPlayer(
            GetEnabledScenes(),
            apkPath,
            BuildTarget.Android,
            options
        );

......
```
