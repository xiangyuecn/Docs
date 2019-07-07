> 回到本初，看到多年前写的一段移动端App内嵌入的H5兼容处理代码，有段专门兼容处理输入框类型的代码：
> - 针对`Android 5.0.1,5.0.2` `time`类型的输入框统统改成`text`类型（当年的记忆犹新：这两个版本有些手机上的弹框居然只有重置和取消两个按钮，被客户叼了一顿）;
> - 不管是`IOS`还是`Android`，`datetime,datetime-local`统统使用`text`类型，手输比默认的弹框选择器高效多了。
> 
> 然后前些天在`Android 9.0`爪机上试了一下`webview`默认的时间日期选择，还是当年的 **傻大黑粗** 未曾改变，故事就这样开始了......

[TOC]

在Android App里面嵌入了`webview`做不可描述的事情，因为网页里面的默认原生时间选择器长的太丑，并且兼容的不确定性太多，正好分类性质（商品分类、省市区这种存在上下级关系）的弹出组件太丑想要升级一下，于是就开始着手写设计和编写一个通用的选择器。

**最后花了3天多时间打磨，终于大功告成，手机上使用一下，被惊艳到了٩(๑❛ᴗ❛๑)۶**

![功能图](%E8%AE%BE%E8%AE%A1%E5%92%8C%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AA%E5%BC%82%E6%AD%A5%E9%80%9A%E7%94%A8Picker%E9%80%89%E6%8B%A9%E5%99%A8%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%97%B6%E9%97%B4%E6%97%A5%E6%9C%9F%E3%80%81%E5%9F%8E%E5%B8%82%E3%80%81%E5%95%86%E5%93%81%E5%88%86%E7%B1%BB%E7%9A%84%E9%80%89%E6%8B%A9_files/1.png)

![最终实现效果](%E8%AE%BE%E8%AE%A1%E5%92%8C%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AA%E5%BC%82%E6%AD%A5%E9%80%9A%E7%94%A8Picker%E9%80%89%E6%8B%A9%E5%99%A8%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%97%B6%E9%97%B4%E6%97%A5%E6%9C%9F%E3%80%81%E5%9F%8E%E5%B8%82%E3%80%81%E5%95%86%E5%93%81%E5%88%86%E7%B1%BB%E7%9A%84%E9%80%89%E6%8B%A9_files/2.gif)

