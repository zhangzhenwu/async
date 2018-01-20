Javascript语言的执行环境是单线程的，即一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。虽然这种模式实现起来比较简单，执行环境相对单纯，只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。因此，Javascript还有异步模式，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。比如最常用的Ajax。下面就简单的介绍下异步模式的前世今生。

1.回调函数实现
```
let fs = require('fs');
fs.readFile('./3.js','utf-8',(err,data)=>{
    if (err) return;
    console.log(data);
})
console.log('begin');
```
上例结果首先会打印begin，然后在回调函数中获取读取文件的结果。但是回调函数有缺点：1.不能用try catch捕捉错误 2.不能return 3.容易进入回调低于，使得代码难以维护，不易阅读

2.事件发布/订阅模型
```
let fs = require('fs');
let EventEmitter = require('events');
let eve = new EventEmitter();
let html = {};
eve.on('ready',function(key,value){
  html[key] = value;
  if(Object.keys(html).length==2){
    console.log(html);
  }
});
function render(){
  fs.readFile('./3.js','utf8',function(err,data3){
    eve.emit('ready','data3',data3);
  })
  fs.readFile('./1.js','utf8',function(err,data1){
    eve.emit('ready','data1',data1);
  })
}
render();
```
3.哨兵变量
```
function render(length,cb){
  let html={};
  return function(key,value){
    html[key] = value;
    if(Object.keys(html).length == length){
      cb(html);
    }
  }
}
let done = render(2,function(html){
  console.log(html);
});
fs.readFile('./3.js', 'utf8', function (err, data3) {
  done('data3',data3);
})
fs.readFile('./1.js', 'utf8', function (err, data1) {
  done('data1',data1);
})
```
4.Promises对象+Generator
Promises对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。然后利用co模块进行自动调用
```
let fs = require('fs');
function readFile(filename) {
  return new Promise(function (resolve, reject) {
    fs.readFile(filename, 'utf8', function (err, data) {
      err ? reject(err) : resolve(data);
    });
  })
}
function *read() {
  console.log('Begin');
  let a = yield readFile('1.txt');
  console.log(a);
  let b = yield readFile('2.txt');
  console.log(b);
  let c = yield readFile('3.txt');
  console.log(c);
  return c;
}
function co(gen) {
  let it = gen();//我们要让我们的生成器持续执行
  return new Promise(function (resolve, reject) {
    !function next(lastVal) {
        let {value,done} = it.next(lastVal);
        if(done){
          resolve(value);
        }else{
          value.then(next,reject);
        }
    }()
  });
}
co(read).then(function (data) {
  console.log(data);
});
```
5.使用async关键字，是目前我觉得最好的方法，也是现在在工作中使用最多的一种方法，它的语法比较简洁，值得注意的是它仅仅是语法糖而已，底层还是Promises。
```
async function read(){
 //await后面必须跟一个promise,
 let a = await readFile('./1.txt');
 console.log(a);
 let b = await readFile('./2.txt');
 console.log(b);
 let c = await readFile('./3.txt');
 console.log(c);
 return 'end';
 }
```
