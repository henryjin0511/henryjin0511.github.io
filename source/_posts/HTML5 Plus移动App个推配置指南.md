---

title: HTML5 Plus移动App个推配置指南

date: 2017-04-11 13:23:12

categories: 
- h5+

tags: 
- 个推
- html5+

---

> 截止本篇文章最后修改日期2017年4月12日，文中测试的数据结构依然有效。

在使用DCLOUD出品的MUI框架做H5+APP开发的最后阶段一般都会接入推送模块，HBuilder基座继承了推送功能，我们只需要配置SDK即可，但是由于官方文档并没有较为完整的测试说明，导致开发过程中调试较为困难，这篇文章是本人经过实际测试各种情况最终总结的一些经验，希望能够给开发者提供一些帮助（如果只想正常使用，建议直接看总结部分）。

通知消息分为普通消息和透传消息，其中普通消息只支持ANDROID平台，且只支持通知、下载和打开网址三种类型的用户点击事件，在个推平台均有明确参数设置，在此不再赘述。本次测试主要针对透传消息，因为实际运用中如果想要进行双端通知并且有自定义事件的监听那么就只能采用透传消息，而透传消息受到OS平台，应用前后台，是否按照官方标准格式提交参数三方面影响，所以本次测试针对以下八种情况：
* ANDROID前台标准格式通知接收
* ANDROID前台非标准格式通知接收
* ANDROID后台标准通知接收
* ANDROID后台非标准通知接收
* IOS前台标准格式通知接收
* IOS前台非标准格式通知接收
* IOS后台标准通知接收
* IOS后台非标准通知接收

<!-- more -->

## 官方标准通知内容格式

```
{title:"通知标题",content:"通知内容",payload:"通知去干嘛这里可以自定义"}
```

## 测试情况

### ANDROID

ANDROID设备的通知栏标题和通知栏内容直接在消息内容中设置`title`和`content`两个字段

<a name="ANDROID前台标准格式通知接收" />
#### ANDROID前台标准格式通知接收（响应click事件）

##### 个推端配置

* 通知标题 : xxx(标记此次通知，无意义类似于备注)
    
* 消息内容 : `{title:"安卓通知栏标题",content:"安卓通知栏内容",payload:{aaa:"自定义参数a",bbb:'自定义参数b',ccc:"自定义参数c"}}`

##### 手机端接受（响应click事件）
    
```javascript
var message = null;
mui.plusReady(function() {
	message = document.getElementById("message");
	plus.push.addEventListener( "click", function( msg ) {
	    //需要注意的是，安卓端msg为对象，但是msg.payload为json字符串，使用时需要JSON.parse(msg)
	    console.log(JSON.parse(msg.payload));  //{aaa:"自定义参数a",bbb:'自定义参数b',ccc:"自定义参数c"}
	}, false );
});
```

<a name="ANDROID前台非标准格式通知接收" />
#### ANDROID前台非标准格式通知接收（响应receive事件）

##### 个推端配置
        
* 通知标题 : xxx(标记此次通知，无实际意义仅作为备注)
    
* 消息内容 : `{aaa:"aaa",bbb:"bbb",payload:{ccc:"ccc",ddd:"ddd",eee:"eee"}}`
    
##### 手机端接收
    
```javascript
mui.plusReady(function() {
	plus.push.addEventListener( "receive", function( msg ) {
        //alert(JSON.stringify(msg));
        //msg依然为对象
    }, false );
});
```
> 消息通过`msg.content`可以被获取

当参数里面有payload时

```
消息内容 {aaa:"aaa",bbb:"bbb",payload:{ccc:"ccc",ddd:"ddd",eee:"eee"}}

响应回的 msg 参数为 {"content":{"aaa":"aaa","bbb":"bbb","payload":{"ccc":"ccc",ddd:"ddd",eee:"eee"}},"payload":{"ccc":"ccc","ddd":"ddd","eee":"eee"},"appid":"xxxxxxxx","title":"xxx","_UUID":xxxxxx}

```
当参数为任意值（不带payload）
    
```
消息内容 {aaa:"aaa",bbb:"bbb",ccc:{ddd:"ddd",eee:"eee",fff:"fff"}}

页面响应 {"content":{"aaa":"aaa","bbb":"bbb","ccc":{"ddd":"ddd","eee":"eee","fff":"fff"}},"payload":{"aaa":"aaa","bbb":"bbb","ccc":{"ddd":"ddd","eee":"eee","fff":"fff"},"appid":"xxxxxxxx","title":"xxx","_UUID":xxxxxx}

```
可以看出，即使在格式不完全正确的情况下，payload参数依然会影响相应结果，即当配置消息内容中有payload参数，响应回的`msg.payload`与配置消息内容中的payload一致，当配置消息内容中没有payload参数那么相应回的`msg.content = msg.payload`且都与配置消息内容相同 


#### ANDROID后台标准格式通知接收（响应click事件）

