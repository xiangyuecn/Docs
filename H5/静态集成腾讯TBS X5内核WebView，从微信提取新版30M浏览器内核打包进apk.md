> 由于X5内核打包后有30多M，x5官网文档也着重提醒：“由于内核体积较大，官网SDK默认为静默下载方案，首次使用需要在网络中静默下载约30M的内核，可能存在一定的加载失败率，目前线上加载成功率约为90%。如果您有业务需要强依赖X5内核相关功能，请使用静态集成方式进行集成”。
> 
> 当某些特殊app必须可靠加载X5内核时（非普通用户使用，没有装微信），等待网络慢慢下载还不一定成功，那就太废了；似乎静态集成给了一条活路。
> 
> 关键就在于这个“静态集成”，整个x5官网，除了常见问题中有提到，其他任何地方都找不到如何静态集成，更找不到SDK；后面发现是早年有提供文档和下载，不过后面就关闭了，文档也删了；也许是用的人多，又不挣钱，KPI下的产物吧，没利谁给你免费维护。
> 
> 经过搜罗历史资料（GitHub搜代码真稳的一比），结合下载到的老版本静态集成SDK，经过一番摸索，实现了：用老版本的jar + 最新的TBS X5内核，进行静态集成，将内核直接打包进APK。

# 前情提要
本篇文章只针对X5内核的静态集成，将30多M内核直接打包进Apk，TBS X5官网已经没有相关资料了；如果你不是要把内核打包进Apk，请直接阅读官网 https://x5.tencent.com/ 文档就行了，不用折腾。

![](%E9%9D%99%E6%80%81%E9%9B%86%E6%88%90%E8%85%BE%E8%AE%AFTBS%20X5%E5%86%85%E6%A0%B8WebView%EF%BC%8C%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%8F%90%E5%8F%96%E6%96%B0%E7%89%8830M%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8%E6%89%93%E5%8C%85%E8%BF%9Bapk_files/1.png)

# 第一步：下载老版本SDK得到jar

## 获取SDK
当前时间：2020-08-07，能找到的最新的一个静态集成SDK，看里面文件名时间是2017-10-11，虽然老了点，但是能用来加载最新的X5内核，没有问题。

老版本SDK下载地址：http://soft.tbs.imtt.qq.com/17421/tbs_res_imtt_tbs_release_integrateWithX5core_43500SDK_43656Core.zip

下载后，只需要提取里面的`tbs_sdk_thirdapp_v3.5.0.1063_43500_staticwithdownload_withoutGame_obfs_20171011_195714.jar`，另外一个apk文件是内核（这个内核太老，就不要了）。

如果上面地址失效了，我在github里面存了一份jar，地址：
``` html
https://github.com/xiangyuecn/Docs/blob/master/H5/静态集成腾讯TBS X5内核WebView，从微信提取新版浏览器内核打包进apk_files/tbs_sdk_thirdapp_v3.5.0.1063_43500_staticwithdownload_withoutGame_obfs_20171011_195714.jar
```

![](%E9%9D%99%E6%80%81%E9%9B%86%E6%88%90%E8%85%BE%E8%AE%AFTBS%20X5%E5%86%85%E6%A0%B8WebView%EF%BC%8C%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%8F%90%E5%8F%96%E6%96%B0%E7%89%8830M%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8%E6%89%93%E5%8C%85%E8%BF%9Bapk_files/2.png)

## 集成SDK
把得到的这个tbs_sdk_*.jar copy到项目的libs目录中（如果你已经导入了新版的`tbs_sdk_*.jar`，把新版删掉就行，它们没有QbSdk.preinstallStaticTbs静态内核加载方法）。

这样SDK就集成好了（见文末图），此时就算不集成静态内核也能正常运行，就是x5加载不太稳定。



# 步骤二、下载提取最新TBS X5内核

有两种方法，一个是从微信里面提取；另外一个就是app内访问tbs调试页面，然后安装新内核，再提取。

