# Vue源码分析

[TOC]

## Vue架构概览-src

* ./compiler目录是编译模板
* ./core目录是vue的核心（也是后面的重点）
* ./entries目录是生产打包的入口（在之前的版本中存在）
* ./platforms目录是针对核心模块的‘平台’模块
    * platforms目录下有：web目录、weex目录
    * web目录下有对应的./comliler、./runtime、./server、./util目录
* ./server目录是处理服务器渲染
* ./sfc目录处理单文件vue
* ./shared目录提供全局用到的工具函数

## vue在编译的时候能够编译很多个版本，在不同场景下使用

* 位置：`build/config`
* 编译的过程并不是直接输出`*.min.js`，而是`build`出多个模板环境
    * `web-full-prod`生成全部的生产环境
    * `web server renderer-CommonJS`专门给服务器端使用的`CommonJS`环境
    * `weex-factory`专门提供给`weex`环境使用
* 结论:`Vue.js`的组成是由 `core` + 对应的 ‘平台’ 补充代码构成(独立构建和运行时构建 只是 `platforms` 下 `web` 平台的两种选择)。

## core解剖

> vue.js的目标是通过尽可能简单的API实现响应的数据绑定和组合的视图组件

* components：模板编译的代码
* global-api：最上层的文件接口
* instance：生命周期->init.js
* observer：数据收集与订阅
* util：常用工具方法类
* vdom：虚拟dom

## 双向绑定（响应式原理）所涉及到的技术

> Object.defineProperty
> Observer
> Watcher
> Dep
> Directive

