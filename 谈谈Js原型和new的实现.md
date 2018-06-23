new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一

一般而言，实例对象的简单实现方法如下所示：

    let PRI_NUM = 999;

    function testNew(name, age) {
    	this.name = name;
    	this.age = age;
    
    	this.num = PRI_NUM;
    }
    
    testNew.prototype.habit = 'study';
    
    testNew.prototype.doSomething = function() {
    	console.log('I love' + this.name);
    }
    
    let test = new testNew('huihui', 18);
    console.log(test.num)
    
实例test实现的功能点包括访问testNew函数对象的属性和函数对象原型的属性。

js继承的思想是通过原型将一个引用类型继承另一个引用类型的属性和方法。


prototype和__proto__其实很简单，一般而言实例对象只有__proto__属性,函数对象既有prototype属性，也有__proto__属性；原型对象有constructor和__proto__属性。具体的关系如下所示：

![image](https://upload-images.jianshu.io/upload_images/4740192-f3276363e4b5341b.jpg)

有关系图可知，实例对象的__proto__和构造函数的prototype相同，即

    test.__proto__ == testNew.prototype

沿用上面代码实例：
    
    console.log(test.__proto__, test.__proto__ == testNew.prototype)
    console.log(testNew.prototype.__proto__, Object.prototype == testNew.prototype.__proto__)
    console.log(Function.prototype, testNew.__proto__ == Function.prototype)
    console.log(Function.prototype.__proto__)
    
执行结果如下所示：

    testNew { habit: 'study', doSomething: [Function] } true
    {} true
    [Function] true
    {}
    
从上面可以发现testNew这个函数对象既有__proto__属性，也有prototype属性，而函数对象的__proto__指向的是Function对象。并且可以发现如果一个实例对象或者函数对象如果调用__proto__属性，可以指向它的一个原型对象，默认情况下，testNew.prototype又会作为Object的实例对象，原型对象只有__proto__属性，所以可以一直调用__proto__，直至为null。

下面说说new的实现方式：

我们使用一个函数对象newTest()来模拟new的实现

    let PRI_NUM = 999;

    function testNew(name, age) {
        this.name = name;
        this.age = age;

        this.num = PRI_NUM;
    }
    
    testNew.prototype.habit = 'study';
    
    testNew.prototype.doSomething = function() {
        console.log('I love ' + this.name);
    }
    
    function newTestFactory() {
        let obj = new Object();
        let Constructor = Array.prototype.shift.call(arguments);
        obj.__proto__ = Constructor.prototype;
        Constructor.apply(obj, arguments);
        return obj;
    }
    
    let test = newTestFactory(testNew, 'huihui', 18);
    
    console.log(test.name)  //huihui
    test.doSomething()      //I love huihui
    
    
newTestFactory是new实现的主要核心，主要原理如下：

1、用new Object() 的方式新建了一个对象 obj

2、利用[].shift和call方法取出参数中的构造函数

3、将 obj 的原型指向构造函数，这样 obj 就可以访问到构造函数原型中的属性

4、使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性

5、返回 obj

当然，这边的第3，第4步肯定是有其他改造方式的，比如属性继承方面，我们可以使用原型式继承来实现，如下所示：

    function beget(s) {
        let F = function(){};
        F.prototype = s;
        return new F();
    }
    
    function newTestFactory() {
        let obj = new Object();
        let Constructor = Array.prototype.shift.call(arguments);
        obj = beget(Constructor.prototype)
        Constructor.apply(obj, arguments);
        return typeof ret === 'object' ? ret : obj;
    }
    
使用原型式继承构造函数原型中方法和属性的好处就是，得到一个纯洁的对象。这里纯洁的意思就是没有实例属性。

ES5提供了Object.create()函数，内部就是原型式继承。

所以还可以继续改进代码为：

    function newTestFactory() {
        let obj = new Object();
        let Constructor = Array.prototype.shift.call(arguments);
        obj = Object.create(Constructor.prototype)
        Constructor.apply(obj, arguments);
        return typeof ret === 'object' ? ret : obj;
    }
    

    
    













