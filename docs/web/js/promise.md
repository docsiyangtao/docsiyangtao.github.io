# 基本概念

## 对象

1. 函数对象：将函数作为对象使用时，成为函数对象。
2. 实例对象：new函数产生的对象，简称为对象。

```javascript
function Fn() {}
const fn = new Fn();
Fn.call()		// 这里的Fn是函数对象
fn.xxx			// 这里的fn是实例对象
```

`括号左边是函数，点的左边是对象`

## 回调函数

1. 同步回调：立即执行，完全执行了才结束，不回放入回调队列中

   ```javascript
   const arr = [1, 2, 3];
   arr.forEach(item => {
       console.log(item);
   });
   ```

2. 异步回调：不会立即执行，放入回调队列中执行

   ```javascript
   setTimeout(() => {
       console.log(123);
   }, 0)
   ```

## JS错误

1. 类型

   * Error：所有错误的父类型
   * ReferenceError：引用的变量不存在（xxx is not defined）
   * TypeError：数据类型不正确（Cannot read property 'xxx' of undefined / xxx if not a function）
   * RangeError：数据值不在其允许的范围内（Maximum call stack size exceeded）
   * SyntaxError：语法错误（Unexpect string）

2. 处理

   * 捕获

     ```javascript
     try {
         xxxxxx
     } catch (error) {
         console.log(error.message);		// 错误原因
         console.log(error.stack);		// 更多内容，含错误类型及出错位置
     }
     ```

   * 抛出

     ```javascript
     function handle(x) {
         if (x % 2 === 0) {
             throw new Error('错误信息');
         }
     }
     ```

   

# Promise

## 概念

* JavaScript中进行**异步编程的新的解决方案**
* Promise是一个**构造函数**
* Promise对象用于**封装一个异步操作并可以获取其结果**



## 状态

一个promise对象的状态**只能改变一次**，无论成功还是失败，都将**返回一个结果数据**，且**创建一个新的Promise对象**

* **成功**变为`resolved`
* **失败**变为`rejected`

![image-20220415232636168](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220415232636168.png)



## 初体验

```javascript
// 1. 创建Promise对象
const promise = new Promise((resolve, reject) => {
    // 2. 执行异步操作
    setTimeout(() => {
        const now = Date.now();
        // 成功操作
        if (now % 2 === 0) {
            resolve('success！');
        }
        // 失败操作
        else {
            reject('fail！');
        }
    }, 1000)
})

promise.then(
    // onResolved
    value => {
        console.log(value);
    },
    // onRejected
    reason => {
        console.log(reason);
    }
)
```



## 好处

1. **指定回调函数更加灵活**

   纯回调形式必须在启动异步任务前指定回调函数，而使用promise可以在启动异步任务之后给promise对象绑定回调函数（甚至可以在异步完成之后再指定回调函数）

   ```javascript
   const promise = new Promise((resolve, reject) => {
       if (xxx) {
           resolve('success');
       } else {
           reject('fail');
       }
   }).then(value => {
       console.log(value);		// success
   }).catch(reason => {
       console.log(reason);	// fail
   })
   ```

2. 链式调用**解决回调地域问题**

   回调函数嵌套调用，外部回调函数异步执行的结果是嵌套的回调函数执行的条件，不便于阅读及异常处理

   ```javascript
   // 传统
   doSomething(function(result) {
       doSomethingElse(result, function(newResult) {
           doThirdThing(newResult, function(finalResult) {
               console.log('The final result: ' + finalResult);
           }, failureCallback)
       }, failureCallback)
   }, failureCallback)
   
   
   // promise
   doSomething()
   .then(function(result) {
       return doSomethingElse(result)
   })
   .then(function(newResult) {
       return doThirdThing(newResult)
   })
   .then(function(finalResult) {
       console.log('The final result: ' + finalResult);
   })
   .catch(failureCallback)
   
   // 最佳实践
   async function request() {
       try {
           const result = await doSomething();
           const newResult = await doSomethingElse(result);
           const finalResult = await doThirdThing(newResult);
           console.log('The final result: ' + finalResult);
       } catch (error) {
           failureCallback(error)
       }
   }
   ```



## API

1. Promise构造函数：Promise (excutor) {}
    excutor函数：同步执行  (resolve, reject) => {}
    resolve函数：内部定义成功时调用的函数 value => {}
    reject函数：内部定义失败时调用的函数 reason => {}
    说明：excutor会在Promise内部立即同步回调，异步操作在执行器中执行
    
    ```javascript
    const promise = new Promise((resolve, reject) => {
        if (xxx) {
            resolve(xxx);
        } else {
            reject(xxx);
        }
    });
    ```
    
2. Promise.prototype.then方法：(onResolved, onRejected) => {}
    onResolved函数：成功的回调函数  (value) => {}
    onRejected函数：失败的回调函数 (reason) => {}
    说明：指定用于得到成功value的成功回调或用于得到失败reason的失败回调，返回一个新的promise对象

    ```javascript
    const p1 = promise.then(value => {
        // 成功操作
    });
    
    const p2 = promise.then(reason => {
        // 失败操作
    });
    ```

