---

title: nodejs+express上传图片实践

date: 2017-05-03 10:05:25

categories: 
- nodejs

tags:
- express
- nodejs
- 上传图片

---

> 近期在总结项目时，偶然翻起之前做过的一个资料进件的上传模块，前台采用vue构建，后台则是php接收，由于项目类似单页应用，大部分数据均在本地完成操作，最终提交时将整个json数据上传即可，所以萌生兴趣想要利用vue全家桶和express框架进行一次整体实现，以下则是express框架上传图片并且添加验证的部分。

## 使用工具

* Express: 基于 Node.js 平台，快速、开放、极简的 web 开发框架。
* Postman: 网页调试与发送网页HTTP请求的Chrome插件，主要用于测试

<!-- more -->

## express框架安装

[安装链接](http://www.expressjs.com.cn/starter/installing.html)

## 安装文件处理插件

```
npm install multer --save
```

express原生不支持`multipart/form-data`数据的格式化，multer是一款第三方express中间件，可以帮助我们更加快速的处理file文件的上传，用法也并不复杂，这里取官网上常用的三种情况做简单说明，另附[npm包介绍链接](https://www.npmjs.com/package/multer)，[官网中文说明](https://github.com/expressjs/multer/blob/master/doc/README-zh-cn.md)


```javascript
var express = require('express')
var multer  = require('multer')
var upload = multer({ dest: 'uploads/' })	//缓存文件夹
 
var app = express()
 
app.post('/profile', upload.single('avatar'), function (req, res, next) {
  // req.file 是key为avatar的文件信息
  // req.body 是其他非文件字段的对象集合，没有则为{}
})
 
app.post('/photos/upload', upload.array('photos', 12), function (req, res, next) {
  // req.files 是key为photos的文件数组的信息
  // req.body 是其他非文件字段的对象集合，没有则为{}
})
 
var cpUpload = upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'gallery', maxCount: 8 }])
app.post('/cool-profile', cpUpload, function (req, res, next) {
  // req.files 是一个对象,键是文件名, 值是文件数组
  //
  // 例如：
  //  req.files['avatar'][0] -> File
  //  req.files['gallery'] -> Array
  //
  // req.body 是其他非文件字段的对象集合，没有则为{}
})
```

## 安装图片宽高处理插件

```
npm install image-size --save
```

由于项目中后台只负责验证图片宽高是否超过限制，并不对图片做处理，所以选择了[image-size](https://www.npmjs.com/package/image-size)，支持常规BMP，GIF，JPEG，PNG，PSD，TIFF，WebP，SVG文件宽高的获取，使用方法较为简单，以下为同步写法，异步写法可直接在官网查询。

```javascript
var sizeOf = require('image-size');
var dimensions = sizeOf('images/funny-cats.png');	//图片路径
console.log(dimensions.width, dimensions.height);
```

注：image-size并不需要结合express使用，直接结合原生node.js也可直接使用

## 项目实战

以下为express的项目目录

```
.
├── app.js	-change/修改
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   |── users.js
│   └── upload.js	-new/新增
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade

```

### 修改./app.js

```javascript
//...
app.use('/', index);
app.use('/users', users);
app.use('/upload',upload);	//新增upload

// catch 404 and forward to error handler
//...
```

定义upload中间件,捕获`'/upload'`路由上的事件

### 新增 ./routes/upload.js

```javascript
const path = require('path'),  //获取path模块
	fs = require('fs'),  //获取文件管理模块
	express = require('express'),  //获取express模块
	router = express.Router(),  //获取express路由模块
	multer = require('multer'),  //获取multer插件模块
	upload = multer({dest: path.resolve(__dirname,'../public/cache/')}),  //定义multer处理程序且定义缓存路径
	sizeOf = require('image-size'),  //定义获取图片宽高模块
	fileSavePath = path.resolve(__dirname,'../public/upload/');  //定义文件储存路径

router.post('/',(req,res,next)=>{
	upload.single('imgFile')(req,res,function (err) {
		if(err){
			if(err.code === 'LIMIT_UNEXPECTED_FILE'){	//文件key值有误或者当调用upload.none()且上传文件时会有此错误
				res.json({returnCode:1006,msg:'文件有误'});return false;
			}else{
				res.json({returnCode:1007,msg:'内部错误'});return false;
			}
		}
		console.log(req.body);
		console.log(req.file);
		let file = req.file;
		let tmp_path = req.file.path;
		let target_path = fileSavePath+'/'+req.file.originalname;  //实际项目中，文件名应重新定义防止重复
		if(file.size > 2*1024*1024){
			res.json({returnCode:1004,msg:'超出大小2M'});return false;
		}else if(['image/jpeg','image/jpg','image/png','image/gif'].indexOf(file.mimetype) === -1){
			res.json({returnCode:1003,msg:'文件类型必须为jpg,png,gif,jpeg'});return false;
		}else{
			let imageSize = sizeOf(tmp_path);
			if(imageSize.width > 1024 || imageSize.height >1024){
				res.json({returnCode:1005,msg:'宽高1024限制'});return false;
			}
		}
		(function func(){
			fs.rename(tmp_path,target_path,function (err) {
				if (err) {
					if (err.code === "ENOENT") {  //未创建文件夹的错误处理
						fs.mkdir(fileSavePath,(err)=>{
							if(err)throw err;
							func();
						});
					} else {
						fs.unlink(tmp_path,function (err) {
							if(err)throw err;
						});
						throw err;
					}
				}else {
					res.json({returnCode:1001,msg:'上传成功'});return false;
				}
			});
		})();
	});
});
module.exports = router;
```

代码总结
* 代码的大部分都在处理错误信息，稍显冗长
* 下面的自执行函数主要是为了处理第一次上传未创建文件夹的问题，如果服务端一开始创建好文件夹，下面代码可省略大部分
* 代码中大量引入`res.json();`是假想上传情景为异步，如果为同步上传，可自行调用其他方法处理。

### 运行

```
npm start
```

此处建议使用chrome插件postman进行测试，快速且方便，默认端口为3000，测试时可以直接向url为`http://127.0.0.1:3000/upload`post数据即可，注意前端`Content-Type`为`multipart/form-data`


## 总结

express完美的继承了javascript语言的强扩展性，自身精简，利用各种中间件的扩展可以快速实现某个功能的扩展。此处对于文件的处理便是如此，虽然express并不提供req,files的支持，但是加入第三方中间件以后，便补足了在文件上传方面的功能。