## 方法一：从微信中提取

微信中打开`http://debugtbs.qq.com`，进入界面后点击`拷贝内核`按钮，会弹出保存的路径（参考顶上【图3】），打开这个路径后提取里面的`core_private/x5.debug.tbs`这个文件，其实这个是一个apk/zip文件，30多M，解压后得到一堆so和jar等文件，先复制出来再说，文件名加一个zip后缀。

X5官网中 `关于TBS` -> `平台适配` 中已经写明了只支持`armeabi、armeabi-v7a、arm64-v8a` 这3种架构，因此对于`x86`等架构是不支持的（AS模拟器），抛开模拟器，大部分似乎只需提供`armeabi`架构就ok了。

微信中提取出来的这个内核的架构可能是`arm64-v8a`，如果你要`armeabi`架构，请用下面方法二来获取内核。

## 方法二：App内内访问tbs调试页安装新内核

集成了上面的老版本SDK后，你的App就可以通过访问`http://debugtbs.qq.com`页面来手动下载最新
X5内核（如果下载不了，尝试改回新版本SDK下载，然后再改回来）。点击`安装线上内核`（参考顶上【图1】），它会自动识别App的架构，下载到`armeabi`或者`arm64-v8a`架构的TBS内核包，下载完后重启App就可以进行内核提取操作了。

这就有两种途径获取到内核包了，一个是在安装时监控App的网络请求，得到下载地址；另外一个就是和微信里面一样，点击`拷贝内核`按钮，参考上面微信的流程。

这里提供我拿到的一个内核下载地址：
``` html
http://soft.tbs.imtt.qq.com/17421/tbs_res_imtt_tbs_release_integrateWithX5core_43500SDK_43656Core.zip

30多M，内核版本：45318（20200714112122），Chrome 77
```

# 集成内核到App中

## 解压内核得到so
内核就是一个apk文件，直接zip解压就行了，里面有`lib` + `assets` 两个目录，我们要把这两个目录内的所有文件合到一起放到一个目录再操作：

`lib`目录内可能是`armeabi` 或者 `arm64-v8a`，不同架构是因为是根据你App架构（或微信） + 手机支持的架构TBS自动下载的，如果你需要的架构类型和`lib`里面的不同，那么请参考上面重新提取内核。

将lib/arm*内的so文件复制出来，和`assets/webkit`内的文件放到一起，总共一起共40来个文件。

![](%E9%9D%99%E6%80%81%E9%9B%86%E6%88%90%E8%85%BE%E8%AE%AFTBS%20X5%E5%86%85%E6%A0%B8WebView%EF%BC%8C%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%8F%90%E5%8F%96%E6%96%B0%E7%89%8830M%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8%E6%89%93%E5%8C%85%E8%BF%9Bapk_files/3.png)

## so改名
将刚才复制到一起的一堆文件，文件名统统加上`libtbs.`前缀 + `.so`后缀，比如`abc.so`文件，改名后变成`libtbs.abc.so.so`。

附：CMD命令行批量改名
``` bat
::直接在当前这堆文件的目录执行下面代码，批量改名
for /F %i in ('dir /A:-D /B') do move %i "libtbs.%i.so"
```

![](%E9%9D%99%E6%80%81%E9%9B%86%E6%88%90%E8%85%BE%E8%AE%AFTBS%20X5%E5%86%85%E6%A0%B8WebView%EF%BC%8C%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%8F%90%E5%8F%96%E6%96%B0%E7%89%8830M%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8%E6%89%93%E5%8C%85%E8%BF%9Bapk_files/4.png)

## 集成内核
将改好名的所有文件，copy到项目的`src/main/jniLibs/armeabi`目录中（如果是`arm64-v8a`的一样copy），这样内核就集成好了（见文末图）。下面我们只需要在app运行时激活这个内核就ok了。

