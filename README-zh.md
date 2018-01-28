# Node Schedule

[![NPM version](http://img.shields.io/npm/v/node-schedule.svg)](https://www.npmjs.com/package/node-schedule)
[![Downloads](https://img.shields.io/npm/dm/node-schedule.svg)](https://www.npmjs.com/package/node-schedule)
[![Build Status](https://travis-ci.org/node-schedule/node-schedule.svg?branch=master)](https://travis-ci.org/node-schedule/node-schedule)
[![Join the chat at https://gitter.im/node-schedule/node-schedule](https://img.shields.io/badge/gitter-chat-green.svg)](https://gitter.im/node-schedule/node-schedule?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![NPM](https://nodei.co/npm/node-schedule.png?downloads=true)](https://nodei.co/npm/node-schedule/)

Node Schedule 是 Nodejs 中一款灵活的类 cron 但又不完全像 cron 的任务调度管理库
它允许你使用可选的 recurrence 规则, 在特定的时间执行任务调度(任意 function)
它在给定一个准确的时间后只使用一个单一的定时器而不是每分每秒去重新运算任务的执行时间

## 用法

### 安装

你可以使使用 [npm](https://www.npmjs.com/package/node-schedule) 来安装.

```
npm install node-schedule
```

### 概述

Node Schedule 是基于时间调度而不是基于时间间隔调度的
尽管你可以很容易的按照自己的需求去实现一些功能, 比如你只是想做类似 "每五分钟运行一次 function"
你会发现 `setInterval` 实现起来更为容易并且也比较适用. 但如果你想这样运行, 比如 
"每个月的第三个星期二的每个小时的第20分钟和第50分钟运行一次 function" 你会发现 Node Schedule
会更符合你的需求. 另外, 除非 Nodejs 实现完全的支持, 否则
Node Schedule 在windows下可能不会对 cron 实现完全的支持. 

提示一下, Node Schedule 是为处于进程中的调度而设计的, 打个比方, 只有当你的脚本处于运行中时, 
任务调度才会触发, 并且在脚本执行完毕后任务调度会消失. 如果你需要在脚本处于未运行状态下依然保持
任务调度, 你可能需要真正的 [cron] 去实现. 

### 任务和调度

任何一个可以调度的任务在 Node Schedule 中都是由 `Job` 对象来表示. 你可以手动的创建任务然后执行 
`schedule()` 方法去应用一个调度, 或者使用更为方便的 `scheduleJob()` function , 在后面会提到


所有 `Job` 对象都是继承自 `EventEmitter` 的, 并且在每次执行之后都会 emit 一个 `run` 事件. 
并且在他们开始调度执行时还会 emit 一个 `scheduled` 事件, 在一个 invocation 执行之前取消的话, 
会产生一个 `canceled` 事件(所有的事件都会带有一个 JavaScript 的时间对象作为参数). 注意, 
任务的调度会在第一时间执行, 因此如果你使用 `scheduleJob()` 这个便捷方法, 你可能会收不到第一个
`scheduled` 事件, 但是你可以手动去查询 invocation (参阅下文). 另外注意, `canceled` 使用的是
美式拼写, 是一个 l 而不是 两个 l. 


### Cron风格的调度

 cron 格式如下:
```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    │
│    │    │    │    │    └ 一个星期中的某一天 (0 - 7) (0 或 7 表示的是周日)
│    │    │    │    └───── 月份 (1 - 12)
│    │    │    └────────── 一个月中的某一天 (1 - 31)
│    │    └─────────────── 小时 (0 - 23)
│    └──────────────────── 分钟 (0 - 59)
└───────────────────────── 秒 (0 - 59, 可选的)
```

cron 格式的一些例子:

```js
var schedule = require('node-schedule');

var j = schedule.scheduleJob('42 * * * *', function(){
  console.log('一切物种生命, 宇宙万物的答案!');
});
```

每个小时的42分钟执行 cron 任务 (e.g. 19:42, 20:42, 等等.).

还有:

```js
var j = schedule.scheduleJob('0 17 ? * 0,4-6', function(){
  console.log('Rebecca Black 设定的是今天!');
});
```

每五分钟执行一次 cron 任务 = */5 * * * *

在每个任务的 invocation 执行时, 你都可以获取到它是什么时候被调度的: 
```js
var j = schedule.scheduleJob('0 1 * * *', function(fireDate){
  console.log('这个任务被设定在 ' + fireDate + ' 运行, 但实际运行时间是:  ' + new Date());
});
```
在系统繁忙的状态下当你需要检查在任务 invocation 的时候是否产生了延时会非常有用, 或者为了 audit 可以保存这个任务的所有 invocation 的记录
#### 不支持的 Cron 特性

目前, `W` (最近的工作日), `L` (每月/每星期的最后一天), 和 `#` (每个月的第n个工作日) 还不支持. 大多数比较受欢迎的 cron 实现都支持的很好.

[cron-parser] 用于解析 crontab 指令.

### 基于日期的调度

假如你希望在2012年12月21日上午5点30分执行一个 function.
请记住 - 在 JavaScript 中 - 0 代表 1月, 11 代表 12月.

```js
var schedule = require('node-schedule');
var date = new Date(2012, 11, 21, 5, 30, 0);

var j = schedule.scheduleJob(date, function(){
  console.log('今天是世界末日.');
});
```

你可以使用 bind 在以后使用当前的数据:

```js
var schedule = require('node-schedule');
var date = new Date(2012, 11, 21, 5, 30, 0);
var x = 'Tada!';
var j = schedule.scheduleJob(date, function(y){
  console.log(y);
}.bind(null,x));
x = 'Changing Data';
```
当调度任务执行时, 这将会打印出 'Tada!' 而不是 'Changing Data'
是因为 x 是在调度之后才立即改变的

### Recurrence 调度规则

你也可以构建一个 recurrence 规则来声明任务在何时可以 recur. 比如, 
下面这个规则就是在每个小时的42分钟会执行 function: 

```js
var schedule = require('node-schedule');

var rule = new schedule.RecurrenceRule();
rule.minute = 42;

var j = schedule.scheduleJob(rule, function(){
  console.log('一切物种生命, 宇宙万物的答案!');
});
```

你也可以使用数组去声明一个可用的数值的列表,  `Range` 对象可以接受一个可选的步长参数
用来声明一个数值开始和结束的范围. 比如, 
下面这个例子会在 周四, 周五, 周六, 周日的下午5点打印结果

```js
var rule = new schedule.RecurrenceRule();
rule.dayOfWeek = [0, new schedule.Range(4, 6)];
rule.hour = 17;
rule.minute = 0;

var j = schedule.scheduleJob(rule, function(){
  console.log('Rebecca Black 设定的是今天!');
});
```

#### Recurrence规则成员

- `second`
- `minute`
- `hour`
- `date`
- `month`
- `year`
- `dayOfWeek`

> **提示**: 值得注意的是, 每个成员的默认值都是 `null` 
>  (除了 second, second 默认值是 0 ,这样对 cron 更友好). *如果我们没有明确的设置 `minute` 为 0, 消息可能会在
> 5:00pm, 5:01pm, 5:02pm, ..., 5:59pm 被打印.* 这可能不是我们想要的结果.

#### 对象字面量语法

为了让事情变得简单, 对象字面量语法也是支持的, 比如下面这个例子会在每个周日的2:30pm打印消息:

```js
var j = schedule.scheduleJob({hour: 14, minute: 30, dayOfWeek: 0}, function(){
  console.log('下午茶时间!');
});
```

#### 设置 StartTime 和 EndTime

下面这个例子会在5秒后执行并且在10秒后停止.
上面的规则也是支持的.

```js
let startTime = new Date(Date.now() + 5000);
let endTime = new Date(startTime.getTime() + 5000);
var j = schedule.scheduleJob({ start: startTime, end: endTime, rule: '*/1 * * * * *' }, function(){
  console.log('喝茶时间!');
});
```

### 处理任务和任务的 Invocations

有一些可以获取任务信息和处理任务以及任务的 Invocations 的 function.


#### job.cancel(reshedule)
使用 `cancel()` 方法, 你可以立即使任何任务无效:

```js
j.cancel();
```
所有计划的 invocations 将会被取消. 当你设置参数 ***reschedule***为 true 时, 调度会在之后重新被安排

#### job.cancelNext(reshedule)
这个方法可以使下一个计划的任务的 invocation 无效.
当你设置参数 ***reschedule***为 true 时, 调度会在之后重新被安排.

#### job.reschedule(spec)
这个方法会取消所有待处理的任务并且根据给定的 spec 重新注册任务调度.
成功返回 true 失败返回 false.

#### job.nextInvocation()
这个方法会返回这个任务的下一次 invocation 的时间. 如果没有 invocation 则会返回 null.

## 贡献代码

这个模块由 [Matt Patenaude] 开发, 由
[Tejas Manohar] 和 [其他卓越贡献者] 所维护.

我们很希望你能来贡献代码. 任何个人都可以贡献他们认为对该项目重要的或有价值的代码.

在入坑之前, 请先查阅我们入坑 [指南] !

## Copyright and license

Copyright 2015 Matt Patenaude.

Licensed under the **[MIT License] [license]**.


[cron]: http://unixhelp.ed.ac.uk/CGI/man-cgi?crontab+5
[贡献代码]: https://github.com/node-schedule/node-schedule/blob/master/CONTRIBUTING.md
[Matt Patenaude]: https://github.com/mattpat
[Tejas Manohar]: http://tejas.io
[license]: https://github.com/node-schedule/node-schedule/blob/master/LICENSE
[Tejas Manohar]: https://github.com/tejasmanohar
[其他卓越贡献者]: https://github.com/node-schedule/node-schedule/graphs/contributors
[cron-parser]: https://github.com/harrisiirak/cron-parser
