---
title: google translate
date: 2017-07-14 09:07:23
tags: [javascript]
categories: 技术
---

## 需求
使用Google translate api 批量翻译文件

## 调研

### Google translate API
[官方文档](https://cloud.google.com/translate/docs/)

需要使用google云平台提供的服务
1. 开通云平台（试用1年）
2. 新建google translate 项目
3. 生成key

[使用限制](https://cloud.google.com/translate/limits)：
1. 每天 2,000,000 characters 
2. 100 seconds 100,000 characters
3. 100 seconds 1000 request

Google会提供每日统计结果。
![我的使用记录](http://om49hkcv7.bkt.clouddn.com/gapi_%20-%20https___console.cloud.google.com_ap.png)

<!-- more -->

### javascript for google translate api

使用[github 上Localize 的google-translate](https://github.com/Localize/node-google-translate)

### 多语言文件处理(csv格式)
使用[github 上 fast-csv](https://github.com/C2FO/fast-csv)

## 开发
### 问题一，javascript stream 处理
readstream  -> pipe( processStream )  
processStream -> pipe( writeStream)

fast-csv 即 processStream
on("data", callProcessFunction(){//process;})
### 问题二，nodejs 实现 for循环中 同步等待后执行
原因： 单条循环调用，差不多500多笔 google开始返回报错

解决思路：使用 定时器，每批调用后 等待3秒， 需要同步等待
尝试了 generator co
最终： 使用递归调用

``` javascript
let csvdata  = []; // array for translate
let ITEM_SIZe = 20; // translate record number every time

// for iterator sleep delay senconds
function forWithDelay(i, length, array, fn, delay) {
   setTimeout(function () {
      fn(i, length, array);
      i++;
      if (i * length < array.length) {
         forWithDelay(i, length, array, fn, delay); 
      }
  }, delay);
}

// function call google translate & process data copy from array 
function callTranslate(i, length, array) {
    let tranData = [];
    const arrayLength = array.length;
    for( let j = 0; j<length; j++) {
        if (i*length + j < arrayLength){
            tranData[j] = array[i*length +j][1];
            array[i*length +j][2] ="AA";
            //console.log(array[i*length +j]);
        }
    }
    googleTranslate.translate(tranData, 'en', function(err, translation) {
        if (err) {
            console.log(err.stack());
            return;
        }
        if (!translation ) {
            console.log("EEE");
        } else {
            translation.forEach(function(item,index,arr) {
                console.log(index);
                console.log(item.translatedText);
                array[i*length+index][2] = item.translatedText;
                console.log(array[i*length+index]);
                csvWriteStream.write(array[i*length+index]);
            });
        }
    });
}
forWithDelay(0, ITEM_SIZE, csvdata, callTranslate, 3000);
```

