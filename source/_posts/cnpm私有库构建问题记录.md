---

title: cnpm私有库构建

date: 2018-04-20 16:00:00

categories: 
- npm

tags: 
- npm

---

> 项目组搭建私有npm源已经有一段时间，今天偶然间发现一些问题，解决后打算将这些经验重新整理做记录

## 为什么要搭建私有npm

大部分时间我们确实不需要这样一个东西，淘宝cnpm源已经足够好用，但是当我们写项目时出现了这样一种情况，你就不得不考虑一下私有npm源了

开发A项目时，写了一个弹窗组件，开发B项目时，复制过来，这个时候设计过来说，要调整，那好吧，改完之后再复制一遍，开发C项目时，再复制，设计又过来...可能一般人到了B开始就会考虑如何维护这种公共组建，npm的模式无疑让我们考虑，如果能有一个类似npm的内部空间就好了，没错，cnpm就能实现你的想法，甚至能做的比你想的更多。

<!-- more -->

## 如何搭建

这里就不班门弄斧了，祭出一篇cnpm开发者点赞的教程[CNPM搭建私有的NPM服务](http://blog.fens.me/nodejs-cnpm-npm/)，当然，配合[官方github项目说明](https://github.com/cnpm/cnpmjs.org)食用味道更佳

## 如何使用

### cnpm配置解析

参考这篇[跟我一起部署和定制 CNPM——基础部署](https://xcoder.in/2016/07/09/lets-cnpm-base-deploy/)

### cnpm命令

1. 私有源只有admin用户才能够publish或unpublish（admin用户的设置见cnpm配置解析）
2. 其他所有命令与npm一致，[npm命令doc](https://docs.npmjs.com/cli/npm)，当然也有中文版，[npm命令doc中文版](https://www.npmjs.com.cn/all/)，有一定英文基础的还是推荐英文版


## 遇到问题怎么办

不得不说，cnpm问题确实不少，笔者在此就遇到了npm5.6版本登录无效的问题，不过好在github中问题解决问题足够积极，所以在此推荐大家，有问题直接点击[issues](https://github.com/cnpm/cnpmjs.org/issues)，不会错~