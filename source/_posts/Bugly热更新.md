---
title: Bugly热更新
date: 2017-12-14 11:01:54
tags: [热更新]
categories: [技术]
---
// 签名配置

```
signingConfigs {
    release {
        try {
            storeFile file("key.jks")
            storePassword "******"
            keyAlias "key"
            keyPassword "******"
        } catch (ex) {
            throw new InvalidUserDataException(ex.toString())
        }
    }
}

```

必须要配置
多渠道签名和热更新需要的

```
app.build
apply from: 'tinker-support.gradle'
// 多渠道使用walle示例（注：多渠道使用）
apply plugin: 'walle'
```



```
//    热更新 热升级
    compile "com.android.support:multidex:1.0.1" // 多dex配置
    compile 'com.tencent.bugly:crashreport_upgrade:1.3.1'//其中latest.release指代最新版本号，也可以指定明确的版本号，例如1.2.0
    compile 'com.tencent.bugly:nativecrashreport:latest.release' //其中latest.release指代最新版本号，也可以指定明确的版本号，例如2.2.0
    //瓦力的多渠道打包
    compile 'com.meituan.android.walle:library:1.1.3'
```



```
walle {
//    打包执行的命令  gradlew clean assembleReleaseChannels
    // 指定渠道包的输出路径
    apkOutputFolder = new File("${project.buildDir}/outputs/channels");
    // 定制渠道包的APK的文件名称
    apkFileNameFormat = '${appName}-${packageName}-${channel}-${buildType}-v${versionName}-${versionCode}-${buildTime}.apk';
    // 渠道配置文件
    channelFile = new File("${project.getProjectDir()}/channel")
}
```




项目.build

```
// tinkersupport插件, 其中lastest.release指拉取最新版本，也可以指定明确版本号，例如1.0.4
classpath "com.tencent.bugly:tinker-support:1.0.8"
classpath 'com.meituan.android.walle:plugin:1.1.3'

public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        //设置是开发者的设备
        Bugly.setIsDevelopmentDevice(getApplicationContext(),true);
        String channel = WalleChannelReader.getChannel(getApplication());
        Bugly.setAppChannel(getApplication(), channel);

        Bugly.init(this, "******", true);
        Beta.smallIconId = R.drawable.pikaqiu;
        Beta.defaultBannerId  = R.drawable.pikaqiu;
        Beta.canShowUpgradeActs.add(MainActivity.class);
        Beta.storageDir = Environment.
                getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS);
        Beta.initDelay = 1 * 1000;
        Beta.autoInit = false;
    }
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        // you must install multiDex whatever tinker is installed!
        MultiDex.install(base);
        // 安装tinker
        Beta.installTinker();
    }
}
```


这个是  正式版打包（目前上的基准包）


发布 补丁的话  要求


```
里面的
/**
* 此处填写每次构建生成的基准包目录  新版本发布的时候 建议保存一份
*/
def baseApkDir = "app-0630-13-26-10"

// 构建基准包和补丁包都要指定不同的tinkerId，并且必须保证唯一性
tinkerId = "1.0.4"（唯一的一个名字就行 不用和项目的版本号一样）
要更改
安装正式包的  需要安装基带下的apk  或者用瓦力打包  瓦力打包下的 相当于加了基带

修改需要改动的bug代码  

```

**执行buildAllFlavorsTinkerPatchRelease生成所有渠道补丁包**
![image](https://bugly.qq.com/docs/img/hotfix/android/Snip20170209_13.png?v=20170627170213)

得到的patch文件夹的  7zip这个文件就是补丁啦