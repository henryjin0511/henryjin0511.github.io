---

title: 基于CodeIgniter 2的前端模板生成

date: 2017-04-13 10:05:25

categories: 
- php

tags:
- php
- CodeIgniter
- 前端模板

---

现代化网站项目构建，无非两种渲染方式，后端渲染与前台渲染，两种方式各有优劣，在此不一一列出，本文针对的则是后端渲染的方式，在长期实践中发现很多网站的后端渲染方式都存在的或多或少的问题点，所以便将项目中实践的模板方案分享出来。

首先贴出CodeIgniter-2.2.6的初始文件组织结构

```
CodeIgniter-2.2.6
   |——application
   |    ...
   |    |——helpers
   |    |——views
   |    ...
   |——system
   |——user_guide
   |——.gitignore
   |——.travis.yml
   |——index.php
   |——license.txt
   |——README.md
```

<!-- more -->

## 旧方案

### 文件结构

```
CodeIgniter-2.2.6
   |——application
   |    ...
   |    |——helpers
   |    |——views
   |        |——index.php
   |        |——header.php
   |        |——footer.php
   |        ...
   |    ...
   |——system
   |——user_guide
   |——.gitignore
   |——.travis.yml
   |——index.php  入口文件
   |——license.txt
   |——README.md
```

### 代码组织

`application/views/index.php`

```php
<?php $this->load->view('header');?>
<div>
//页面代码
</div>
<?php $this->load->view('footer');?>
<script src="jquery.min.js"></script>
<script>
//js逻辑
</script>
```

`application/views/header.php`

```php
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8" />
	<title></title>
</head>
<body>
	<header>
	</header>
```

`application/views/footer.php`

```php
	<footer>
	</footer>
</body>
</html>
```
 
### 最终渲染生成页面结构（假设有第三方组件a包括a.css和a.js）

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8" />
	<title></title>
</head>
<body>
	<header>
	</header>
	<link rel="stylesheet" href="a.css" />
	<footer>
	</footer>
</body>
</html>
<script src="a.js"></script>
<script>
//...
</script>
```

### 问题点

1. 这种方案生成的前端代码组织并不符合前端规范，每个页面内JS代码都需要书写在`</html>`标签之外，且每个页面如果引入JS插件时，其中的CSS文件部分会在`<body>`内加载，JS文件则在`<html>`外部加载，均不利于前端的性能优化。
1. 一旦网站有多种布局形式，就会产生不同的头文件和底文件。
2. 当你用大型编辑器或者自动化html压缩工具时，`header.php`和`footer.php`中的`<html>`和`<body>`的非闭合标签会导致一系列报错或是自动添加标签的错误出现。

## 新方案

基于上述情况，在结合了CI框架的类库扩展，做出了前端的模板系统，此处模拟了项目中的商城结构，所以会有`*_mall`相关字样

### 文件结构

```
CodeIgniter-2.2.6
   |——application
   |    ...
   |    |——config
   |        |——fe_helper.php
   |    |——views
   |        |——template
   |            |——template_mall.php
   |            |——head_mall.php
   |            |——script_mall.php
   |        |——mall
   |            |——index.php
   |        ...
   |    ...
   |——system
   |——user_guide
   |——.gitignore
   |——.travis.yml
   |——index.php  入口文件
   |——license.txt
   |——README.md
