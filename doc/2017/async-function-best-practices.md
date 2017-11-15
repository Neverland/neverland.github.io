# Node.js Async 函数最佳实践

`Node.js`7.6起, Node.js 搭载了有`async`函数功能的V8引擎。当Node.js 8于10月31日成为LTS版本后，我们没有理由不使用`async`函数。接下来，我将简要介绍`async`函数，以及如何改变我们编写Node.js应用程序的方式。

## 什么是`async`函数

`async`函数可以让你编写基于Promise的代码使他像同步的一样。每当你用`asnyc`关键字定义了一个函数体，你可以在函数体内使用 `await`关键字。当这个`async`函数被调用，会得到一个`Promise`实例。当`async`函数返回值时这个promise会执行。如果`async`函数抛出错误，那么会进入promise的rejected流程。

`await`关键字可以用来等待`Promise` 进入 `resolved`并有完成返回值。如果传给`await`的值不是`Promise`实例，它会被转为 `Promise`的 `resolved` 流程。 


````ecmascript 6
    const rp = require('request-promise');
    
    async function main () {
        const result = await rp('https://google.com');
        const twenty = await 20;
    
        // sleeeeeeeeping  for a second
        await new Promise (resolve => {
        setTimeout(resolve, 1000);
        });
        
        return result
    }

    main()
        .then(console.log)
        .catch(console.error);
````

## 迁移到`async`函数

如果你的Node.js应用已经使用了`Promise`, 你只需要使用 `await`替代Promise链式调用。如果你的代码是基于 `callback`, 迁移到 `async`函数需要逐步修改现有代码。可以在新到功能中使用新的技术，如果必须保留旧有功能则可以使用 `Promise` 进行简单的包装。为此你可以使用内置的`util.promisify`（译者注：Node.js 8.0+） 方法！

````ecmascript 6
    const util = require('util');
    const {readFile} = require('fs');
    const readFileAsync = util.promisify(readFile);
    
    async function main () {
      const result = await readFileAsync('.gitignore');
    
      return result
    }
    
    main()
      .then(console.log)
      .catch(console.error);
````

## `async`函数最佳实践

### 在`express`中使用`async`函数

As express supports Promises out of the box, using async functions with express is as simple as:

`express` 是支持 `Promise`的，所以使用`async`函数可以把代码简化为：

````ecmascript 6
    const express = require('express');
    const app = express();
    
    app.get('/', async (request, response) => {
        // awaiting Promises here
        // if you just await a single promise, you could simply return with it,
        // no need to await for it
        const result = await getContent();
        
        response.send(result);
    });
    
    app.listen(process.env.PORT);
````


Edit1：如Keith Smith所指出的那样，上面的例子有一个严重的问题 - 如果`Promise`进入`rejected`，express路由处理程序就会hang住，因为那里没有错误处理。

要解决这个问题，你应该把你的异步处理程序封装在一个处理错误的函数中：

````ecmascript 6
    const awaitHandlerFactory = middleware => {
    
        return async (req, res, next) => {
            try {
              await middleware(req, res, next)
            } catch (err) {
              next(err)
            }
        }
    }
    
    // and use it this way:
    app.get('/', awaitHandlerFactory(async (request, response) => {
        const result = await getContent();
        
        response.send(result);
    }));
````

### 并行

假设你正在做类似的事情，当一个操作需要两个输入，一个来自数据库，另一个来自外部服务：

````ecmascript 6
    async function main () {
      const user = await Users.fetch(userId);
      const product = await Products.fetch(productId);
    
      await makePurchase(user, product);
    }
````

在这个case中，将会发生以下情况：

 - 你的代码将首先获得用户资源，
 - 然后获得产品资源，
 - 并最终进行购买。

正如所见，你可以同时做前两个操作，因为它们之间没有依赖关系。 为此应该使用 `Promise.all` 方法：

````ecmascript 6
    async function main () {
        const [user, product] = await Promise.all([
            Users.fetch(userId),
            Products.fetch(productId)
        ]);
        
        await makePurchase(user, product);
    }
````

在某些情况下，您只需要最快`resolving`得到`Promise`的结果 - 在这种情况时可以使用`Promise.race`方法。

### 错误处理

参考下面的代码

````ecmascript 6
    async function main () {
        await new Promise((resolve, reject) => {
            reject(new Error('💥'));
        });
    };
    
    main()
        .then(console.log);
````

如果运行这段代码，你会在terminal上看到类似的消息：

````ecmascript 6
    (node:69738) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 2): Error: 💥
    (node:69738) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
````


在新版的Node.js中，如果Promise拒绝不会被处理，那么会导致整个Node.js进程。 因此，在必要时应该使用try-catch块：


````ecmascript 6
    const util = require('util');
    
    async function main () {
        try {
            await new Promise((resolve, reject) => {
              reject(new Error(' '));
            });
        } 
        catch (err) {
        // handle error case
        // maybe throwing is okay depending on your use-case
        }
    }
    
    main()
        .then(console.log)
        .catch(console.error);
````


但是，如果使用try-catch块，会丢失重要的异常如系统错误，那么就要重新抛出异常。 要了解更多关于什么时候应该重新投掷的信息，我强烈建议阅读Eran的[Learning to Throw Again.](https://medium.com/@eranhammer/learning-to-throw-again-79b498504d28)。

### 复杂的控制流程

Node.js的第一个异步控制流库是由 `Caolan McMahon` 编写的一[async](http://caolan.github.io/async/)的异步控制流库。 它提供了多个异步助手：

 - mapLimit,
 - filterLimit,
 - concatLimit,
 - priorityQueue.

如果你不想重新造轮子，而且不想使用这个库，那么可以重度使用 `async`函数和 `util .promisify`方法：


````ecmascript 6
    const util = require('util');
    const async = require('async');
    const numbers = [
        1, 2, 3, 4, 5
    ];

    mapLimitAsync = util.promisify(async.mapLimit);
    
    async function main () {
        return await mapLimitAsync(numbers, 2, (number, done) => {
            setTimeout(function () {
                done(null, number * 2);
            }, 100)
        });
    };
    
    main()
        .then(console.log)
        .catch(console.error);
````

[原文地址](https://nemethgergely.com/async-function-best-practices/)