3. Promise.prototype.catch方法：(onRejected) => {}
    onRejected函数：失败的回调函数 (reason) => {}
    说明：then()的语法糖，相当于then(undefined, onRejected)

    ```javascript
    const p3 = promise.catch(reason => {
        // 失败操作
    })
    ```

4. Promise.resolve方法：(value) => {}
    value：成功的数据或promise对象
    说明：返回一个成功的promise对象

    ```javascript
    const p4 = Promise.resolve('成功时传的参数');
    p4.then(value => {
        console.log(value);		// 成功时传的参数
    })
    ```

5. Promise.reject方法：(reason) => {}
    reason：失败的原因
    说明：返回一个失败的promise对象

    ```javascript
    const p5 = Promise.reject('失败时传的参数');
    p5.then(null, reason => {
        console.log(reason);	// 失败时传的参数
    });
    
    p5.catch(reason => {
        console.log(reason);	// 失败时传的参数
    })
    ```

6. Promise.all方法：(promises) => {}
    promises：包含n个promise的数组
    说明：返回一个新的promise，只有所有的promise都成功才结果状态才为成功（value为所有成功的promise值数组），只要有一个失败了就直接失败（reason值为第一个失败的promise值）

    ```javascript
    const p6 = Promise.all([p4, p5]);
    p6.then(value => {
        console.log(value);
    }).catch(reason => {
        console.log(reason);
    })
    ```

7. Promise.race方法：(promises) => {}
    promises：包含n个promise的数组
    说明：返回一个新的promise，**第一个完成**（无论成功还是失败）的promise的结果状态及值就是最终的结果状态及值

    ```javascript
    const p7 = Promise.all([p4, p5, p6]);
    p7.then(value => {
        console.log(value);
    }).catch(reason => {
        console.log(reason);
    })
    ```



##   关键问题

1. **promise状态什么时候改变以及怎么改变**

   * resolve时：pending -> resolved
   * reject时：pending -> rejected
   * 抛出异常时：pending -> rejected

2. **一个promise指定多个成功/失败回调函数，都会调用吗？**

   * 会，只要promise状态改变

3. **改变promise状态和指定回调函数顺序？**

   都有可能

   * 先改变状态再指定回调函数

     ```javascript
     const promise = new Promise((resolve, reject) => {
         resolve(1);
     });
     setTimeout(() => {
         promise.then(value => {
             console.log(value);
         });
     }, 1000);
     ```

   * 先指定回调函数再改变状态

     ```javascript
     new Promise((resolve, reject) => {
         setTimeout(() => {
             resolve(1);
         }, 2000);
     }).then(value => {
         console.log(value);
     });
     ```

4. **回调函数获取数据的时机？**

   * 先改变状态再指定回调函数：回调函数调用时获取数据
   * 先指定回调函数再改变状态：改变状态时获取数据

5. **promise1.then()返回的新promise2状态由什么决定？**

   promise2的状态由promise1的回调函数执行的结果决定

   ```javascript
   const promise = new Promise((resolve, reject) => {
       resolve();
       // reject(2);
       // throw new Error('错误了');
   })
   
   const promise2 = promise.then(value => {
       return Promise.reject('promise1 resolved');
   }, reason => {
       return Promise.resolve('promise1 rejected');
   })
   
   promise2.then(value => {
       console.log('promise2 resolved & ' + value);
   }, reason => {
       console.log('promise2 rejected & ' + reason);
   })
   
   
   /*=======执行结果=======*/
   promise2 rejected & promise1 resolved
   ```

   promise1成功，此处定义其回调函数执行结果返回Promise.reject，因此promise2执行了失败的回调

6. **promise如何串连多个操作任务？**

   promise.then()返回一个新的promise，因此可通过`.then`形成链式调用，串连多个同步/异步任务

   ```javascript
   new Promise((resolve, reject) => {
       setTimeout(() => {
           console.log("执行任务1(异步)")
           resolve(1)
       }, 1000);
   }).then(
       value => {
           console.log('任务1的结果: ', value)
           console.log('执行任务2(同步)')
           return 2
       }
   ).then(
       value => {
           console.log('任务2的结果:', value)
           return new Promise((resolve, reject) => {
               // 启动任务3(异步)
               setTimeout(() => {
                   console.log('执行任务3(异步))')
                   resolve(3)
               }, 1000);
           })
       }
   ).then(
       value => {
           console.log('任务3的结果: ', value)
       }
   )
   
   
   /*=======执行结果=======*/
   执行任务1(异步)
   任务1的结果:  1
   执行任务2(同步)
   任务2的结果: 2
   执行任务3(异步))
   任务3的结果:  3
   ```

