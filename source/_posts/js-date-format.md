
---
title: JavaScript Date 格式化 以及 本周本月的获取
category: js
tags: js
---

获取本周第一天和最后一天，Date获取本月第一天和最后一天。

#  Date 的主要使用----单独取值
```js
Date() //------返回当日的日期和时间。
getDate() //---从 Date 对象返回一个月中的某一天 (1 ~ 31)。
getDay() //----从 Date 对象返回一周中的某一天 (0 ~ 6)。
getMonth()//---从 Date 对象返回月份 (0 ~ 11)。
getFullYear()//从 Date 对象以四位数字返回年份。
getYear()//----请使用 getFullYear() 方法代替。
getHours()//---返回 Date 对象的小时 (0 ~ 23)。
getMinutes()// -返回 Date 对象的分钟 (0 ~ 59)。
getSeconds()//-返回 Date 对象的秒数 (0 ~ 59)。
getMilliseconds()//返回 Date 对象的毫秒(0 ~ 999)。
getTime()//-----返回 1970 年 1 月 1 日至今的毫秒数。
```
<!--more-->
#  Date 的主要使用----赋值
```js
setDate()//-----设置 Date 对象中月的某一天 (1 ~ 31)。
setMonth()//----设置 Date 对象中月份 (0 ~ 11)。
setFullYear()//-设置 Date 对象中的年份（四位数字）。
setYear()//-----请使用 setFullYear() 方法代替。
setHours()//----设置 Date 对象中的小时 (0 ~ 23)。
setMinutes()//--设置 Date 对象中的分钟 (0 ~ 59)。
setSeconds()//--设置 Date 对象中的秒钟 (0 ~ 59)。
setMilliseconds()//设置 Date 对象中的毫秒 (0 ~ 999)。
setTime()//------以毫秒设置 Date 对象。
```
#  Date 的主要使用----格式化值
```js
toString()//-----把 Date 对象转换为字符串。没多大意义
toTimeString()//-把 Date 对象的时间部分转换为字符串。标准格式，没多大意义
toDateString()//-把 Date 对象的日期部分转换为字符串。标准格式，没多大意义
toUTCString()//-根据世界时，把 Date 对象转换为字符串。
toLocaleString()//根据本地时间格式，把 Date 对象转换为字符串。
toLocaleTimeString()//根据本地时间格式，把 Date 对象的时间部分转换为字符串。当然是按照格式来的，不过不能自定义格式
toLocaleDateString()//根据本地时间格式，把 Date 对象的日期部分转换为字符串。 当然是按照格式来的，不过不能自定义格式
d.valueOf()//----返回的是定义时的值，没多大意义，如果使用new Date（）那么返回毫秒数，等价于getTime（）
```
#  Date 的格式化函数，简单格式化
```js
// 对Date的扩展，将 Date 转化为指定格式的String
// 月(M)、日(d)、小时(h)、分(m)、秒(s)、季度(q) 可以用 1-2 个占位符，
// 年(y)可以用 1-4 个占位符，毫秒(S)只能用 1 个占位符(是 1-3 位的数字)
// 例子：
// (new Date()).Format("yyyy-MM-dd hh:mm:ss.S") ==> 2006-07-02 08:09:04.423
// (new Date()).Format("yyyy-M-d h:m:s.S")      ==> 2006-7-2 8:9:4.18
Date.prototype.format = function(fmt){
//author: meizz
  var o = {
    "M+" : this.getMonth()+1,                 //月份
    "d+" : this.getDate(),                    //日
    "h+" : this.getHours(),                   //小时
    "m+" : this.getMinutes(),                 //分
    "s+" : this.getSeconds(),                 //秒
    "q+" : Math.floor((this.getMonth()+3)/3), //季度
    "S"  : this.getMilliseconds()             //毫秒
  };
  if(/(y+)/.test(fmt))
    fmt=fmt.replace(RegExp.$1, (this.getFullYear()+"").substr(4 - RegExp.$1.length));
  for(var k in o)
    if(new RegExp("("+ k +")").test(fmt))
            fmt = fmt.replace(RegExp.$1, (RegExp.$1.length==1) ?
                     (o[k]) : (("00"+ o[k]).substr((""+ o[k]).length)));
 return fmt;
}
// 示例代码
var today = new Date().format('yyyy-MM-dd hh:mm:ss.S');
console.info(today);
```
#  Date 获取本月第一天 最后一天 ，本周第一天 最后一天
```js
/**
 * 本周第一天
 */
function showWeekFirstDay()
 {
     var Nowdate=new Date();
     var WeekFirstDay=new Date(Nowdate-(Nowdate.getDay()-1)*86400000);
     return WeekFirstDay.format('yyyy-MM-dd hh:mm:ss.S')
 }
 /**
  * 本周最后一天
  */
 function showWeekLastDay()
 {
     var Nowdate=new Date();
     var WeekFirstDay=new Date(Nowdate-(Nowdate.getDay()-1)*86400000);
     var WeekLastDay=new Date((WeekFirstDay/1000+6*86400)*1000);
     return WeekLastDay.format('yyyy-MM-dd hh:mm:ss.S')
 }
 /**
  * 本月第一天
  */
 function showMonthFirstDay()
 {
     var Nowdate=new Date();
     var MonthFirstDay=new Date(Nowdate.getFullYear(),Nowdate.getMonth(),1);
     return MonthFirstDay.format('yyyy-M-d hh:mm:ss.S')
 }
 /**
  * 本月最后一天
  */
 function showMonthLastDay()
 {
     var Nowdate=new Date();
     var MonthNextFirstDay=new Date(Nowdate.getFullYear(),Nowdate.getMonth()+1,1);
     var MonthLastDay=new Date(MonthNextFirstDay-86400000);
     return MonthLastDay.format('yyyy-MM-dd hh:mm:ss.S')
 }
 ```