本文主要起到记录和参考作用，不管是用在H5移动端的设计、还是Android、IOS的界面设计，包括里面交互的设计，都是有意义的；但并没有开源的打算(和自己的库结合得太紧，剥不出来，自己的库又依赖了swiper4)，[点此体验>>](https://jiebian.life/start/test/app?picker=1)

 ![](%E8%AE%BE%E8%AE%A1%E5%92%8C%E7%BC%96%E5%86%99%E4%B8%80%E4%B8%AA%E5%BC%82%E6%AD%A5%E9%80%9A%E7%94%A8Picker%E9%80%89%E6%8B%A9%E5%99%A8%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%97%B6%E9%97%B4%E6%97%A5%E6%9C%9F%E3%80%81%E5%9F%8E%E5%B8%82%E3%80%81%E5%95%86%E5%93%81%E5%88%86%E7%B1%BB%E7%9A%84%E9%80%89%E6%8B%A9_files/qr.png)


# 一、功能规划
事先并没有画图，只是脑子里面过了一遍，上面的功能图是事后画的。

1. **必须支持异步**，因为异步可以当同步使用，同步只能同步到死；比较大的分类数据用异步加载食用效果更好；时间日期这种，虽然是异步调用，但数据是立即返回的，所以本质上还是同步的。
2. **支持默认值的异步初始化**，给个默认值，异步下不管几级分类都能选中这个默认值。
3. **弹框界面需要有个清空按钮**，相当于把输入框的内容删除，但当用户进行了选择动作时，这个按钮改成返回（观摩了很多Android、H5的Picker，无一例外没有带这种功能，只有返回，但我觉得是非常重要的）；
4. **点击空白区域返回**；
5. **弹框界面每列需支持标头**（表头？），每列都能定义自己的名字或便于识别的标志；
6. **美观、用户操作上符合主流的操作方式**。


# 二、最底层基础实现

## （1）Picker界面和功能实现
这是最底层部分，只定义数据格式和展示风格，具体数据由使用者通过异步回调提供。

**只负责：**
- √ 定义接收数据的格式和接收数据;
- √ 弹出界面和更新界面;
- √ 用户交互;

**不管：**
- × 数据是什么;
- × 数据有多少;
- × 数据分多少级，`1 - ∞`都是可以的，只要显示的下;
- × 数据的有效性（包括是否允许子级缺失）;
- × 更细的显示外观、控制。

**因此可以定义为（摘录的一部分注释和伪代码仅供参考，下同）：**
``` javascript
Picker=func(set,onChange,onCancel){...}

其中最为核心的设置参考：
set={
    value:any //初始值。注意：null为特殊值，代表没有任何选择，其他类型的空值应该转换成这个标准空值，比如0需要转换成null
    ,title:"标题" //当然可以是手写的html，自定义样式
    
    ,columns:[ //定义选择列，限定了选择层级数量
        {
            name:"列名称"
            ,weight:1 //列宽度权重
            ,... //更多列风格配置
        }
        ,...
    ]
    
    ,load:func //加载指定选项的子级列表数据
        /*load(vals,onLoad,onError)
            vals=[level0,level1,....levelx]
            当数据成功加载时使用者需要回调onLoad(childs)，此处定义了数据的格式...
            出错回调onError(msg)，包括数据无效时也是此回调
        */
    ,resolve:func //对初始值value进行反向解析出所有上级，如果初始值value=null空值时不会进行解析调用
        /*resolve(value,onLoad,onError)
            onLoad(vals) 反向解析出来的层级列表[level0,level1,...,value]
            onError(msg)
        */
}
```

界面实现上参考大部分开源的选择器样式，挑个美观的照着画和配色就ok啦；最后总结出来一个比较好使用的界面：选择器显示7行候选项，每行45px，在观看和操作上都是比较优良的；上面的gif因为要截图所以设定了5行；还要留意一下滚动选项时如果滚动组件没有回调，我们可以通过监控选中位置变化来强制刷新界面，swiper偶尔动快了会丢失回调，还好处理手段蛮多。

最大的挑战还是**应对复杂多变的配置项和组合逻辑**，如何应用到界面里面，不过我有100行不到的过气html模板解析引擎，反正随意到没朋友，再复杂的界面也应对自如，写完这个选择器还特地更新了一下文档，[前往GitHub BuildHTML围观](https://github.com/xiangyuecn/BuildHTML)。


## （2）不同类型的选择器基础实现
Picker已经搞定啦，但针对不同的数据源，我们还是要封装一下，不然每个类型的选择器直接调用Picker那会太复杂了，比如：时间、日期的操作可以共享很多相同代码，异步类型的`load`、和`resolve`数据请求部分可以进行一次封装。

**因此就分成了3部分：**
1. **时间日期类**，这部分分为`Time`、`Date`、`DateTime`，他们有部分逻辑可用共用，比如时间的计算，但界面上是不同的，此处不进行分解，放到后面的数据源部分进行分解;
2. **同步类 Type Sync**，此种类型数据是已全部提前准备好，不存在`load`、`resolve`复杂异步操作；虽然同种具体类型的界面和异步的完全一样，但还是要单独分开为一类。
3. **异步类 Type Async**，封装好`load`、`resolve`这些低级繁重操作给上层具体类型使用。

**`同步类`、`异步类`两个方法定义为：**
``` javascript
PickerType=func(set,onChange,onCancel)

其中最为核心的设置参考：
set={
    value:123 //默认值
    ,title:"请选择"
    ,data:{} //必填，完整的类型数据，具体数据格式在这里统一定义，会自动转成Picker需要的格式
    
    ,allowLose:false //是否允许有的选项没有下一级，当然不允许啦，如果缺失了下级，`load`的时候会直接走错误回调
    
    ,columns:[] //必填，为Picker.columns选项
    
    ,picker:{} //picker配置，columns、title不用在这里写
    
    ,itemFormat:func //对选项进行格式化，比如选项名称特殊处理一下
    ,itemsSort:func //对选项列表进行排序
}
```

```
PickerTypeAsync=func(set,onChange,onCancel)

其中最为核心的设置参考：
set={
    extend PickerType.set +* -data
    //和PickerType的基本相同，只是没有data数据而已，增加下面两个
    
    type:"load 要加载的数据类型" //load、resolve应该调用后端统一的一个接口，通过type参数控制加载具体类型的数据
    ,hotData:[] //可选热启动数据，比如前几级的完整数据比较小可以预先加载
}

另外此函数应该对load、resolve获取到的数据进行缓存，避免每滑动一下就请求服务器
```


# 三、数据源层

## （1）时间日期
`Time`、`Date`、`DateTime`选择器除了界面不一样外，数据基本相似：
1. 都可以限定大小区间;
2. `DateTime`的计算就包括了`Time`、`Date`两个的实现;

**可以抽象出两个方法搞定这个3个具体类型的数据生成：**
（1）通过`[年、月]`提供0-2个上级，就能生成年、月、日3个级别的列表数据：
``` javascript
/*生成日期部分的js完整代码
set提供大小范围的Date实例
vals为年、月取值
    vals=[] 生成年份列表
    vals=[2010] 生成2010年的月份列表
    vals=[2010,2] 生成2010年2月的天数列表

如genDate({min:new Date("2012-01-01"),max:new Date("2012-02-06")},[2012,2]) 当然set是在初始化时就准备好的，不可能这样写
*/
function genDate(set,vals){
    var min=set.min;
    var max=set.max;
    
    var a,b;
    var minY=min.getFullYear(),maxY=max.getFullYear();
    var minM=min.getMonth()+1,maxM=max.getMonth()+1;
    var y=vals[0],m=vals[1];
    var fixed=-2;
    if(vals.length==0){
        a=minY;
        b=maxY;
        fixed=-4;
    }else if(vals.length==1){
        a=y==minY?minM:1;
        b=y==maxY?maxM:12;
    }else{
        a=y==minY&&m==minM?min.getDate():1;
        if(y==maxY&&m==maxM){
            b=max.getDate();
        }else{
            if("|1|3|5|7|8|10|12|".indexOf("|"+m+"|")+1){
                b=31;
            }else if(m==2){
                if(y % 4 == 0 && y % 100 != 0 || y % 400 === 0){
                    b=29;
                }else{
                    b=28;
                };
            }else{
                b=30;
            };
        };
    };
    
    var rtv=[];
    for(var i=a;i<=b;i++){
        rtv.push({
            text:("0"+i).substr(fixed)
            ,value:i
        });
    };
    return rtv;
};
```

（2）通过`[时]`或`[年、月、日、时]`提供0-1个(`Time`) 或3-4个(`DateTime`)上级，就能生成时、分2个级别的列表数据：
``` javascript
/*生成时间部分的js完整代码
set提供大小范围的日期或时间数字
vals为年、月、日、时取值，前3个在DateTime类型时才有，不然就是Time类型
    vals=[] 生成小时列表
    vals=[22] 生成分钟列表

如Time类：genTime({min:10*60+56,max:21*60+3},[21])
如DateTime类：genTime({min:new Date("2012-01-01 10:56"),max:new Date("2012-02-06 21:03")},[2012,2,6,21])
*/
function genTime(set,vals){
    var min=set.min;
    var max=set.max;
    var h=vals[0];
    if(vals.length>2){//DateTime
        var y=vals[0],m=vals[1],d=vals[2];
        if(y==min.getFullYear()&&m==min.getMonth()+1&&d==min.getDate()){
            min=min.getHours()*60+min.getMinutes();
        }else{
            min=0;
        };
        if(y==max.getFullYear()&&m==max.getMonth()+1&&d==max.getDate()){
            max=max.getHours()*60+max.getMinutes();
        }else{
            max=23*60+59;
        };
        h=vals[3];
    };
    
    var a,b;
    var minH=Math.floor(min/60),maxH=Math.floor(max/60);
    if(h==null){
        a=minH;
        b=maxH;
    }else{
        a=h==minH?min%60:0;
        b=h==maxH?max%60:59;
    };
    
    var rtv=[];
    for(var i=a;i<=b;i++){
        rtv.push({
            text:("0"+i).substr(-2)
            ,value:i
        });
    };
    return rtv;
};
```

**有了这两个方法，我们就可以写着3个类型的具体实现啦：**
``` javascript
PickerTime=func(set,onChange,onCancel)
PickerDate=func(set,onChange,onCancel)
PickerDateTime=func(set,onChange,onCancel)

3个最为核心的设置都基本类似：
set={
    min:123 ||"00:00" //最小时间
    max:123 ||"23:59" //最大时间
    value:123 ||"10:01" //设定时间，如果为null为当前时间部分
    
    title:"选择时间"
    picker:{} //Picker更多配置项
}


各类型内部调用Picker时load写法
PickerTime
load:function(vals,onLoad,onError){
    onLoad(genTime(set,vals));
}

PickerDate
    onLoad(genDate(set,vals));
    
PickerDateTime
    onLoad(vals.length>2?genTime(set,vals):genDate(set,vals));
```
这3个类型直接调用的`Picker`方法，在内部生成`columns`、`load`、和`reverse`配置项，使用者无需关系这些最底层的复杂配置。


## （2）多级同步分类，如：城市
因为在Picker之上已经实现了同步类的选择器`Type Sync`，因此我们只需要直接调用`PickerType`这个同步方法，传入分类数据即可。

比如省市区3级的选择，我们就把城市省市区3级数据一股脑的加载到页面里即可。

## （3）多级异步分类，如：城市
因为在Picker之上已经实现了异步类的选择器`Type Async`，因此我们只需要直接调用`PickerTypeAsync`这个同步方法，传入要异步加载的类型即可，类型可以是：省市区这种城市、也可以是商品分类，甚至很古怪的分类也可以支持。

比如省市区镇4级的选择，我们只需要把`type="city"`之类的设置一下就ok啦；为了提升响应速度，可以预先把省市区3级加载为热数据。

另附：[GitHub AreaCity-JsSpider-StatsGov 省市区镇数据](https://github.com/xiangyuecn/AreaCity-JsSpider-StatsGov)，一年来还是更新的蛮勤快的，我自己在用，还有快1000的star啦。



# 四、最终的调用层
如果直接使用`Picker`，那会折磨死人，因为要写复杂的数据加载和解析函数。

因此有了上一层的封装：`PickerTime`、`PickerDate`、`PickerDateTime`、`PickerType`、`PickerTypeAsync`。

**但这些功能还是需要一个个手动调用，不够简单，我想要：**
1. 给个`dom节点`(比如输入框)，赋个城市ID，自动转换成省市区名字显示;
2. 点击`dom节点`，自动弹出选择，选择完后自动更新名字显示;

于是我进一步对`Picker*`进行了封装，得到了最顶上的两层，而真正使用的也就是这两层，很少会去调用太过底层的`Picker*`。

这两层是用我自己最为得意的编写习惯来写的，别人看到了这种写法可能会吐，我就不特别介绍了，其实也没有什么好介绍的，最终结果就是本文开头的那张gif图里面的那些表单，可点击、点击自动弹出Picker。如果感兴趣，可以在控制台里面查看一下这些`dom节点`就知道咋实现的啦。

-------
**> 完 <**