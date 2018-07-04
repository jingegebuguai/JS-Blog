### typeof

这是最常见的类型判断方法，估计很多人也只会这一个方法。
    
    console.log(typeof('huihui')) // string

typeof 是一元操作符，放在其单个操作数的前面，操作数可以是任意类型。返回值为表示操作数类型的一个字符串。

ES6的Js有七种数据类型，分别为：

    Undefined、Null、Boolean、Number、String、Object、Symbol
    
它们所对应的结果分别是undefined、object、boolean、number、string、object、symbol




    typeof 'huihui' //string
    typeof 1    //number
    typeof true //boolean
    typeof undefined    //undefined
    typeof null //object
    typeof []   //object
    let s = Symbol()
    typeof s    //symbol
    
这里注意null和Object返回的都是object

    let a = function() {}
    let b = new Function()
    let date = new Date()
    let array = new Array()
    console.log(typeof a)   //function
    console.log(typeof b)   //function
    console.log(typeof date)    //object
    console.log(typeof array)   //object
    
Object 下还有很多细分的类型呐，如 Array、Function、Date、RegExp、Error 等。他们的typeof结果都是object，区别就成了一个问题。

### Object.prototype.toString
具体作用如下：

- 如果 this 值是 undefined，就返回 [object Undefined]
- 如果 this 的值是 null，就返回 [object Null]
- 让 O 成为 ToObject(this) 的结果
- 让 class 成为 O 的内部属性 [[Class]] 的值
- 最后返回由 "[object " 和 class 和 "]" 三个部分组成的字符串
    
这里的class指的是对象的内部属性。
    
    console.log(Object.prototype.toString.call(undefined))  //[object Undefined]
    let date = new Date()
    let error = new Error()
    console.log(Object.prototype.toString.call(date))   //[object Date]
    console.log(Object.prototype.toString.call(error))  //[object Error]


封装的类型判断api

    var class2type = {};

    "Boolean Number String Function Array Date RegExp Object Error".split(" ").map(function(item, index) {
        class2type["[object " + item + "]"] = item.toLowerCase();
    })
    
    function type(obj) {
        if (obj == null) {
            return obj + "";
        }
        return typeof obj === "object" || typeof obj === "function" ?
            class2type[Object.prototype.toString.call(obj)] || "object" :
            typeof obj;
    }
    
在 IE6 中，null 和 undefined 会被 Object.prototype.toString 识别成 [object Object]！


    function isFunction(obj) {
        return type(obj) === "function";
    }