记得build.gradle中配置上ndk，如
``` java
defaultConfig {
	...
	ndk {abiFilters "armeabi","x86"} //真机 + 模拟器，谈性能？不如喂狗
}
```


## 激活X5内核
不要用QbSdk.initX5Enviroment方法，改用QbSdk.preinstallStaticTbs方法。

在显示webview前，你要先把X5内核安装好（我管他叫激活）。代码就一句话，你可以在Application里面，或者当前Activity（你得激活完后再手动创建WebView）里面执行：
``` java
//此方法非常耗时，应当开个线程
QbSdk.preinstallStaticTbs(getApplicationContext());

//这里就可以安全的创建WebView了，只要你的ABI架构没有问题，那么这里一定能加载到X5内核
```

简单点就在Application里面激活就行，比如这样子：
``` java
public class MyApp extends Application{
	public static Boolean X5Ok=null;

	@Override
	public void onCreate(){
		super.onCreate();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				X5Ok=QbSdk.preinstallStaticTbs(getApplicationContext());
			}
		}).start();
	}
}

/*使用的时候就在Activity里面来点暴力吧：
	onCreate(...) {
		while(MyApp.X5Ok==null){ System.currentTimeMillis(); }
		
		super.onCreate(...);
		setContentView(...);
		
		if(!MyApp.X5Ok){
			throw new RuntimeException("什么垃圾玩意");
		}
	}
*/
```


# 其他的一些配置，参考官网就OK

## 权限
```
<uses-permission ....
抱歉，先毛权限也不给。你App本来需要什么权限，就给什么权限（网络、录音、摄像头），不用管X5官网的那一坨。
等他崩溃再一个个给，似乎只要INTERNET、ACCESS_NETWORK_STATE权限就够了。
```

![](%E9%9D%99%E6%80%81%E9%9B%86%E6%88%90%E8%85%BE%E8%AE%AFTBS%20X5%E5%86%85%E6%A0%B8WebView%EF%BC%8C%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%8F%90%E5%8F%96%E6%96%B0%E7%89%8830M%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8%E6%89%93%E5%8C%85%E8%BF%9Bapk_files/5.png)

## 包名替换
Java文件中：`android.webkit.WebView` -> `com.tencent.smtt.sdk.WebView`。

布局文件中：`<WebView />` -> `<com.tencent.smtt.sdk.WebView />`

详细的包名替换阅读X5官网。

## WebView网页权限
当网页访问摄像头、麦克风时，X5默认会弹一个确认对话框，自己的App网页就没有这么多条条框框了，可以静默授权：
``` java
//webkit是 WebChromeClient.onPermissionRequest 处理网页授权
//x5是 IX5WebChromeClientExtension.onPermissionRequest 处理网页授权

webView.setWebChromeClientExtension(new IX5WebChromeClientExtension() {
	...
	@Override
	public boolean onPermissionRequest(String s, long l, MediaAccessPermissionsCallback callback) {
		你的App摄像头、录音权限申请( 申请成功回调{
			long allowed = 0;
			allowed = allowed | MediaAccessPermissionsCallback.ALLOW_AUDIO_CAPTURE | MediaAccessPermissionsCallback.ALLOW_VIDEO_CAPTURE;
			boolean retain = true;
			callback.invoke(s, allowed,retain);
		});
		return true;
	}
	...
});
```

## 首次初始化冷启动优化
用不着，QbSdk.initX5Enviroment方法也用不着。

## 混淆、文件、视频
阅读X5官网文档。


--------
欢迎关注我的GitHub: 

H5、Hybrid App录音库，支持mp3、wav、语音识别：https://github.com/xiangyuecn/Recorder

省市区镇数据，提供坐标、边界，和shp、geojson、sql支持：https://github.com/xiangyuecn/AreaCity-JsSpider-StatsGov

如果本篇文章对你有帮助，您也可以到上面仓库中对作者进行打赏~