```

注意：增加了`application/helpers/fe_config.php`文件，用来配置前端模板类，注意在`application/config/autoload.php`里的`$autoload['helper'] = array();`里加入我们的`fe`，即`$autoload['helper'] = array('fe');`

### 代码组织

* 配置 `application/helpers/fe_helper.php`

```php
<?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');
/*
	* 前端SEO CSS JS类
*/
class front_end
{
	private $feConfig;
	function __construct(){
		$this->CI = & get_instance();
		$this->feConfig = array(
			//全站主页
			'index/homepage'=>array(
				'seo'=>array(
					'title'=>'测试title',
					'description'=>'测试description',
					'keywords'=>'测试keywords'
				),
                'css'=>array(
                    'https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min'    //单纯测试用
                ),
                'js'=>array(
                    'https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min'    //单纯测试用
                )
			)
		);
	}
	public function get_seo($page_name = ''){
        $feConfig = $this->feConfig;
        if(!$page_name) return "";
        $seo = (isset($feConfig[$page_name]['seo']) && !empty($feConfig[$page_name]['seo']))?$feConfig[$page_name]['seo']:array();
        $seo_title = (isset($seo['title']) && !empty($seo['title']))?$seo['title']:'默认标题';
        $seo_description = (isset($seo['description']) && !empty($seo['description']))?$seo['description']:'默认标题';
        $seo_keywords = (isset($seo['keywords']) && !empty($seo['keywords']))?$seo['keywords']:'默认标题';
        $seo_string = '<title>'.$seo_title.'</title><meta name="description" content="'.$seo_description.'" /><meta name="keywords" content="'.$seo_keywords.'" />';
        return $seo_string;
    }
	public function get_css($page_name = ''){
		$feConfig = $this->feConfig;
		if(!$page_name) return "";
		$css_string = "";
		if(isset($feConfig[$page_name]['css']) && !empty($feConfig[$page_name]['css'])){
			foreach ($feConfig[$page_name]['css'] as $k => $v){
				$css_string .= (($k==0)?"":"\r\n").'<link rel="stylesheet" href="'.$v.'.css" />';
			}
		}
		return $css_string;
	}
	public function get_js($page_name = ''){
		$feConfig = $this->feConfig;
		if(!$page_name) return "";
		$js_string = "";
		if(isset($feConfig[$page_name]['js']) && !empty($feConfig[$page_name]['js'])){
		    foreach ($feConfig[$page_name]['js'] as $k => $v){
				$js_string .= (($k==0)?"":"\r\n").'<script type="text/javascript" src="'.$v.'.js"></script>';
			}
		}
		return $js_string;
	}
	
}
/* End of file fe_helper.php */
/* Location: ./application/helper/fe_helper.php */
?>
```

* 案例中将前端配置直接写在了类的私有变量中，实际中可以单独整合一个json文件来储存页面相关信息
* 案例中有get_seo();get_css();get_js();三个函数，实际使用中可以根据需求继续扩展

* 控制器 `application/controllers/mall/index.php`

```php
...
public function index()
{
	$data['fe_file_path'] = 'mall/index';//路径views内的路径地址
	$this->load->view('template/template_mall',$data);
}
...
```

之前旧方案中没有放controllers中的文件配置，是因为之前的方法无需修改此文件，按照传统方式载入即可，这里我们载入的视图文件不再是直接的目标文件，而是载入了模板文件。

* 整体模板文件`application/views/template_mall.php`

```php
<?php $data['feConfig'] = new front_end();?>
<!DOCTYPE html>
<html>
<?php $this->load->view('template/head_ix',$data);?>
<body>
<?php $this->load->view($fe_file_path);?>
</body>
</html>
```

* head模板 `application/views/template/head_mall.php`

```php
<head>
	<meta charset="utf-8" />
	<?=$feConfig->get_seo($fe_file_path);?>
	<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
	<meta name="renderer" content="webkit">
	<link rel="icon" type="image/png" href="/public/assets_dev/img/favicon.png" />
    <!--[if lt IE 9]>
    <script src="http://res.zuanbank.com/static/lib/js/html5shiv/3.7.2/html5shiv.min.js"></script>
    <![endif]-->
	<?=$feConfig->get_css($fe_file_path);?>
</head>
```

* script模板 `application/views/template/script_mall.php`

```php
<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
<?=$feConfig->get_js($fe_file_path);?>
```

* 页面 `application/views/mall/index.php`

```php
<div>
//页面代码
</div>
<?php $this->load->view('template/script_mall');?>
<script>
	$(function(){
		console.log('测试');
	})
</script>
```

### 最终渲染生成页面结构（假设有第三方组件a包括a.css和a.js）

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8" />
    <title>测试title</title>
    <meta name="description" content="测试description" />
    <meta name="keywords" content="测试keywords" />	
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
    <meta name="renderer" content="webkit">
    <link rel="icon" type="image/png" href="/public/assets_dev/img/favicon.png" />
    <!--[if lt IE 9]>
    <script src="http://res.zuanbank.com/static/lib/js/html5shiv/3.7.2/html5shiv.min.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" />
</head>
<body>
	<header>
	</header>
	<footer>
	</footer>
    <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
    <script type="text/javascript" src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    <script>
    $(function(){
        console.log('测试');
    })
	</script>
</body>
</html>
```

### 简单分析

* 方案中利用CI中自定义的模板类，将html各个部分作为组件载入同一个模板，文件结构清晰，标签嵌套完善，符合规范。
* 并且可以根据项目来定制多个模板，同时兼容非模板使用方法。
* 方案中且SEO信息和静态资源都做了集中管理，通过一个文件来管理全局的SEO和静态资源载入。
* 方案中完善了优化了html结构，即使新加入的css也可以引入到head标签中，js文件也会放在</body>之前。

附上据此方案制作的项目[DEMO](https://github.com/jinming6568/CodeIgniter-Fe-Template) 

需要注意的点有

* `application/helperss/autoload`中`fe_helper.php`
* `application/config/autoload`中`$autoload['helper'] = array('fe');`
* `application/controllers/`中mall文件夹
* `application/views/`中mall文件夹


