### 和浏览器事件循环差异
在node中，事件循环表现出的状态与浏览器中大致相同。不同的是node中有一套自己的模型。node中事件循环的实现是依靠的libuv引擎。我们知道node选择chrome v8引擎作为js解释器，v8引擎将js代码分析后去调用对应的node api，而这些api最后则由libuv引擎驱动，执行对应的任务，并把不同的事件放在不同的队列中等待主线程执行。 因此实际上node中的事件循环存在于libuv引擎中。

### 浏览器事件循环

#### 执行栈

参考文章：https://github.com/jingegebuguai/JavaScript-/blob/master/Js%E5%8F%98%E9%87%8F%E5%AF%B9%E8%B1%A1%EF%BC%8C%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%EF%BC%8C%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%93%BE%E5%9F%BA%E7%A1%80.md

#### 事件循环的解释

js引擎遇到一个异步事件后并不会一直等待其返回结果，而是会将这个事件挂起，继续执行执行栈中的其他任务。当一个异步事件返回结果后，js会将这个事件加入与当前执行栈不同的另一个队列，我们称之为事件队列。被放入事件队列不会立刻执行其回调，而是等待当前执行栈中的所有任务都执行完毕， 主线程处于闲置状态时，主线程会去查找事件队列是否有任务。如果有，那么主线程会从中取出排在第一位的事件，并把这个事件对应的回调放入执行栈中，然后执行其中的同步代码...，如此反复，这样就形成了一个无限的循环。这就是这个过程被称为“事件循环（Event Loop）”的原因。

