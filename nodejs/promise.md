


* 支持链式调用，解决回调地狱


```js
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.callbacks = [];

    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.callbacks.forEach(callback => callback.onFulfilled(value));
      }
    };

    const reject = (reason) => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.value = reason;
        this.callbacks.forEach(callback => callback.onRejected(reason));
      }
    };

    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }

  then(onFulfilled, onRejected) {
    if (this.state === 'fulfilled') {
      return new MyPromise((resolve, reject) => {
        try {
          const result = onFulfilled(this.value);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
    }

    if (this.state === 'rejected') {
      return new MyPromise((resolve, reject) => {
        try {
          const result = onRejected(this.value);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
    }

    if (this.state === 'pending') {
      return new MyPromise((resolve, reject) => {
        this.callbacks.push({
          onFulfilled: (value) => {
            try {
              const result = onFulfilled(value);
              resolve(result);
            } catch (error) {
              reject(error);
            }
          },
          onRejected: (reason) => {
            try {
              const result = onRejected(reason);
              resolve(result);
            } catch (error) {
              reject(error);
            }
          }
        });
      });
    }
  }

  catch(onRejected) {
    return this.then(undefined, onRejected);
  }
}

// 示例用法
const promise = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    const randomValue = Math.random();
    if (randomValue > 0.5) {
      resolve(randomValue);
    } else {
      reject("Value is too small");
    }
  }, 1000);
});

promise
  .then(value => {
    console.log("Success:", value);
    return value * 2;
  })
  .then(value => {
    console.log("Double:", value);
  })
  .catch(error => {
    console.error("Error:", error);
  });

```


参考:   
[简述promise原理](https://juejin.im/post/5ac5c4465188255c3200f51d)     
[剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类](https://github.com/xieranmaya/blog/issues/3)