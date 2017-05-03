---

title: 常用meta标签

date: 2016-10-23 17:49:18

categories: 
- html

tags: 
- head
- meta

---

## 常用类

### 声明文档字符编码

```html
<meta charset="utf-8" />
<!--or-->
<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
```

### IE渲染声明

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
<!--IE=edge:IE将按照最高版本进行渲染-->
<!--chrome=1:Google Chrome Frame(谷歌内嵌浏览器框架GCF)安装后可以使IE按照CHROME内核进行渲染-->
```

<!-- more -->

### 定义描述

```html
<meta name="description" content="这里是对于页面的描述" />
<!--http://www.jd.com-->
<meta name="description" content="京东JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!" />
```

### 定义针对搜索引擎的关键词

```html
<meta name="keywords" content="这里是页面的一些关键词" />
<!--http://www.jd.com-->
<meta name="Keywords" content="网上购物,网上商城,手机,笔记本,电脑,MP3,CD,VCD,DV,相机,数码,配件,手表,存储卡,京东" />
```

### 网站标志

```html
<link rel="icon" type="image/png" href="favicon.png" />
<!--or-->
<link rel="icon" type="image/x-icon" href="favicon.ico" />
```
		
## 移动端

### 响应式布局

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=yes" />
<!--width - viewport的宽度-->
<!--height - viewport的高度-->
<!--initial-scale - 初始的缩放比例-->
<!--minimum-scale - 允许用户缩放到的最小比例-->
<!--maximum-scale - 允许用户缩放到的最大比例-->
<!--user-scalable - 用户是否可以手动缩放-->
```

### IOS设备

#### 添加到主屏幕上书签的图标
        
```html
<!--原图-->
<link rel="apple-touch-icon-precomposed" href="favicon.ico" />
<!--高光（IOS6及以下）-->
<link rel="apple-touch-icon" href="favicon.ico" />
```

#### 添加到主屏幕上书签的标题

```html
<meta name="apple-mobile-web-app-title" content="标题">
```

#### 启用 WebApp 全屏模式

```html
<meta name=”apple-mobile-web-app-capable” content=”yes” />
```

#### 控制状态栏样式（必须启用WebApp全屏模式）    

```html
<meta name=”apple-mobile-web-app-status-bar-style” content=”default” />
<meta name=”apple-mobile-web-app-status-bar-style” content=”black” />
<meta name=”apple-mobile-web-app-status-bar-style” content=”black-translucent” />
```

#### 移动端手机号码识别（IOS SAFARI）
 
```html
<meta name="format-detection" content="telephone=no" />
<!--7位数字，形如：1234567-->
<!--带括号及加号的数字，形如：(+86)123456789-->
<!--双连接线的数字，形如：00-00-00111-->
<!--11位数字，形如：13800138000-->

<!--单独开启电话功能（忽略meta兼容ANDROID）-->
<a href="tel:123456">123456</a> 
<!--单独开启短信功能（忽略meta兼容ANDROID）-->
<a href="sms:123456">123456</a> 
```

### ANDROID设备

#### 邮箱识别（默认开启）
        
```html
<meta name="format-detection" content="email=no" />
<!--单独开启邮箱功能（兼容IOS）-->
<a href="mailto:lovejimmy2014@gmail.com">lovejimmy2014@gmail.com</a>
```

## 其他

### 标注网页作者

```html
<meta name="author" content="" />
```

### 自动刷新并指向新页面
            
```html
<meta http-equiv="Refresh" content="5;url=http://www.w3school.com.cn" />
<!--content为等待时间-->
<!--url为你想要跳转的地址（可不填写）-->
```
       
### 360双核浏览器控制展示内核

```html
<!--若页面需默认用极速核，增加标签（文档模式Chrome 21，支持HTML5，不支持ActiveX控件）：-->
<meta name="renderer" content="webkit">       
<!--若页面需默认用ie兼容内核，增加标签（文档模式为IE6/7，不支持HTML5，支持ActiveX控件）：-->
<meta name="renderer" content="ie-comp">    
<!--若页面需默认用ie标准内核，增加标签（文档模式为IE9/IE10/IE11，取决于用户的IE，支持HTML5，支持ActiveX控件）：-->
<meta name="renderer" content="ie-stand">
```
     
### 缓存机制

```html
<meta http-equiv="cache-control" content="no-cache">      
<!--Public：指示响应可被任何缓存区缓存。-->
<!--Private：指示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这允许服务器仅仅描述当用户的部分响应消息，此响应消息对于其他用户的请求无效。-->
<!--no-cache：指示请求或响应消息不能缓存。-->
<!--no-store：用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存。-->
<!--max-age：指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。-->
<!--min-fresh：指示客户机可以接收响应时间小于当前时间加上指定时间的响应。-->
<!--max-stale：指示客户机可以接收超出超时期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。       -->

<!--一般来讲，强制页面不缓存会用到-->
<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="0">   
<!--但是将引用资源后加MD5更为稳妥有效-->
```
        
	    
### Set-Cookie (cookie设定)

```html
<!--需要强调的是expires必须是GMT格式-->
<meta http-equiv=set-cookie content="name=value;expires=Wed, 28 Oct 2016 10:12:57 GMT; path=/">
```

PS：利用JS可以很轻易得到相对于当前时间的GMT格式时间

```javascript
var tDate = new Date();
tDate.setDate(tDate.getDate()+5);
console.log(tDate.toGMTString());
```
       
### 防止外链

```html
<!--经测试无效，可以采用javascript的方式解决-->
<meta http-equiv="window-target" content="_top">
```

PS：这段代码可以阻断任何框架形式，包括自己站内网页

```javascript
if (top != self) {
    top.location.href = self.location.href;
}
```

想要对自己站内页面区分对待，可以参考阮一峰所写的，[防止网页被嵌入框架的代码（续）](http://www.ruanyifeng.com/blog/2010/08/anti-frameset_javascript_codes_continued.html)        

### WIN8磁铁图标

```html
<!--Windows 8 磁贴颜色：-->
<meta name="msapplication-TileColor" content="#000" />
<!--Windows 8 磁贴图标：-->
<meta name="msapplication-TileImage" content="icon.png" />
```
        


		
		