![](https://pic2.zhimg.com/80/v2-da078fa3eadf3db4bf455904ae06f84b_hd.jpg)


### macro task&micro task
异步任务之间互不相同， 他们的执行优先级也有区别，不同的异步任务分为两类：微任务和宏任务。

#### macro task
- setInterval()
- seTimeout()
- setImmediate()
- I/O

#### micro task
- process.nextTick()
- promise()
- Object.observe()


在一个事件循环中，异步事件返回结果后会被放到一个任务队列中。然而，根据这个异步事件的类型，这个事件实际上会被对应的宏任务队列或者微任务队列中去。并且在当前执行栈为空的时候，主线程会 查看微任务队列是否有事件存在。如果不存在，那么再去宏任务队列中取出一个事件并把对应的回到加入当前执行栈；如果存在，则会依次执行队列中事件对应的回调，直到微任务队列为空，然后去宏任务队列中取出最前面的一个事件，把对应的回调加入当前执行栈...如此反复，进入循环。

注意：当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行。




    console.log('start')

    const interval = setInterval(() => {  
      console.log('setInterval')
    }, 0)
    
    setTimeout(() => {  
      console.log('setTimeout 1')
      Promise.resolve()
          .then(() => {
            console.log('promise 3')
          })
          .then(() => {
            console.log('promise 4')
          })
          .then(() => {
            setTimeout(() => {
              console.log('setTimeout 2')
              Promise.resolve()
                  .then(() => {
                    console.log('promise 5')
                  })
                  .then(() => {
                    console.log('promise 6')
                  })
                  .then(() => {
                    clearInterval(interval)
                  })
            }, 0)
          })
    }, 0)
    
    Promise.resolve()
        .then(() => {  
            console.log('promise 1')
        })
        .then(() => {
            console.log('promise 2')
        })
        
执行结果如下
        
    start
    promise 1
    promise 2
    setInterval
    setTimeout 1
    promise 3
    promise 4
    setInterval
    setTimeout 2
    promise 5
    promise 6

### 事件循环流程分析

下面看一段代码，然后在分析具体的浏览器事件循环过程


    console.log(1)

    setTimeout(() => {
    	console.log(6)
    }, 1000)
    
    setTimeout(() => {
    	console.log(5)
    })
    new Promise((resolve, reject) => {
    	console.log(2)
    	resolve(4)
    }).then((value) => {
    	console.log(value)
    }) 
    
    console.log(3)
    
1、 第一步main()函数执行上下文，进入ECS(执行上下文栈)中；

2、执行console.log(1), 进栈，console.log没有回调，只是一个普通webkit内核支持方法，立即执行，输出1，出栈；

3、执行setTimeout(),将其进栈；

4、此时调用栈检查出setTimeout()是webApi中的api，所以立即出栈，将延迟执行的函数交给浏览器timer模块处理;

5、timer模块在一旁处理延迟执行的函数，此时执行引擎继续往下执行setTimeout(() => console.log(5)),注意这里虽然没有延迟处理，但是它依然会先进栈，然后出栈，将console.log(5)加入宏任务事件队列；

6、紧接着执行引擎将promise加入webapi，并将console.log(2)进栈，将resolve(4)回调事件加入到微任务事件队列；

7、输出2，log(2)出栈；

8、将log(3)进栈，输出3，出栈；

9、此时ECS为空，先查看微任务队列是否有事件，取出resolve(4),进栈，执行log(4),输出4，出栈；

10、此时微任务队列为空，查看宏任务队列，取出log(5),进栈，输出5，出栈；

11、当timer模块执行完成，将其加入到宏任务队列后，此时task执行完毕，将log（6）进栈，输出6，出栈。



### Node.js的事件循环机制
    
    
       ┌───────────────────────┐
    ┌─>│        timers         │
    │  └──────────┬────────────┘
    │  ┌──────────┴────────────┐
    │  │     I/O callbacks     │
    │  └──────────┬────────────┘
    │  ┌──────────┴────────────┐
    │  │     idle, prepare     │
    │  └──────────┬────────────┘      ┌───────────────┐
    │  ┌──────────┴────────────┐      │   incoming:   │
    │  │         poll          │<─────┤  connections, │
    │  └──────────┬────────────┘      │   data, etc.  │
    │  ┌──────────┴────────────┐      └───────────────┘
    │  │        check          │
    │  └──────────┬────────────┘
    │  ┌──────────┴────────────┐
    └──┤    close callbacks    │
       └───────────────────────┘


#### 执行顺序

外部输入数据-->轮询阶段(poll)-->检查阶段(check)-->关闭事件回调阶段(close callback)-->定时器检测阶段(timer)-->I/O事件回调阶段(I/O callbacks)-->闲置阶段(idle, prepare)-->轮询阶段...

#### poll
先查看poll queue中是否有事件，有任务就按先进先出的顺序依次执行回调。 当queue为空时，会检查是否有setImmediate()的callback，如果有就进入check阶段执行这些callback。但同时也会检查是否有到期的timer，如果有，就把这些到期的timer的callback按照调用顺序放到timer queue中，之后循环会进入timer阶段执行queue中的 callback。 这两者的顺序是不固定的，收到代码运行的环境的影响。如果两者的queue都是空的，那么loop会在poll阶段停留，直到有一个i/o事件返回，循环会进入i/o callback阶段并立即执行这个事件的callback。

#### check

check阶段专门用来执行setImmediate()方法的回调，当poll阶段进入空闲状态，并且setImmediate queue中有callback时，事件循环进入这个阶段。

#### close
当一个socket连接或者一个handle被突然关闭时（例如调用了socket.destroy()方法），close事件会被发送到这个阶段执行回调。否则事件会用process.nextTick（）方法发送出去。

#### timer

这个阶段以先进先出的方式执行所有到期的timer加入timer队列里的callback，一个timer callback指得是一个通过setTimeout或者setInterval函数设置的回调函数。

#### I/O
如上文所言，这个阶段主要执行大部分I/O事件的回调，包括一些为操作系统执行的回调。例如一个TCP连接生错误时，系统需要执行回调来获得这个错误的报告。

#### idel,prepare
仅node内部使用;


#### nextTick&setImmediate
process.nextTick 不属于事件循环的任何一个阶段，它属于该阶段与下阶段之间的过渡, 即本阶段执行结束, 进入下一个阶段前, 所要执行的回调。有给人一种插队的感觉.

setImmediate的回调处于check阶段, 当poll阶段的队列为空, 且check阶段的事件队列存在的时候，切换到check阶段执行.

#### setTimeout()和setImmediate()

    setTimeout(() => {
        console.log('timeout');
    }, 0);
    
    setImmediate(() => {
        console.log('immediate');
    });
    
这段神奇的代码执行顺序是不确定的。

这取决于这段代码的运行环境。运行环境中的各种复杂的情况会导致在同步队列里两个方法的顺序随机决定。但是，在一种情况下可以准确判断两个方法回调的执行顺序，那就是在一个I/O事件的回调中。

    const fs = require('fs');
    
    fs.readFile(__filename, () => {
        setTimeout(() => {
            console.log('timeout');
        }, 0);
        setImmediate(() => {
            console.log('immediate');
        });
    });
    
这段代码的结果是固定的。

因为在I/O事件的回调中，setImmediate方法的回调永远在timer的回调前执行。

