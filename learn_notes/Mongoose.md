# 每个项目都要单独安装
```sh
创建目录 mkdir /mongoose   cd /mongoose
npm install mongoose
```
# 配置与连接
```sh
var mongoose=require("mongoose");
var uri=' mongodb://username:password@hostname:port/databasename';
uri='mongodb://localhost/dbname';
mongoose.connect(uri);
```
# model与schema
- model模块建立nodejs中的对象，与mongodb中的文档对应，操作model即可对mongodb的文档增删改查
- schema提供对model中类型和数据结构的定义，从而在无模式的mongodb中实现模式化数据存储

```sh
文件名为model.js
ar mongoose=require('mongoose');
var uri='mongodb://localhost:27017/newdb';

mongoose.connect(uri);
mongoose.connection.on('connected', function () {    
    console.log('Mongoose connection open to ' + uri);  
});    

/**
 * 连接异常
 */
mongoose.connection.on('error',function (err) {    
    console.log('Mongoose connection error: ' + err);  
});    
 
/**
 * 连接断开
 */
mongoose.connection.on('disconnected', function () {    
    console.log('Mongoose connection disconnected');  
});

var BookSchema=new mongoose.Schema({//一个参数，定义数据结构和类型
name:String,
author:String,
pulishTime:Date
});

mongoose.model('Book',BookSchema);//schema创建完毕即可创建model,名为Book,第一个字符最好大写

```
```sh
文件名为insert.js
var mongoose=require('mongoose');
require('./model.js');
var Book=mongoose.model('Book');//此时返回Book的model
var book=new Book({
	name:"zhangsan",
	author:"lisi",
	pulishTime:new Date()
});
book.author='JIM';
book.save(function(err){//保存方法的回调函数
console.log('status',err?'faile':'suc');
});
```
```sh
node insert.js
```
