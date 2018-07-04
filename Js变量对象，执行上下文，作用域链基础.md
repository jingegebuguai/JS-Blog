### Js作用域

作用域是指程序源代码中定义变量的区域。

作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。

JavaScript 采用词法作用域(lexical scoping)，也就是静态作用域。

    var value = 1;

    function foo() {
        console.log(value);
    }
    
    function bar() {
        var value = 2;
        foo();
    }
    
    bar();  //1
    
执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。


    var scope = "global scope";
    function checkscope(){
        var scope = "local scope";
        function f(){
            return scope;
        }
        return f();
    }
    checkscope();   //local scope
    
JavaScript采用的是词法作用域，函数的作用域基于函数创建的位置。

### EC（执行上下文，Execution Context）
当控制器可以执行js代码的时候，控制器进去一个执行上下文。

执行上下文包括

- 全局级别的代码，默认的代码运行环境，一旦代码加载，最先引入这个代码
- 函数级别的代码，执行一个函数时，运行函数代码
- eval函数代码

EC建立分为两个阶段：进入执行上下文和执行阶段。

1.进入上下文阶段：发生在函数调用时，但是在执行具体代码之前（比如，对函数参数进行具体化之前）

2.执行代码阶段：变量赋值，函数引用，执行其他代码。

    EC={
        VO:{/* 函数中的arguments对象, 参数, 内部的变量以及函数声明 */},
        this:{},
        Scope:{ /* VO以及所有父执行上下文中的VO */}
    }


### ECS（执行上下文栈，Execution Context Stack）

一系列的执行上下文在逻辑上形成ECS，栈底总是全局上下文，栈顶是当前执行上下文。

我们可以用数组的形式来表示环境栈:

    ECS = [局部EC， 全局EC]

当javascript代码文件被浏览器载入后，默认最先进入的是一个全局的执行上下文。当在全局上下文中调用执行一个函数时，程序流就进入该被调用函数内，此时引擎就会为该函数创建一个新的执行上下文，并且将其压入到执行上下文堆栈的顶部。浏览器总是执行当前在堆栈顶部的上下文，一旦执行完毕，该上下文就会从堆栈顶部被弹出，然后，进入其下的上下文执行代码。这样，堆栈中的上下文就会被依次执行并且弹出堆栈，直到回到全局的上下文。

### VO（变量对象，Variable Object）
每一个VO对应一个变量对象VO，EC中定义的所有变量和函数都存放在其对应的VO中

VO分为：

- 全局上下文VO
- 函数上下文VO

VO：{
      // 上下文中的数据 (变量声明（var）, 函数声明（FD), 函数形参（function arguments）)
}


### AO（活动对象，Active Object）
在函数的执行上下文中，VO是不能直接访问的。

就是当EC环境为函数时，我们访问的是AO，而不是VO。

    VO(functionContext) === AO

AO是进入函数的执行上下文时创建的，并为该对象初始化一个arguments属性，该属性的值为Arguments对象

    AO = {
        arguments: {
            callee: ,
            length: ,
            properties-indexes
        }
    }

这里函数声明形式为

    function f() {
        
    }
    
### 执行上下文栈示例

    function fun3() {
        console.log('test')
    }
    function fun2() {
        return fun3()
    }
    function fun1() {
        return fun2()
    }
    fun1()
    
当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。

    // fun1()
    ECStack.push(<fun1> functionContext);
    
    // fun1中竟然调用了fun2，还要创建fun2的执行上下文
    ECStack.push(<fun2> functionContext);
    
    // 擦，fun2还调用了fun3！
    ECStack.push(<fun3> functionContext);
    
    // fun3执行完毕
    ECStack.pop();
    
    // fun2执行完毕
    ECStack.pop();
    
    // fun1执行完毕
    ECStack.pop();
        
下面代码做个比较：

    var scope = "global scope";
    function checkscope(){
        var scope = "local scope";
        function f(){
            return scope;
        }
        return f();
    }
    checkscope();
    
    ECStack.push(<checkscope> functionContext);
    ECStack.push(<f> functionContext);
    ECStack.pop();
    ECStack.pop();
        
    var scope = "global scope";
    function checkscope(){
        var scope = "local scope";
        function f(){
            return scope;
        }
        return f;
    }
    checkscope()(); //这里先执行checkscope函数，结束后，调用f——>f()
    
    ECStack.push(<checkscope> functionContext);
    ECStack.pop();
    ECStack.push(<f> functionContext);
    ECStack.pop();
    
    
### 作用域链

当查找变量时，会从当前上下文的变量对象中查找，如果没有找到，就会从父级（词法）上下文变量对象查找，一直找到全局上下文，也就是全局对象。

函数有一个内部属性 [[scope]]，当函数创建的时候，就会保存所有父变量对象到其中，你可以理解 [[scope]] 就是所有父变量对象的层级链。

    funtion func1() {
        function func2 {
            ...
        }
    }
    
    func1.[[scope]] = [
        globalContext.VO
    ]
    
    func2.[[scope]] = [
        booContext.AO
        globalContext.VO
    ]
    
函数激活：

当函数激活时，进入函数上下文，创建 VO/AO 后，就会将活动对象添加到作用链的前端。

这时候执行上下文的作用域链，我们命名为 Scope：

    Scope = [AO].concat([[Scope]]);
    
至此，作用域链创建完毕。

参考链接： https://github.com/mqyqingfeng/Blog/issues/6


### 变量对象


#### 全局对象
全局上下文一般用AO表示。

全局对象是预定义的对象，作为 JavaScript 的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。

1、顶层js代码中，可使用this引用全局对象

    console.og(this)    //{}
    
2、全局对象时Object构造函数实例化的一个对象
    
    console.log(typeof(this))
    console.log(this instanceof Object)
    
3、预定义若干函数和属性

    console.log(Math.random())
    
4、作为全局变量的宿主

    var a = 1
    this.a

5、客户端js中，全局对象有window属性指向自身

    var a = 1
    window.a
    this.window.b = 2
    this.b
    
#### 函数上下文

在函数上下文中，我们用活动对象(activation object, AO)来表示变量对象。

只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 activation object 呐，而只有被激活的变量对象，也就是活动对象上的各种属性才能被访问。

活动对象是在进入函数上下文时刻被创建的，它通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。

#### 执行过程

- 进入执行上下文
- 代码执行

##### 进入执行上下文

当进入执行上下文时，这时候还没有执行代码，

变量对象会包括：

1、 函数的所有形参 (如果是函数上下文)

- 由名称和对应值组成的一个变量对象的属性被创建
- 没有实参，属性值设为 undefined

2、函数声明

- 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建
- 如果变量对象已经存在相同名称的属性，则完全替换这个属性

3、变量声明

- 由名称和对应值（undefined）组成一个变量对象的属性被创建；
- 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

举个例子：

    function foo(a) {
      var b = 2;
      function c() {}
      var d = function() {};
    
      b = 3;
    
    }
    
    foo(1);
    
AO:
    
    AO = {
        arguments: {
            0: 1,
            length: 1
        },
        a: 1,
        b: undefined,
        c: reference to function c(){},
        d: undefined
    }
    
##### 代码执行

    AO = {
        arguments: {
            0: 1,
            length: 1
        },
        a: 1,
        b: 3,
        c: reference to function c(){},
        d: reference to FunctionExpression "d"
    }

- 全局上下文的变量对象初始化是全局对象

- 函数上下文的变量对象初始化只包括 Arguments 对象

- 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值

- 在代码执行阶段，会再次修改变量对象的属性值