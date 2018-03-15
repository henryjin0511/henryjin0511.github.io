---

title: chrome控制台中输出object成员信息错误分析

date: 2018-03-15 11:23:12

categories: 
- chrome

tags: 
- js
- object

---

偶然间在用chrome调试时发现一个小小的问题，代码如下

```javascript
let a = {key1: 'val1', key2: 'val2'}
console.log(a)
a.key3 = 'val3'
console.log(a)
```

问题当然不会是console输出什么，奇怪的点在于下边这个三角符号

![控制台中三角符号未打开状态](/images/1803/chrome/key.png)

看起来似乎没有什么问题，那么打开呢？

![控制台中三角符号打开状态](/images/1803/chrome/key-open.png)

<!-- more -->

很奇怪为什么打开三角符号之后，显示的对象值均为修改过后的，毫无疑问这种表现是会影响到我们的判断，后经过一番查询，大致确认，打开三角符号来查询对象属性是一种惰性的行为，当你点击三角符号的那一刻，chrome才会在内存中寻找对象的真实信息，所以当我们打开三角符号的时候，对象其实已经改变，在[stackoverflow的一个回答中](https://stackoverflow.com/questions/4057440/is-chromes-javascript-console-lazy-about-evaluating-arrays)也有类似的解释

能够合理去除这种疑惑的方式之一就是将引用类型转变为简单的值，抛砖引玉，JSON.stringify()即可。

```javascript
let a = {key1: 'val1', key2: 'val2'}
console.log(JSON.stringify(a))
a.key3 = 'val3'
console.log(JSON.stringify(a))
```

再次打印，是不是消除了我们的疑惑~

![修改后控制台输出](/images/1803/chrome/key-open2.png)
	
	

