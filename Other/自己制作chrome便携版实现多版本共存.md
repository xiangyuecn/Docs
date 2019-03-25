# 自己制作Chrome便携版实现多版本共存


> 本文只针对Windows下的Chrome浏览器的使用。
>
> 有时候我们需要使用老版本Chrome，或者仅仅体验一下最新版。
> 
> 上古时代有IETester用来测试多个IE版本，和本机的IE不冲突。
> 
> Chrome别人也制作了很多便携版，**但不知道有没有加料**。

本文介绍一个自己制作便携版的方法：
- 支持任意版本Chrome
- 自己存手工制作，简单安全可靠
- 不影响Windows系统内已安装的Chrome，便携版的数据存储在自己的目录内


## 原理

利用`GoogleChromePortable.exe`启动器来启动Chrome主程序，所有Chrome用户数据都指向本程序所在的`Data`目录，从而实现和系统安装的Chrome隔离。

## 制作步骤

### 【1】提取启动器

下载[`Google Chrome Portable` https://portableapps.com/apps/internet/google_chrome_portable](https://portableapps.com/apps/internet/google_chrome_portable)  ，不要安装，用7-Zip打开这个压缩包，根目录下面有一个`GoogleChromePortable.exe`文件，提取出来，这个文件就是我们需要的启动器。

![提取GoogleChromePortable](%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98_files/1.png)

> 注：你会发现这个文件的数字签名是`2016-11-19`，生命力顽强的一个软件。

另外这个安装包内有`help.html`，介绍了`GoogleChromePortable.exe`如何使用，和参数，可全部提取出来查看。

你可以不用自己提取，可以下载我提取好的， 373 k大小，可验证签名，[https://github.com/xiangyuecn/Docs/raw/master/Other/自己制作chrome便携版实现多版本共存_files/GoogleChromePortable.exe](https://github.com/xiangyuecn/Docs/raw/master/Other/%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98_files/GoogleChromePortable.exe)。


### 【2】提取Chrome主程序
下载需要的任意Chrome版本版本离线安装包，你可以自行搜索，这里有一个版本比较全的地址：https://www.chromedownloads.net/chrome64win-stable/ ，下载完后注意检查数字签名。

离线安装包下载好后，不要运行，我们同样用7-Zip打开这个压缩包，会发现里面有一个`chrome.7z`文件，我们把他提取出来。

![提取chrome.7z](%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98_files/2.png)

> 注：如果你打开看到的是`102~`这种，不是`chrome.7z`的话，说明你下载的不是离线安装包，这种是离线升级安装的，从chrome官网下载到的一般是这种。
>
> 另外离线安装包的图标比升级包的丑很多，不信你看下面的图片

![正确的离线安装包](%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98_files/3.png)

### 【3】制作便携版
**步骤：**
1. 新建一个文件夹，用来存放便携版，比如`41`文件夹 (我下载的Chrome 41这个版本)。
2. 复制`GoogleChromePortable.exe`到这个文件夹，可以改名成自己想要的名称，比如`Chrome41.exe`。
3. 新建`App`文件夹，把`chrome.7z`解压到这个目录内，注意只要`Chrome-bin`文件夹，完成后的目录结构应该是`/41/App/Chrome-bin`。

这样就完成制作了，非常简单。双击`GoogleChromePortable.exe (Chrome41.exe)`就能启动这个Chrome了。

![准备完毕](%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98_files/4.png)

第一次运行会在文件夹内生成`Data`目录，里面存放的是这个版本的用户数据，和系统内安装的Chrome不冲突，也不影响。

把`Chrome41.exe`生成一个快捷方式到桌面，多个Chrome想用哪个用哪个，本人独爱`41.0.22x`这个古董版本，因为有很多好用特性是新版本所废弃的。


## 温馨提示

涉及到的所有软件下载完成后记得检查数字签名，如果没有签名或者签名失效，请立即删除，重新去别的地方下载！！！不然本文没有意义。

![检查可靠性](%E8%87%AA%E5%B7%B1%E5%88%B6%E4%BD%9Cchrome%E4%BE%BF%E6%90%BA%E7%89%88%E5%AE%9E%E7%8E%B0%E5%A4%9A%E7%89%88%E6%9C%AC%E5%85%B1%E5%AD%98_files/5.png)


`GoogleChromePortable.exe`运行后，把浏览器关闭后，这个进程可能不会自动退出，应该是秀逗了，哈哈，正常情况下应该是会和Chrome.exe主进程一块退出。

另外：[用户数据不能在多台电脑之间共享](https://portableapps.com/node/42637)，运行中安装的扩展和cookies等信息在另外一台电脑上打开时将会丢失。便携特性只针对Chrome主程序本身，不含用户数据；多版本共存不受此影响。

本文涉及到Github：https://github.com/xiangyuecn/Docs/tree/master/Other ，里面有本文的所有资源。



# :star:捐赠
如果这个库有帮助到您，请 Star 一下。

你也可以选择使用支付宝给我捐赠：

![](https://github.com/xiangyuecn/Recorder/raw/master/.assets/donate-alipay.png)  ![](https://github.com/xiangyuecn/Recorder/raw/master/.assets/donate-weixin.png)
