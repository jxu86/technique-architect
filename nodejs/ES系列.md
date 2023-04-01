




### ES6（2015）
1. 类（class）
```js
class Man {
  constructor(name) {
    this.name = '小豪';
  }
  console() {
    console.log(this.name);
  }
}
const man = new Man('小豪');
man.console(); // 小豪
```

2. 模块化(ES Module)
```js
// 模块 A 导出一个方法
export const sub = (a, b) => a + b;
// 模块 B 导入使用
import { sub } from './A';
console.log(sub(1, 2)); // 3
```

3. 箭头（Arrow）函数
```js
const func = (a, b) => a + b;
func(1, 2); // 3
```

4. 函数参数默认值
```js
function foo(age = 25,){ // ...}
```

5. 模板字符串
```js
const name = '小豪';
const str = `Your name is ${name}`;
```

6. 解构赋值
```js
let a = 1, b= 2;
[a, b] = [b, a]; // a 2  b 1
```

7. 延展操作符
```js
let a = [...'hello world']; // ["h", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d"]
```

8. 对象属性简写
```js
const name='小豪',
const obj = { name };
```

9. Promise
```js
Promise.resolve().then(() => { console.log(2); });
console.log(1);
// 先打印 1 ，再打印 2
```

10. let和const
```js
let name = '小豪'；
const arr = [];
```
### ES7（2016）
1. Array.prototype.includes()
```js
[1].includes(1); // true
```

2. 指数操作符
```js
2**10; // 1024
```

### ES8（2017）
1. async/await
异步终极解决方案
```js
async getData(){
    const res = await api.getTableData(); // await 异步任务
    // do something    
}
```

2. Object.values()

```js
Object.values({a: 1, b: 2, c: 3}); // [1, 2, 3]
```
3. Object.entries()
```js
Object.entries({a: 1, b: 2, c: 3}); // [["a", 1], ["b", 2], ["c", 3]]
```

4. String padding
```js
// padStart
'hello'.padStart(10); // "     hello"
// padEnd
'hello'.padEnd(10) "hello     "
```
5. 函数参数列表结尾允许逗号

6. Object.getOwnPropertyDescriptors()   
获取一个对象的所有自身属性的描述符,如果没有任何自身属性，则返回空对象。

7. SharedArrayBuffer对象    
SharedArrayBuffer 对象用来表示一个通用的，固定长度的原始二进制数据缓冲区，
```js
/**
 * 
 * @param {*} length 所创建的数组缓冲区的大小，以字节(byte)为单位。  
 * @returns {SharedArrayBuffer} 一个大小指定的新 SharedArrayBuffer 对象。其内容被初始化为 0。
 */
new SharedArrayBuffer(10)
```
8. Atomics对象          
Atomics 对象提供了一组静态方法用来对 SharedArrayBuffer 对象进行原子操作。


### ES9（2018）

1. 异步迭代

await可以和for...of循环一起使用，以串行的方式运行异步操作
```js
async function process(array) {
  for await (let i of array) {
    // doSomething(i);
  }
}
```
2. Promise.finally()

```js
Promise.resolve().then().catch(e => e).finally();
```

3. Rest/Spread 属性
```js
const values = [1, 2, 3, 5, 6];
console.log( Math.max(...values) ); // 6
```

4. 正则表达式命名捕获组
```js
const reg = /(?<year>[0-9]{4})-(?<month>[0-9]{2})-(?<day>[0-9]{2})/;
const match = reg.exec('2021-02-23');
```




参考:   
[JS语法 ES6、ES7、ES8、ES9、ES10、ES11、ES12新特性](https://segmentfault.com/a/1190000039272641)