7. **promise异常穿透？**

   使用链式调用时，可以在最后指定失败的回调

   ```javascript
   new Promise((resolve, reject) => {
       let random = Math.floor(Math.random() * 1000);
       resolve(random);
   }).then(value => {
       console.log('promise1回调：' + value);
       if (value % 2 === 0) {
           return Math.floor(Math.random() * 1000);
       } else {
           return Promise.reject(value);
       }
   }).then(value => {
       console.log('promise2回调：' + value);
       if (value % 2 === 0) {
           return Promise.resolve('执行完成！');
       } else {
           return Promise.reject(value);
       }
   }).then(value => {
       console.log(value);
   }).catch(reason => {
       console.log('出错了：' + reason);
   })
   
   
   /*=======执行结果=======*/
   promise1回调：183
   出错了：183
   
   /*=======执行结果=======*/
   promise1回调：290
   promise2回调：274
   执行完成！
   ```

   调用链中，遇到奇数则会传到最后的回调函数中处理

8. **如何中断promise链？**

   在回调函数中返回一个pending状态的promise对象

   ```javascript
   new Promise((resolve, reject) => {
     let random = Math.floor(Math.random() * 1000);
     resolve(random);
   }).then(value => {
     console.log('promise1回调：' + value);
     if (value % 2 === 0) {
       return Math.floor(Math.random() * 1000);
     } else {
       console.log('遇到奇数，返回pending promise中断...');
       return new Promise(() => {});
     }
   }).then(value => {
     console.log('promise2回调：' + value);
     if (value % 2 === 0) {
       return Promise.resolve('执行完成！');
     } else {
       console.log('遇到奇数，返回pending promise中断...');		// 中断
       return new Promise(() => {});
     }
   }).then(value => {
     console.log(value);
   })
   
   
   /*=======执行结果=======*/
   promise1回调：40
   promise2回调：888
   执行完成！
   
   /*=======执行结果=======*/
   promise1回调：61
   遇到奇数，返回pending promise中断...
   ```



## async与await

* **async函数的返回值为promise对象**
* 如果返回值不是promise对象，则会**将返回值作为一个值创建一个Promise对象**，状态为fulfilled

```javascript
async function fn2() {
    return Promise.resolve(2);
}

const result = fn2();
result.then(value => {
    console.log('success: ' + value);
}, reason => {
    console.log('fail: ' + reason);
});


/*=======执行结果=======*/
success: 2
```

```javascript
async function fn1() {
    return 1;
}

const result = fn1();
console.log(result);


/*=======执行结果=======*/
Promise
[[Prototype]]: Promise
[[PromiseState]]: "fulfilled"
[[PromiseResult]]: 1
```

* 如果promise对象使用await修饰，则返回值为promise对象成功的值
* 如果表达式是其它值，直接将此值作为await的返回值
* 如果await的promise失败了, 就会抛出异常, 需要通过try...catch来捕获处理
* 注意：await对象必须在async函数中）

```javascript
function fn2() {
    // return Promise.resolve(2);
    // return Promise.reject(2);
    // throw new Error('2');
}

async function fn3() {
    try {
        const result = await fn2();
        console.log('结果本身：' + result);
    } catch (error) {
        console.log('出错了：' + error);
    }
}

fn3();


/*=======执行结果=======*/
结果本身：2

/*=======执行结果=======*/
出错了：2
```



## 宏队列与微队列

JS中用来存储待执行回调函数的队列有两种

1. **宏列队**：用来保存待执行的宏任务(回调)，如定时器回调/DOM事件回调/ajax回调
2. **微列队**：用来保存待执行的微任务(回调)，比如：promise的回调/MutationObserver的回调

![image-20220415232650544](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220415232650544.png)

**JS代码执行顺序**

1. 先执行所有的初始化同步任务代码
2. 执行所有微队列中的任务
3. 执行所有宏队列中的任务

```javascript
setTimeout(() => {
    console.log('定时器任务1');
    Promise.resolve(3).then(value => {
        console.log('Promise' + value);
    });
}, 0);
setTimeout(() => {
    console.log('定时器任务2');
}, 0);
Promise.resolve(1).then(value => {
    console.log('Promise' + value);
});
Promise.resolve(2).then(value => {
    console.log('Promise' + value);
});


/*=======执行结果=======*/
Promise1
Promise2
定时器任务1
Promise3
定时器任务2
```

分析：取出所有Promise任务，加入到微队列，此时已没有微任务，因此开始取宏任务，取第一个宏任务加入到宏队列中，该宏任务包含一个微任务，因此将微任务加入到微队列中，最后取剩余的宏任务加入到宏队列中。

**pending状态的promise并不会加入微队列**

```javascript
setTimeout(() => {
    console.log(1);
}, 0);

new Promise((resolve) => {
    console.log(2);
    resolve();
}).then(() => {
    console.log(3);
}).then(() => {
    console.log(4);
});

new Promise(resolve => {
    resolve();
}).then(() => {
    console.log(6)
});

console.log(5)


/*=======执行结果=======*/
2
5
3
6
4
1
```

分析：

1. 执行同步代码，打印2  5
2. 分配任务到队列，此时 微队列[3, 6]，宏队列[1]，第7行的promise还是pending状态，并没有加入微队列
3. 执行微任务，打印3  6，此时第8行promise状态变为resolve，因此加入微队列，微队列为[4]，宏队列为[1]
4. 继续执行微任务，打印4，微任务执行完，执行宏队列中的任务，打印1
5. 总体顺序：2  5  3  6  4  1



<center>【END】</center>
