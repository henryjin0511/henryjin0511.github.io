---

title: IE8异步文件上传

date: 2017-04-14 23:23:12

categories: 
- javascript

tags: 
- 异步上传文件
- iframe

---

## 为什么要讲IE8异步文件上传

现阶段浏览器环境下异步上传的方法一般有三种

1. flash 这种方式产生较早，阿里，百度均有相应插件可利用，优点是使用较为简单，兼容性强，只需要看几遍API就可上手，缺点则是可定制性不强，部分插件样式固定无法调整，且需要引入外部flash文件资源，整体较为笨重。（国内阿里和百度出品都已做到根据浏览器适配上传方法，在资源使用方面占有优势）。
2. iframe + form 模拟异步上传，简单来说，利用form的target属性在浏览器中提交form到iframe中，获取到iframe的body内容即是服务器响应内容，然后根据需求进一步处理即可。
3. ajax 利用formdata对象来构建form表单直接通过XMLHttpRequest发送即可，当然，现阶段这种方法的兼容性可谓是十分之差，移动端尚可接受，PC基本可以放弃，具体兼容性问题可参考[caniuse](http://caniuse.com/#search=formdata)

鉴于上述情况，iframe + form 显然是当下轻量应用较为明智的选择，下面就和大家分享个人对此种方法的实践经历。

<!-- more -->

## 简单实践

### html部分

```html
<iframe id="upload_iframe" name="upload_iframe" style="display:none;"></iframe>
<form method="post" enctype="multipart/form-data" action="<!--你想要上传的地址-->" target="upload_iframe">
    <input type="file" name="file">
</form>
```

```javascript
var iframe = document.getElementById('upload_iframe');
iframe.onload = function(){
	var doc = this.contentDocument || this.contentWindow.document;
	var data = doc.body.innerHTML
	console.log(data);  //data就是服务器返回的responseText
}
```

上述代码就是iframe + form 组合异步上传文件的简单实现，实际运用中还需要注意以下几点。
1. 建议一开始给iframe加属性 data-first="true" 防止页面初始化时触发iframe的onload函数，后续需要在form提交后将first属性置为false并且在iframe的onload事件中做出判断。
2. 由于上传时间无法确定，请在form表单的submit事件触发后增加loading效果，在iframe触发load事件后关闭即可。

## 封装重用

实际开发中，上述代码已经能够满足大部分情况下的上传，但是在几个页面写完之后，不断地ctrl+c ctrl+v是否会让你感到厌倦呢，基于开发效率和代码复用性的考虑，我们应该对其进行优化。下面就是封装过程

### 需求分析

1. 上传，支持单页面或多页面，单上传组件和多上传组件的混用。
2. 支持回调，包括以下事件触发点
* 上传之前
* 上传之后
* 上传成功
* 上传失败

首先，作为上传组件，上传一定是我们的核心需求，所以页面中的iframe和form元素一定是必须存在的，考虑到尽可能封装彻底，所以这两个元素由js写入html实现。实际应用中，一个页面有可能有一个或多个上传按钮，而且每个按钮对应的接口和文件字段名都不一定一致，这个时候我们应尽可能重用页面元素，所以页面中的form元素的action属性和input的name值我们都应该动态写入

### 实现分析

1. 对于支持单页面或多页面，单上传组件和多上传组件的混用，最简单的情况则是多实例，每一个实例处理一个上传按钮，但是出于对代码的封装，我们决定复用页面中的iframe和form元素，将form元素的action属性和input的name值动态写入。
2. 回调方面，在上传流程中间加入相应回调函数即可

### 具体实现

具体实现如下。（以下代码均依赖于jQuery，主要是针对一些dom操作的简化，原生实现可根据代码进行轻度改造。）

```javascript
//上传组件
(function () {
	function UploadPlugin() {
		this.opts = {
			fileName:'imgFile',  //上传文件name值
			url:'',  //接口链接
			parseJSON:true,  //是否以json格式解析返回值，默认为true
			beforeUpload:null,  //上传之前的事件处理
			complete:null,  //完成上传
			success:null,  //上传成功（一般认为iframe触发load之后符合要求格式即为成功上传）
			error:null  //上传失败，如果iframe中body中的值不符合格式即为失败，parseJSON参数为false时不会触发此函数）
		};
		this.init();
	}
	UploadPlugin.prototype.init = function () {
		var _this = this;
		/*构建dom*/
		var $form = $('#uploadFileForm');
		if($form.length >= 1){
			_this.$form = $form.eq(0);
			_this.$input = this.$form.children('input').eq(0);
		}else {
			_this.$form = $('<form id="uploadFileForm" enctype="multipart/form-data" method="post" target="uploadFileIframe" style="display: none;"><input type="file" /></form>');
			_this.$input = this.$form.children('input');
			$('body').append(this.$form);
		}
		var $iframe = $('#uploadFileIframe');
		if($iframe.length >= 1){
			_this.$iframe = $iframe.eq(0)
		}else {
			_this.$iframe = $('<iframe data-first="true" id="uploadFileIframe" name="uploadFileIframe" style="display: none;"></iframe>');
			$('body').append(this.$iframe);
		}
		/*构建dom结束*/
		/*监听事件*/
		_this.$iframe.on('load',function () {
			if(_this.$iframe.data('first'))return false;
			var _$this = $(this),
				doc = _$this[0].contentDocument || _$this[0].contentWindow.document,
				data = doc.body.innerHTML;
			_this.selfConfig.complete.call(_this,data);
			if(_this.selfConfig.parseJSON !== true){
				_this.$input.val('');
				_this.selfConfig.success.call(_this,data);
				return false;
			}
			try {
				data = JSON.parse(data);
			}catch(e){
				_this.$input.val('');
				_this.selfConfig.error.call(_this,data);
				return false;
			}
			_this.selfConfig.success.call(_this,data);
			_this.$input.val('');
		});
		_this.$input.on('change',function () {
			if(!this.value)return false;
			_this.$iframe.data('first',false);
			_this.selfConfig.beforeUpload.call(_this);
			this.form.submit();
		});
		/*监听事件结束*/
	};
	UploadPlugin.prototype.upload = function (config) {
		var _this = this;
		_this.selfConfig = $.extend({},_this.opts,config);
		_this.$input.attr('name',_this.selfConfig.fileName);
		_this.$form.attr('action',_this.selfConfig.url);
		_this.$input.trigger('click');
	};
	window.uploadPlugin = (function (options) {
		return new UploadPlugin(options);
	})();
})();
```

### 基本介绍

首先`UploadPlugin`对象为原生对象，使用时直接调用`uploadPlugin.upload(config)`即可（默认引入script标签时即创建了上传插件对象）。`config`为上传配置，未做全局化处理是考虑多页面之间多次调用多个接口的情况。

`config`配置
* `fileName` 文件名
* `url`  上传路径
* `parseJSON`  是否json格式化
* `beforeUpload`  上传前回调
* `complete`  上传后回调（在success和error之前）
* `success`  上传成功回调（成功的标准在此处为json格式）
* `error`  （上传失败回调）

### 使用案例

默认已引入jQuery

```html
<a class="uploadBtn" href="javascript:;" data-action="<!--上传路径1-->">点击上传</a>
<a class="uploadBtn" href="javascript:;" data-action="<!--上传路径2-->">点击上传</a>
<a class="uploadBtn" href="javascript:;" data-action="<!--上传路径3-->">点击上传</a>

<script>
$(function(){
	$('.uploadBtn').on('click', function(){
		var action = $(this).data('action');
		uploadPlugin.upload({
			fileName:'file',
			url:action,  //上传路径配置
			parseJSON:true,  //以JSON方式解析，可省略
			beforeUpload:function () {
				//这里可以写loading逻辑
			},
			complete:function () {
				//这里可以取消loading逻辑
			},
			success:function (data) {
				//这里可以处理相应数据，如果开启parseJSON，那么默认为JSON.parse()后的数据
			},
			error:function (data) {
				//这里为JSON.parse失败后处理返回数据的接口
			}
		});
	});
});
</script>
```

可根据不同的业务情景创建单独的配置项





	

