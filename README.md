title: 微信小程序
speaker: 吕士杰
url: https://github.com/ksky521/nodeppt
transition: slide3
files: /js/demo.js,/css/demo.css,/js/zoom.js
theme: moon
usemathjax: yes

[slide]
# 微信小程序原生开发
<img src="/assets/img/logo.jpg" style="display: block;margin: 33px auto;"width="120" height="120" />
[slide]
## 小程序解构
-----    
<img src="/assets/img/60302.jpeg" style="position: relative;right: -55%;max-height: 669px!important;" width="375"  />

<ul class="rollIn" style="position: absolute;top: 30%;left: 42%;">
    <li class="builded" data-index="0">窗口</li>
    <li class="builded" data-index="1">页面</li>
    <li class="builded" data-index="2">底部 tab 栏</li>
    <li class="builded" data-index="3">下拉背景</li>
</ul>
    
[slide]

## 自定义组件内部选项
-----
```javascript
/* 由外部传递进来的值 */
properties: {
    like: {
        type: Boolean
    },
    count: Number
},

/* 组件自身的状态 */
data: {
    isLike: 'images/like.png',
    notLike: 'images/like@dis.png',
},

created() {
    console.log(this.properties === this.data) // true
}
```
 
> 在组建的内部声明 properties 和 data 后，小程序会在调用 created 钩子前，会合并 properties 和 data 里的所有的属性，并挂载到组件实例的 data 和 properties 属性，所以在 created 钩子里打印 this.properties === this.data 结果是true。所以要避免在 properties 和 data 里声明相同变量名。

[slide]

## 自定义组件的试图更新
-----
```javascript
this.setData({
   isLike: true,
   countNum: 20
})
```
> 使用 this.setData更新试图，小程序会把 isLike 和 countNum 属性分别挂载到 this.properties 和 this.data 下



[slide]
## 自定义组件外部传值的获取时机
-----

```javascript
properties: {
    like: {
        type: Boolean,
        observer(newValue, oldValue) { // 改变值后会自定调用此函数，同步数据不会执行 observer，只有异步才会执行
            this.setData({
                like: newValue
            })
        }
    }
},
```
> 如果想变异监听组建外部传递的属性，应使用 ovserver 回调



[slide]

## 复用自定义组件的选项
-----

```javascript
// 如果多个组件拥有相同的选项，可以把公共部分提取出来：
Component({
    properties: {},
    data: {},
    created() {},
    methodes: {},
})
```

```javascript
const classicBeh = Behavior({
    properties: {
        img: {
        type: String,
    },
    created() {}
}) 

export { classicBeh }
```

```javascript
//注册复用项
import {classicBeh} from '../classic-beh.js'
import {aBeh} from '../a-beh.js'
Component({
  behaviors: [ classicBeh, aBeh ], // 继承组件的公共属性
  data: { }
})
```
> behaviors 是个数组，说明可以继承多个父类，相同属性子类覆盖父类，最后一个父类覆盖前面的父类属性。唯独 attached 钩子，会依次执行父类的attached，最后执行子类的attached

[slide]

## 使用自定义组件的 slot
-----

```javascript
// 开启 slot：
Component({
    options: {
        multipleSlots: true
    }
})
```

```javascript
// 组建内使用具名 slot 占位：
<view class="container">
    <slot name="after" /> 
<view>
```

```javascript
// wxml内使用slot：
<v-tag>
    <text slot="after">200</text>
</v-tag>
```

[slide]

## 自定义组件的外部样式
-----

```javascript
// 这是 v-tag 组件
Component({
  externalClasses: ['tag-class'], // 在组建内注册外部样式名
})
```

```html
<!-- 在 v-tag 组件的 wxml 元素上绑定外部样式： -->
<view bind:tap="onTap" class='container tag-class'>
  <text>{{text}}</text>
</view>
```

```html
<!-- 为了改变 v-tag 组件默认的样式，在使用改组件的 wxml 元素上添加，并在 wxss 内添加样式 -->
<v-tag tag-class="ex-tag-class"></v-tag>
``` 
> 有赞（Vant Weapp）组件，会提供外部样式类接口，改变组件的默认样式:
<img src="/assets/img/20181217114005.png" style="max-height: 669px!important;" width="675"  />

[slide]