![双向数据绑定](http://p1fg8xetu.bkt.clouddn.com/bidirectional_data_binding.jpg)

### 实现数据绑定的做法：

1. 发布者-订阅者模式：`backbone.js`
2. 脏值检查：`angular.js`
3. 数据劫持：`vue.js`

### MVC、MVP、MVVM

**MVC**

* View传送指令到Controller
* Controller完成业务逻辑后，要求Model改变状态
* Model将新的数据发送到View，用户得到反馈

*用户可以通过改变View或者改变Controller来达到目的*

**MVP**

* 各部分之间的通信，都是双向的
* View与Model不发生练习，都是通过Presenter传递
* View非常薄，不部署任何逻辑，称为‘被动视图’，重点是<span style="color:red;">没有主动性</span>

**MVVM**

* 和MVP基本上模式完全一样
* 唯一的区别就是，MVVM采用双向绑定，<span style="color:red;">view具有主动性</span>

## `Object.defineProperty()`

> 分为数据属性和访问器属性

* 作用：修改属性默认的特性，即控制变量的变量
* 这里用到的数访问器属性操作中的，getter和setter函数

```
var book = {
	_year:2004,
	edition:1
};
Object.defineProperty(book,"year",{
	get:function(){
		return this._year;
	},
	set:function(newVal){
		if(newVal>2004){
			this._year = newVal;
			this.edition += newVal -2004;
		}
	}
});

console.log(book.year)
// 这里访问Object.defineProperty定义的属性
book.year = 2005;
console.log(book.edition);
```

### 兼容性

* IE8是第一个实现`Object.defineProperty`方法的浏览器版本
* 然后，这个版本的实现存在诸多限制：只能在DOM对象上使用这个方法，而且只能创建访问器属性

## 设计模式

### 什么是设计模式？

* 一套可以被复用的，编目（标准和规则）分明的经验总结

### 作用

* 让我们写的代码可复用，提高我们的代码的可维护性
* 代码更容易被他人理解、保证代码可靠性

### 构成

> 创建型设计模式：解决对象在创建时产生的一系列问题
> 结构型设计模式：解决类或对象之间组合时产生的一些列问题
> 行为型设计模式：解决类或对象之间的交互以及职责分配的一些问题

* 创建型模式：单例模式、抽象工厂模式、建造者模式、工厂模式、原型模式
* 结构型模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式
* 行为型模式：模版方法模式、命令模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式(Interpreter模式)、状态模式、策略模式、职责链模式(责任链模式)、访问者模式

### 设计模式、框架、架构名词解释

#### 设计模式

* 就是可以被复用，众人知晓，编目分明的经验的总结，侧重于解决某个（些）问题的

#### 框架

* 在某一些软件领域中，将公用的部分提取，抽象出来形成统一的整体，往往是一个半成品，我们需要对他们进行再次加工完成项目的开发，设计了一套思想，引导我们去实现

#### 架构

* 设计的蓝图，没有具体的实现，框架是一种实现了的架构

#### 设计模式和框架的区别

* 设计模式是一个单一的思想，存在就是为了解决某一类问题
* 框架是一套思想的统一，因此可以包含多个设计模式，他们在解决问题的思想上是统一的（一个框架可以包含多个设计模式，是设计模式的结晶）

#### 工具库

* 只是一些方法的封装，每一个方法之间具有独立性
* 框架也是一套方法的封装，框架中的方法彼此之间是有联系的，彼此分工合作实现需求

### 观察者模式

#### 定义

* 定义对象间的一种一对多的依赖关系。
* 当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新

#### 观察者模式流程

* 订阅者也被称为观察者（Observer），订阅者是最常用的一种观察者模式的实现
* 观察者观察的对象，即目标（subject）

#### 观察者和订阅者的具体区别

> 发布 + 订阅 = 观察者模式

* 在观察者模式中，观察者需要直接订阅目标事件；在目标发出内容改变的事件后，直接接受事件并作出响应
* 在发布者模式中，发布者和订阅者之间多了一个发布通道；一方面从发布者接受事件，另一方面想订阅者发布事件；订阅者需要从事件通道订阅事件

```
// 基于jQuery
(function($){
	var o = $({});
	$.subscribe = function(){
		o.on.apply(o, arguments);
	};
	$.unsubscribe = function(){
		o.off.apply(o, arguments);
	};
	$.publish = function(){
		o.trigger.apply(o, arguments);
	};
}(jQuery));
// 订阅
$.subscribe('/some/topic',function(e,a,b,c){
	console.log(a+b+c);
});
// 发布
$.publish('/some/topic',['a','b','c']);
// 退订
$.unsubscribe('/some/topic');
```

## 事件队列：异步macrotask宏队列 和 microtask微队列 简介

> 事件队列，通过eventLoop实现快速处理
> 先执行，microtask：`promise`、`observe`
> 最后执行，macrotask：`ajax`、`click`、`set Timeout`

```
// node运行
setTimeout(function(){
	console.log(5);
},0);
setImmediate(function(){
	console.log(6);
});
new Promise(function(resolve){
	console.log(1);
	resolve();
	console.log(2);
}).then(function(){
	console.log(4);
});
console.log('哈哈哈');
process.nextTick(function(){
	console.log(3);
});
// 依次输出
// 1 2 哈哈哈 3 4 5 6
```

![event loop](http://p1fg8xetu.bkt.clouddn.com/eventLoop.jpg)

### 为什么js是单线程？

* js之所以采用单线程，原因是一开始设计的时候不想让浏览器变得太复杂，因为多线程需要共享资源、且有可能修改彼此的运行结果，对于一种网页脚本语言来说，这就太复杂了。
* 比如，假定js同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？
* 在Java中会使用锁来解决这种竞态条件，而js并不想这样来解决。 
* 单线程模型带来的问题：主要是新的任务被加在队列的尾部，只有前面的所有任务运行结束，才会轮到它执行。如果有一个任务特别耗时，后面的任务都会停在那里等待，造成浏览器失去响应，又称“假死”。为了避免“假死”，当某个操作在一定时间后仍无法结束，浏览器就会跳出提示框，询问用户是否要强行停止脚本运行。
* javascript 是的单线程的，于是就产生了一种任务执行机制叫 <span style="color:red;">eventloop</span>。它维护了一个任务队列，完成一个任务才会开始下一个任务。

### setstate

* React通过this.state来访问state，通过this.setState()方法来更新state。当this.setState()方法被调用的时候，React会重新调用render方法来重新渲染UI
* Vue的DOM更新是异步的，所以如果需要等待DOM更新后执行操作就需要：

```
this.show = false
// 这里data更新了，但对应的DOM还没更新
Vue.nextTick(() => {
  // 这里跟`show`对应的DOM也被更新了
})
```

* 类似React里this.setState(state, callback)中的callback

**vue的实际问题**

* 在我们使用vue进行开发的过程中，可能会遇到一种情况：当生成vue实例后，当再次给数据赋值时，有时候并不会自动更新到视图上去；所以一般是把自己用到的值提前的保存在data中，避免这种问题的发生
* 例子：

```
var vm = new Vue({
	el:'#vm_box',
	data:{
		btn:'阿牛',
		box:'内容'
	}
})
vm.$data.sex = '男'
console.log(vm.$data);
```

![setDate](http://p1fg8xetu.bkt.clouddn.com/setDate.jpg)

* 根据上面的原理图，能够明白，当后添加属性的时候，并没有触发`Watcher`
* 所以在Vue3.0把这套原理删除掉，采用es6提出的`Proxy`和`Reflect`

## es6 proxy代理

> 扩展（增强）对象一些功能

**作用：**

* vue中的拦截
* 预警、上报、扩展功能、统计、增强对象等等
* 深入来说，几乎所有的需求都可以用到
* proxy是设计模式的一种，代理模式

**当访问对象没有的属性，抛出错误**

```
const obj = {
	name:'怪岸咖啡'
};
const newObj = new Proxy(obj,{
	get(target,property){
		if(property in target){
			return target[property];
		}else{
			console.warn('^_^')
			throw new ReferenceError(`${property}属性不在此对象上`);
		}
	}
});
console.log(newObj.name);
console.log(newObj.a);
```

## Reflect反射

* 操作对象而提供的API
* 方便了我们进行各种之前的操作，让其更加的合理和友好

## 实例：使用Proxy实现观察者模式

* 观察者模式（Observer mode）指的是函数自动观察数据对象，一旦对象有变化，函数就会自动执行

```
// 观察者函数集合
const queuedObservers = new Set();
const observe = fn => queuedObservers.add(fn);
// 返回一个原始对象Proxy代理
const observable = obj => new Proxy(obj, {set});
// 数据对象person是观察目标
const person = observable({
  name: '张三',
  age: 20
});

observe(print);
person.name = '李四';

// 函数print是观察者，数据对象发生变化，print就会自动执行
function print() {
  console.log(`${person.name}, ${person.age}`)
}
// 拦截函数set，会自动执行所有观察者
function set(target, key, value, receiver) {
  const result = Reflect.set(target, key, value, receiver);
  queuedObservers.forEach(observer => observer());
  return result;
}
```

## 虚拟dom-Virtual DOM

### 为什么说操作DOM是昂贵的？

* 直接代码

```
var dom = document.getElementById('vm_box');
var arr = [];
for(var item in dom){
	arr.push(item);
}
console.log(JSON.stringify(arr));
```

**一个基本的DOM身上绑定了235个属性和方法**

* 什么叫牵一发而动全身
* 什么叫庞然大物
* 这只是第一层，还可以多维遍历

`["align","title","lang","translate","dir","dataset","hidden","tabIndex","accessKey","draggable","spellcheck","contentEditable","isContentEditable","offsetParent","offsetTop","offsetLeft","offsetWidth","offsetHeight","style","innerText","outerText","onabort","onblur","oncancel","oncanplay","oncanplaythrough","onchange","onclick","onclose","oncontextmenu","oncuechange","ondblclick","ondrag","ondragend","ondragenter","ondragleave","ondragover","ondragstart","ondrop","ondurationchange","onemptied","onended","onerror","onfocus","oninput","oninvalid","onkeydown","onkeypress","onkeyup","onload","onloadeddata","onloadedmetadata","onloadstart","onmousedown","onmouseenter","onmouseleave","onmousemove","onmouseout","onmouseover","onmouseup","onmousewheel","onpause","onplay","onplaying","onprogress","onratechange","onreset","onresize","onscroll","onseeked","onseeking","onselect","onstalled","onsubmit","onsuspend","ontimeupdate","ontoggle","onvolumechange","onwaiting","onwheel","onauxclick","ongotpointercapture","onlostpointercapture","onpointerdown","onpointermove","onpointerup","onpointercancel","onpointerover","onpointerout","onpointerenter","onpointerleave","nonce","click","focus","blur","ontouchcancel","ontouchend","ontouchmove","ontouchstart","namespaceURI","prefix","localName","tagName","id","className","classList","slot","attributes","shadowRoot","assignedSlot","innerHTML","outerHTML","scrollTop","scrollLeft","scrollWidth","scrollHeight","clientTop","clientLeft","clientWidth","clientHeight","onbeforecopy","onbeforecut","onbeforepaste","oncopy","oncut","onpaste","onsearch","onselectstart","previousElementSibling","nextElementSibling","children","firstElementChild","lastElementChild","childElementCount","onwebkitfullscreenchange","onwebkitfullscreenerror","setPointerCapture","releasePointerCapture","hasPointerCapture","hasAttributes","getAttributeNames","getAttribute","getAttributeNS","setAttribute","setAttributeNS","removeAttribute","removeAttributeNS","hasAttribute","hasAttributeNS","getAttributeNode","getAttributeNodeNS","setAttributeNode","setAttributeNodeNS","removeAttributeNode","closest","matches","webkitMatchesSelector","attachShadow","getElementsByTagName","getElementsByTagNameNS","getElementsByClassName","insertAdjacentElement","insertAdjacentText","insertAdjacentHTML","requestPointerLock","getClientRects","getBoundingClientRect","scrollIntoView","scrollIntoViewIfNeeded","animate","before","after","replaceWith","remove","prepend","append","querySelector","querySelectorAll","webkitRequestFullScreen","webkitRequestFullscreen","scroll","scrollTo","scrollBy","createShadowRoot","getDestinationInsertionPoints","ELEMENT_NODE","ATTRIBUTE_NODE","TEXT_NODE","CDATA_SECTION_NODE","ENTITY_REFERENCE_NODE","ENTITY_NODE","PROCESSING_INSTRUCTION_NODE","COMMENT_NODE","DOCUMENT_NODE","DOCUMENT_TYPE_NODE","DOCUMENT_FRAGMENT_NODE","NOTATION_NODE","DOCUMENT_POSITION_DISCONNECTED","DOCUMENT_POSITION_PRECEDING","DOCUMENT_POSITION_FOLLOWING","DOCUMENT_POSITION_CONTAINS","DOCUMENT_POSITION_CONTAINED_BY","DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC","nodeType","nodeName","baseURI","isConnected","ownerDocument","parentNode","parentElement","childNodes","firstChild","lastChild","previousSibling","nextSibling","nodeValue","textContent","hasChildNodes","getRootNode","normalize","cloneNode","isEqualNode","isSameNode","compareDocumentPosition","contains","lookupPrefix","lookupNamespaceURI","isDefaultNamespace","insertBefore","appendChild","replaceChild","removeChild","addEventListener","removeEventListener","dispatchEvent"]`

### 虚拟dom诞生

* 相对于 DOM 对象，原生的 JavaScript 对象处理起来更快，而且更简单。DOM 树上的结构、属性信息我们都可以很容易地用 JavaScript 对象表示出来：

```
// html
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>

// js
var element = {
  tagName: 'ul', // 节点标签名
  props: { // DOM的属性，用一个对象存储键值对
    id: 'list'
  },
  children: [ // 该节点的子节点
    {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
  ]
}
```

* 从上面能够明白，虚拟dom是把我们要做的事情，最简单最直接的表现出来
* 没有任何的附加条件，而用dom操作一个dom第一层就有200多个属性和方法，可想而知多个dom是多么可怕的消耗
* 思考牵引：类似我们使用display:none/block;来回切换介绍优化性能，是一个道理，就是让我们最直接的做自己想做的事情

**document.createDocumentFragment**

* 创建一个虚拟的节点对象或者说，用来创建文档碎片节点。它可以包含各种类型的节点，在创建之初是空的
* 文档片段存在于内存中，并不在DOM树中，所以将子元素插入到文档片段时不会引起页面回流(对元素位置和几何上的计算)。因此，使用文档片段document fragments 通常会起到优化性能的作用