同 [ANDROID前台标准格式通知接收](#ANDROID前台标准格式通知接收)


#### ANDROID后台非标准格式通知接收（响应receive事件）

同 [ANDROID前台非标准格式通知接收](#ANDROID前台非标准格式通知接收)

### IOS

> 以下测试均在基础通知模式下

IOS对于通知栏的配置实在下列几个参数里（以下个推IOS配置仅在离线通知栏有作用）

* alert
    * title : 通知栏显示标题
    * body : 通知栏显示内容区域
* badge : 应用右上角数字
* sound : 通知声音
* payload : 可选带自定义参数（如果自定义那么将发送自定义参数，否则和消息内容一致）
* category : 自定义通知按钮事件

#### IOS前台标准格式通知接收（响应receive事件）

##### 个推端配置

* 通知标题 : xxx(标记此次通知，无意义类似于备注)
    
* 消息内容 : `{"title":"测试标题","content":"测试content","payload":{"aaa":"自定义参数a","bbb":"自定义参数b","ccc":"自定义参数c"}}`

##### 手机端接收

```javascript
mui.plusReady(function() {
	plus.push.addEventListener( "receive", function( msg ) {
        console.log(msg);  //{"title":"测试标题","content":"测试content","payload":{"aaa":"自定义参数a","bbb":"自定义参数b","ccc":"自定义参数c"},"type":"receive","aps":null}
        console.log(msg.payload);  //{"aaa":"自定义参数a","bbb":"自定义参数b","ccc":"自定义参数c"}
    }, false );
});
```

#### IOS前台非标准格式通知接收（响应receive事件）

##### 个推端配置

* 通知标题 : xxx(标记此次通知，无意义类似于备注)
    
* 消息内容 : `{aaa:"aaa",bbb:"bbb",ccc:{ddd:"ddd",eee:"eee",fff:"fff"}}`

##### 手机端接收

```javascript
mui.plusReady(function() {
	plus.push.addEventListener( "receive", function( msg ) {
        console.log(msg);  //{"payload":/*这里为json字符串，内容与消息内容一致*/,"title":aa,"content":/*这里为json字符串，内容与消息内容一致*/,"type":"receive","aps":null}
    }, false );
});
```

#### IOS后台标准通知接收（响应click事件）

##### 个推端配置

* 通知标题 : xxx(标记此次通知，无实际意义仅作为备注)
    
* 消息内容 : `{"title":"测试标题","content":"测试content","payload":{"aaa":"自定义参数a","bbb":'自定义参数b',"ccc":"自定义参数c"}}`

##### 手机端接收

```javascript
var message = null;
mui.plusReady(function() {
	message = document.getElementById("message");
	plus.push.addEventListener( "click", function( msg ) {
		console.log(msg);  //{"payload":{"title":"测试标题","content":"测试content","payload":{"aaa":"自定义参数a","bbb":'自定义参数b',"ccc":"自定义参数c"}},"title":"宝分期","content":"测试content","type":"click","aps":{"alert":{title:"/*alert.title*/","body":/*alert.body*/},"sound":"default","mutable-content":1}}
	    console.log(JSON.parse(msg.payload.payload));  //这里注意 msg.payload为对象，但是msg.payload.payload是字符串
	}, false );
});
```

#### IOS后台非标准通知接收（响应click事件）

##### 个推端配置

* 通知标题 : xxx(标记此次通知，无意义类似于备注)
    
* 消息内容 : `{msgTitle:"测试标题",msgContent:"测试content",payload:{"aaa":"自定义参数a","bbb":'自定义参数b',"ccc":"自定义参数c"}}`

##### 手机端接收

```javascript
var message = null;
mui.plusReady(function() {
	message = document.getElementById("message");
	plus.push.addEventListener( "click", function( msg ) {
		console.log(msg);  //{"payload":{"_gurl_":sdk.open........,"payload":"{/*这里为json字符串，内容与消息内容一致*/}","_gmid_":.......,"_ge_":"1"},"title":"宝分期","content":"alert.body","type":"click","aps":{"alert":{title:"/*alert.title*/","body":/*alert.body*/},"sound":"default","mutable-content":1}}
	}, false );
});
```

需要注意，ios返回的数据皆为json对象click事件中需要使用msg.payload.payload  receive事件中需要使用msg.payload  注:不含格式不规范的情况


## 结论

当推送格式严格按照`{title:"通知标题",content:"通知内容",payload:"通知去干嘛这里可以自定义"}`格式的情况下，我们可以通过以下代码获取自定义信息
```javascript
var message = null;
mui.plusReady(function() {
    message = document.getElementById("message");
    //监听点击消息事件
    plus.push.addEventListener( "click", function( msg ) {
        var payload = null;
        switch(plus.os.name){
            case "Android":
	            try{
	                payload = JSON.parse(msg.payload);
	            }catch(e){
	                //console.log('格式错误');
	            }
	            responseMsg(payload);
	            break;
            case "iOS":
                responseMsg(msg.payload.payload);
                break;
            default:
            	break;
        }
    }, false );
    //监听在线消息事件
    plus.push.addEventListener( "receive", function( msg ) {
        var payload = null;
        switch(plus.os.name){
            case "Android":
	            break;
            case "iOS":
                responseMsg(msg.payload);
                break;
            default:
                break;
        }
    }, false );
    function responseMsg(payload){
        //自定义业务逻辑，payload为自定义消息的对象
    }
});
```