## hack 方式更改自定义组件样式
-----
* 有时候使用外部样式类的方式任然不足以更改第三方组件的样式，这时可以使用 hack 方式迳入组件内部样式
<div style="height: 20px;"></div>
```html
<!-- 在需要覆盖样式的组件上添加样式： -->
<v-tag bind:tap="onTap" class='hack-class' />
```

```html
<!-- 写下 hack-class 的样式，v-tag:nth-child(1) 就是组件内部的样式 -->
.hack-class v-tag:nth-child(1) {
    ...
}
``` 
> 组件内部的样式不可使用样式选择器获取，需要使用标签类型作为选择器

[slide]

## 过滤器的使用
-----
```javascript
// 编写过滤器
var cutLine = function(array, length) {
    return array.slice(0, length)
}
module.exports = {
    cutLine: cutLine,
}
```

```html
<!-- 然后在 .wxml 里导入和使用： -->

<wxs src="../filter.wxs" module="filter" />
<block wx:for="{{ filter.cutLine(comments, 5)) }}" wx:key="static">
    <v-tag text="{{item.content}}" />
</block>
``` 

[slide]

# 系统常用 API

[slide]

## 网络请求
-----
```javascript
wx.request({
    url: '/api/position/add', // api url
    method: 'post', // get、post等
    data: { id: 1 }, // 入参
    success: res => { // 服务器返回数据的回调，400、404、500等也会进入成功回调
    },
    fail: err => { // 服务器未返回数据的回调，比如断网之类的问题
    }
}
```

> 以下状态码都会进入 success 回调:
<img src="/assets/img/61115.jpeg" style="max-height: 669px!important;" width="675"  />

[slide]
# 工具使用
 

[slide]
## 添加编译模式
------
<img src="/assets/img/61189.jpeg" style="max-height: 669px!important;" width="675"  />
<img src="/assets/img/61194.jpeg" style="max-height: 669px!important;" width="675"  />


[slide]

# 有赞（Vant Weapp）组件库
------
<img src="/assets/img/20181217132712.png"  width="160" height="160" />

[slide]
## 把 Vant Weapp 添加到项目中
----
* npm init {:&.rollIn}
* npm i vant-weapp -S --production
* 点击开发者工具中的菜单栏：工具 --> 构建 npm，编译成功后项目目录里会生成有赞组件文件<br/>
  <img src="/assets/img/1543977130(1).png"  width="200" height="200" />
* 在 app.json 中全局注册组件，或者在某个页面里局部注册组件：
```javascript
{
    "usingComponents": {
        "van-button": "../../miniprogram_npm/vant-weapp/button/index"
    }
}
```
[slide]
# mpvue
-----
<img src="/assets/img/logo.png"  width="160" height="160" />

[slide]
## 初始化项目模板
----

* npm install --global vue-cli
* vue init mpvue/mpvue-quickstart my-project
* npm install
* npm run dev <br/>
<img src="/assets/img/20181217135027.png"  class="img" width="200" height="760" />
 
<div class="struct">项目结构对照</div> {:&.rollIn}

mpvue | 原生小程序
:-------|:------:
App.vue | app.js、app.wxss
app.json | app.json
pages/ | pages/
components/ | components/
 
<style>
.struct {
    padding: 30px 0;
}
.img {
    position: absolute;
    right: -51%;
    top: -23%;
}
</style>

[slide]
## 引入 Vant Weapp
---
* npm i vant-weapp -S --production  {:&.rollIn}
* 把生成的组建包从 node_modules 中复制到项目根目录下的 static 文件夹下
* 修改 webpack.base.conf.js：
```javascript
module: {
    rules: [
        {
            test: /\.js$/,
            include: [resolve('src'), resolve('test'), resolve('static/vant-weapp')],
            use: [
                'babel-loader',
                {
                    loader: 'mpvue-loader',
                    options: Object.assign({checkMPEntry: true}, vueLoaderConfig)
                },
            ]
        },
    ]
}
```
* 在 app.json 中按需注册
```javascript
"usingComponents": {
    "van-button": "/static/vant-weapp/button/index",
    "van-search": "/static/vant-weapp/search/index",
    "van-badge": "/static/vant-weapp/badge/index",
    "van-badge-group": "/static/vant-weapp/badge-group/index"
}
``` 
[slide]

# 谢谢收看


