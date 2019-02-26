# 应用场景
仅仅应用于单页应用的滑动操作，用`swiper4.x`接管页面的滚动操作。用来支持顶部和尾部的回弹效果，进一步来支持常见那种下拉刷新动画效果。不适用于轮播图那种应用场景。

虽然只是针对`swiper4.x`，但相关原理，在别的框架中也是有参考意义的。


# 出现的问题

## 一、惯性动画不会在触摸时停止
快速滑动页面，手离开屏幕时产生的惯性动画还在运行时，此时触摸屏幕，动画不会停止。导致连续快速滑动页面，看起来有跳来跳去的感觉。

## 二、快速滑动手势大概率识别成慢滑动手势
连续快速多滑几下，有那么1、2下明显能够感知到滑动惯性动画变慢的情景，给人的感觉就是滑动不流畅。

## 三、惯性动画时长不合理
不管是很慢的滑动，还是很快的滑动，惯性动画时长只能设置一个固定值。慢慢移动松手后也会有一个很长的动画，快速滑的时候动画又有点短，综合看起来给人很卡的感觉。


# 解决手段

除了第一个问题，另外两个问题不修改`swiper4.x`源码似乎无法克服。

## 一、解决惯性动画不会在触摸时停止的问题
滑动时手离开屏幕时产生的惯性动画是`css动画`，并未直接提供停止动画的方法。动画还在运行时如果触摸屏幕，不改源代码，那我们就监听一下`touchStart`事件。

用`setTranslate(translate)`可以移除这个动画css，需要提供页面当前的滚动位置。用`getTranslate()`就行了：
 > 这与通过属性`mySwiper.translate` 获取到的数值稍有不同，即使是在过渡时（`animating`）也能获取到，而后者精度较高

### 解决方案代码
``` javascript
//在touchStart事件中执行
mySwiper.setTranslate(mySwiper.getTranslate());
```

## 二、解决快速滑动手势大概率识别成慢滑动手势问题
这个必须修改源代码才能解决，问题出在计算惯性动画起始速度时的计算参数精度不足。

### 原代码
```
if (params.freeModeMomentum) {
        if (data.velocities.length > 1) {
          var lastMoveEvent = data.velocities.pop();
          var velocityEvent = data.velocities.pop(); //此处取值方式会导致精度不够

          var distance = lastMoveEvent.position - velocityEvent.position;
          var time = lastMoveEvent.time - velocityEvent.time;
          swiper.velocity = distance / time;
          swiper.velocity /= 2;
```
可以看出，他计算起始速度`velocity`参考的是`onTouchMove`最后记录的两个点，单纯取最后两个点是不可靠的，可能因为最后两次`onTouchMove`触发间隔比较长，导致计算出来的速度过低。从而导致偶尔会快速滑动但惯性效果是慢滑动的效果。

如果触发间隔很短，导致动画速度变快，变快了其实感知上并无区别。最要命的还是变慢，感觉很卡一样。

### 解决方案
一个很短的时间内，人的滑动方向不太可能会产生变化，但这个短时间内产生的滑动位移能够很好的代表手势结尾的滑动速度。

经过反复尝试，用100毫秒内的位移来计算惯性起始速度最好，100毫秒内，如果是快速滑动，会触发多次`onTouchMove`，计算出来的速度是很接近实际的手势速度。

问题的解决指向了100毫秒内的第一个点。解决代码：
``` javascript
if (params.freeModeMomentum) {
        if (data.velocities.length > 1) {
			//fix 惯性动画
			var velos=data.velocities;
			var firstEvent=velos[0];
			var lastMoveEvent = velos.pop();
			var velocityEvent =velos.pop();
			for(var i=velos.length-1;i>=0;i--){//找出100毫秒内的起始位置
				var velo=velos[i];
				if(lastMoveEvent.time-velo.time>100){
					break;
				}
				velocityEvent=velo;
			}

          var distance = lastMoveEvent.position - velocityEvent.position;
          var time = lastMoveEvent.time - velocityEvent.time;
          swiper.velocity = distance / time;
          swiper.velocity /= 2;
```

## 三、解决惯性动画时长不合理问题
慢滑动时惯性应该很小，动画很短；快速滑动时惯性应该很大，动画很长。`swiper4.x`只能提供一个固定的惯性动画时长，不改源代码是解决不了的。

### 原代码在第二问代码下面一点点
``` javascript
var momentumDuration = 1000 * params.freeModeMomentumRatio; //写死了固定动画时长
```

### 优化动画
其实也简单，小的就小，大的就大，用初始速度`velocity`来做乘法运算即可达到效果。
``` javascript
var momentumDuration = Math.abs(swiper.velocity) * 1000 * params.freeModeMomentumRatio;
```
结果：比原代码好很多，但跟别的app里面的滑动还是区别蛮大，慢滑时还是太快了点。

App原生滑动动画，感知上是慢的更慢，快的更快。轻微滑动慢的要死，快速动一点点，飞快。

![](%E8%AE%B0%E5%BD%95%E4%B8%80%E4%B8%8B%E5%AF%B9swiper4.x.js%E5%9C%A8H5%E5%8D%95%E9%A1%B5%E4%B8%AD%E7%9A%84%E6%BB%91%E5%8A%A8%E4%BC%98%E5%8C%96_files/1.png)

对数曲线！1-0范围蛮符合，缓的地方比线性的还缓，陡的地方奇陡无比。在上面优化的结果基础上用对数加持一下，效果非常不错。另外限定最长动画不超过3秒。
``` javascript
//取对数曲线优化一下0.5-1.5倍之间，慢的越慢，快的越快
momentumDuration*=-Math.log10(Math.min(0.3,Math.max(0.03,(3000-momentumDuration)/3000)));
```

### 解决方案代码
``` javascript
//根据初始速度来确定惯性动画时长
var momentumDuration = Math.min(3000,Math.abs(swiper.velocity) * 1000 * params.freeModeMomentumRatio);
//取对数曲线优化一下0.5-1.5倍之间，慢的越慢，快的越快
momentumDuration*=-Math.log10(Math.min(0.3,Math.max(0.03,(3000-momentumDuration)/3000)));
momentumDuration=Math.min(3000,momentumDuration);
```

# 最终结果
应用`swiper4.x`修改后的单页滚动效果，已经很接近App的滚动效果，虽然还是会有些细微的抖动卡顿感，但要做到完全和App一样的流畅效果，估计蛮难，当前算是已经蛮优了。

还是iscroll省心些，但已经上了`swiper4.x`这条船了就算了，迁移过来又迁移过去白折腾。

改良后的滚动效果，目前[2019-02-27]号以后可以到 [https://jiebian.life/start/xcx/tool_jieri](https://jiebian.life/) 体验（这个时间之前新版本应该还没有上线），小程序和H5共用的页面。

