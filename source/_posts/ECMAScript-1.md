---
title: ECMAScript_1
date: 2019-07-04 13:38:29
tags:
- JavaScript
- ES6
categories: JavaScript
author: 小曾同学
---

今天早晨和昨天晚上都在学习ES6
本篇博客并没有列出所有特性
ES6解决了非常多的诟病

<!-- more -->
# ESMAScipt6_1

## 简介

1. ECMAScipt和JavaScipt

   - ECMA是标准，JS是实现
     - 类似于HTML5是标准，IE10，chrome，FF都是实现
     - 将来也有可能有其他的XXXScipt来实现ECMA

   - ECMAScipt简称ECMA或者ES

## 变量

### var

1. 可以重复声明
2. 无法限制修改
3. 没有块级作用域 （for，if，这样的作用域），像开发过java的同学就知道

### let

1. 不可以重复声明
2. 可以修改
3. 具有块级作用域

### const

1. 不可以重复声明
2. 不可以修改
3. 具有块级作用域

> 讲实话，干过java的同志们，var 的这三个特性，简直是难受的一批，java里面重复声明一个变量，我能把你骂的找不到家

## 箭头函数

原来的函数声明

```javascript
function method (){
  console.log('这是一个函数');
}
```

箭头函数

```js
()=>{
  console.log('这是一个箭头函数');
}
```

1. 如果只有一个参数，`（）`可以省略
2. 如果方法体只有一个return，`{} `可以省略

Example:

```javascript
let show =a=>a*2;
alert(show(10));
```

结果为20

> 这个箭头函数有点像Python里面lambda表达式，还有JDK1.8的lambda表达式
>
> emmmm，从 java 到 php到 js 这路上真坎坷啊，java语法严厉，一板一眼，php 和 js 身为脚本语言，特性完全不同，写的好不习惯….
>
> 慢慢来！！

## 参数

### 收集参数

```js
function show (a,b,...args){
	alert(a);
  alert(b);
  alert(args);
}
show(1,2,3,4,5,6,7);
```

我们可以看到，他除了a与b，其他的参数，全堆到了args里面

但，有一种错误的写法

```js
function show (a,b,...args,c){
	alert(a);
  alert(b);
  alert(args);
}
show(1,2,34,4,5);
```

这样写是错误的，`...args` 必须**放在最后**，语法要求！

### 展开参数

```js
let arr =[1,2,3];
show(..arr);

function show(a,b,c){
    alert(a);
    alert(b);
    alert(c);
}
```

它会逐个打印出来，意思就是将arr数组里面元素，展开入参。

展开后的效果，就跟直接把数组的内容直接写在这里一样。

```js
let a;
let arr=[1,2,3];
a=...arr;
alert(a);
```

不过这样写是错误的，这样赋值太诡异了。

## 默认参数

```js
function test(a,b=12,c=15){
  alert(a);
  alert(b);
  alert(c);
}
show(10);
```

就是这么简单

但如果在这里传递了参数，那么就不会按默认值来计算了

## 解构赋值

Example:

```js
let [a.b,c] = [1,2,3];
console.log(a,b,c);

let [a1,b1,c1]= {a1:12,b1:13,c1:14};
console.log(a1,b1,c1);
```

1. 左右两边解构必须一样
2. 右边必须是个东西
3. 声明和赋值不能分开（必须在一行话里完成）

## 数组

map——映射

reduce—汇总

filter——过滤器

forEach--循环



### MAP

这个新增函数，简单来说，就是一个对一个

Example:

```js
let arr =[1,2,3,4];
arr.map(function(item){
  console.log(item);
});
```

Item 就是数组中的每一个元素

```js
let arr = [11,90,22,60,80];
let result = arr.map(item=>item>60?'及格':'不及格');
```

配合箭头函数可以这样使用

### REDUCE

这个是稍微比较复杂的一个，是一堆里面出来一个

Example:

```js
let arr = [80,90,100,120];
arr.reduce(function(temp,item,index){
  return temp+item;
});
```

第一个参数，就是上次的计算结果，由于第一次没有计算，所以temp=80，然后就是80+90，再就是80+90+100

第二个参数，就是下次要计算的数字，temp = 80 的时候，item 就是 90，temp= 170 的时候，就是100

第三个参数，是下标，此下标从1开始，因为，temp是从80开始的，所以下标是从1开始



### FILTER

就是过滤器，返回true被接受，返回false不被接受

做一个例子，不能被三整除的都要被过滤掉

```js
let arr = [13,4,1,33];
let result = arr.filter(item=>item/3>0?false:true);
```

配合箭头函数可以这样使用



### FOREACH

有编程基础的同志们，这个都不用讲

example

```js
let arr = [1,2,3,4,5];
arr.forEach((item,index)=>{
  alert(item + ':' + index);
});
```

就是这样使用的，第一个为元素内容，第二个为下标。

写的眼睛酸了，下次见

HAPPY CODING !!!


