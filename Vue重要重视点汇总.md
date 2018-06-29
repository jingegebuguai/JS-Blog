#### MVVM

MVVM为Model-View-ViewModel的缩写。是一种设计思想。

- M代表数据模型，在Model可以定义数据修改和操作的业务逻辑；V代表视图，即UI组件，负责将数据模型展示成UI展示出来；ViewModel是同步View和Model的对象。
- View 和 Model 之间并没有直接的联系，而是通过ViewModel进行交互，Model 和 ViewModel 之间的交互是双向的， 因此View 数据的变化会同步到Model中，而Model 数据的变化也会立即反应到View 上。
- ViewModel 通过双向数据绑定把 View 层和 Model 层连接了起来，而View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理。

![](https://cdn-images-1.medium.com/max/406/1*4ucjoZNxL-M9iz5ZvBV0ig.png)

#### Vue的好处
面试官可能常问你，为什么你要用vue？我只能说答案有以下几点：

- 低耦合。试图View可以独立于Model开发，View的改变不影响Model，Model的改变也不影响View；
- 可重用性。ViewModel里可以存放多个视图逻辑，其他的View可重用这些试图逻辑；
- 独立开发。前后端分离的思想，开发侧重与业务逻辑和数据开发，设计侧重于页面设计；
- 可测试。可针对ViewModel进行测试；


#### Vue钩子函数
![](https://segmentfault.com/img/bVEo3w?w=1200&h=2800)

![](https://segmentfault.com/img/bVEs9x?w=847&h=572)


#### v-if&v-show
简单来说

- 使用v-if时，如果值为false，页面将不会有这个html标签生成。
- 使用v-show时，无论值是否为true，false，html元素都是存在的，只是css的display显示或隐藏。

所以在性能方面，v-if会添加删除页面的DOM元素，所以v-if会有更高的切换消耗；v-show只是切换css的display属性，所以有比较高的初始化渲染消耗，后续的消耗比较低。

如果在频繁切换的场景中v-show更好。


#### 组件之间数据传递

一个组件的data必须是一个函数

    data() {
        return {
            args: ''
        }
    }

##### 父组件向子组件传值
通过props向子组件传递数据
    
    //父组件
    <template>
        <Main :obj="data"></Main>
    </template>
    <script>
        import Main form "./main"
        
        exprot default{
            name:"parent",
            data(){
                return {
                    data:"我要向子组件传递数据"
                }
            },
            components:{
                Main
            }
        }
    </script>
    
    //子组件
    <template>
        <div>{{data}}</div>
    </template>
    <script>
        exprot default{
            name:"son",
            props: {
                data: {
                    type: String,
                    reuqired: true
                }
            }
        }
    </script>
    
##### 子組件向父組件
通过事件触发传递数据

    //子组件
    <template>
       <div @click="events"></div>
    </template>
    <script>
        //引入子组件
        import Main form "./main"
        
        exprot default{
            data{
                return {
                    val: 'Fuck You'
                }
            }
            methods:{
                events:function(){
                    this.$emit('childEvent', this.val)       
                }
            }
        }
    </script>
    
    //父组件
    <template>
       <Child @childEvent="FatherEvent"></Child>
    </template>
    <script>
        import Child from './child'
        export default{
            data() {
                return {
                    
                }
            },
            methods: {
                FatherEvent(val) {
                    console.log(val)
                }
            }
        }
    </script>

##### 

#### $refs&nextTick
##### $refs

在Vue的模板中，我们可以在模板中的任何元素中添加ref属性，这样就可以在Vue实例中引用这些元素。更具体地说，可以访问DOM元素。

    <botton ref="myButton" @chick="chlickButton"></button>
    
在js中调用ref属性

    console.log(this.$refs)

这将打印出一个对象，包含了所有ref属性的引用

![](http://chuantu.biz/t6/336/1530250750x-1404755624.png)

这里对象的键值和ref属性指定的名称name相同，其值为DOM元素，所以我们可以通过$refs访问DOM元素

    let _this = this
    Vue.nextTick(function(){
        _this.$refs.viewUser.getElementsByClassName('el-input')[0].children[0].focus()
    })
    
##### Vue.nextTick&vm.$nextTick

Vue 实现响应式并不是数据发生变化之后 DOM 立即变化，而是按一定的策略进行 DOM 的更新。

$nextTick 是在下次 DOM 更新循环结束之后执行延迟回调，在修改数据之后使用 $nextTick，则可以在回调中获取更新后的 DOM
    
    <div class="app">
      <div ref="msgDiv">{{msg}}</div>
      <div v-if="msg1">Message got outside $nextTick: {{msg1}}</div>
      <div v-if="msg2">Message got inside $nextTick: {{msg2}}</div>
      <div v-if="msg3">Message got outside $nextTick: {{msg3}}</div>
      <button @click="changeMsg">
        Change the Message
      </button>
    </div>
    
    new Vue({
      el: '.app',
      data: {
        msg: 'Hello Vue.',
        msg1: '',
        msg2: '',
        msg3: ''
      },
      methods: {
        changeMsg() {
          this.msg = "Hello world."
          this.msg1 = this.$refs.msgDiv.innerHTML
          this.$nextTick(() => {
            this.msg2 = this.$refs.msgDiv.innerHTML
          })
          this.msg3 = this.$refs.msgDiv.innerHTML
        }
      }
    })
    
点击后的效果如下：    
![](https://pic2.zhimg.com/80/v2-174878320d72cdf17c050c7b3c87b9c7_hd.jpg)
    
vue这么做是因为频繁的更新dom是特别耗费性能的，所以搞了一个批处理更新，把所有的update操作放到任务队列中，等主线程中执行栈的所有同步任务执行完毕，系统就会读取任务队列。


#### 路由跳转

    //声明式
    <router-link :to="index">
    
    //编程式
    router.push('index')

#### vuex
Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

![](https://pic2.zhimg.com/v2-be68719a9e63469fb846d7e1dec92b81_1200x500.jpg)

Action 类似于 mutation，不同在于：

Action 提交的是 mutation，而不是直接变更状态。
Action 可以包含任意异步操作。

这里内容比较多，推荐看文档

### vue原理

vue.js 采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。

#### defineProperty

Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

语法：

    Obeject.defineProperty(obj, prop, descriptor)
    
属性描述符：

对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。数据描述符是一个具有值的属性，该值可能是可写的，也可能不是可写的。存取描述符是由getter-setter函数对描述的属性。描述符必须是这两种形式之一；不能同时是两者。

数据描述符和存取描述符均具有以下可选键值：

 - configurable:当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认为 false。
 - enumerable: 当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。默认为 false。
 

数据描述符同时具有以下可选键值：

- value:该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。
- writable: 当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为 false。


存取描述符同时具有以下可选键值：

- get: 一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入this对象（由于继承关系，这里的this并不一定是定义该属性的对象）。
- set: 一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。


这里注意：

value和writable是数据描述符专有，而get和set是存取描述符专有

    let obj = {}
    //定义对象属性
    Object.defineProperty(obj, 'name', {
    	configurable: true,
    	enumerable: true,
    	writable: true,
    	value: 'huihui'
    })
    
    //存取对象属性（添加修改）
    let age = 18
    Object.defineProperty(obj, 'age', {
    	get() {
    		return age
    	},
    	set(newValue) {
    		age = newValue
    	},
    	enumerable: true,
    	configurable: true
    })
    
    obj.age = 20
    
    console.log(obj.name, obj.age)



#### 数据绑定流程

- 实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
- 实现一个指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
- 实现一个Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图
- mvvm入口函数，整合以上三者


![](https://segmentfault.com/img/bVBQYu)

 
详细内容参考：https://segmentfault.com/a/1190000006599500

### vue响应式原理
Vue通过设定对象属性的 setter/getter 方法来监听数据的变化，通过getter进行依赖收集，而每个setter方法就是一个观察者，在数据变更的时候通知订阅者更新视图。

    function observe(value, cb) {
        Object.keys(value).forEach((key) => defineReactive(value, key, value[key] , cb))
    }
    
    function defineReactive (obj, key, val, cb) {
        Object.defineProperty(obj, key, {
            enumerable: true,
            configurable: true,
            get: ()=>{
                /*....依赖收集等....*/
                /*Github:https://github.com/answershuto*/
            },
            set:newVal=> {
                cb();/*订阅者收到消息的回调*/
            }
        })
    }
    
    class Vue {
        constructor(options) {
            this._data = options.data;
            observe(this._data, options.render)
        }
    }
    
    let app = new Vue({
        el: '#app',
        data: {
            text: 'text',
            text2: 'text2'
        },
        render(){
            console.log("render");
        }
    })
 
### vue虚拟DOM

Vue.js将DOM抽象成一个以JavaScript对象为节点的虚拟DOM树，以VNode节点模拟真实DOM，可以对这颗抽象树进行创建节点、删除节点以及修改节点等操作，在这过程中都不需要操作真实DOM，只需要操作JavaScript对象后只对差异修改，相对于整块的innerHTML的粗暴式修改，大大提升了性能。修改以后经过diff算法得出一些需要修改的最小单位，再将这些小单位的视图进行更新。这样做减少了很多不需要的DOM操作，大大提高了性能。


#### 流程

- 用js对象模拟DOM树
- 比较两颗虚拟DOM树的差异
- 将差异应用到真正的DOM树上


    export default class VNode {
      tag: string | void;
      data: VNodeData | void;
      children: ?Array<VNode>;
      text: string | void;
      elm: Node | void;
      ns: string | void;
      context: Component | void; // rendered in this component's scope
      functionalContext: Component | void; // only for functional component root nodes
      key: string | number | void;
      componentOptions: VNodeComponentOptions | void;
      componentInstance: Component | void; // component instance
      parent: VNode | void; // component placeholder node
      raw: boolean; // contains raw HTML? (server only)
      isStatic: boolean; // hoisted static node
      isRootInsert: boolean; // necessary for enter transition check
      isComment: boolean; // empty comment placeholder?
      isCloned: boolean; // is a cloned node?
      isOnce: boolean; // is a v-once node?
    
      constructor (
        tag?: string,
        data?: VNodeData,
        children?: ?Array<VNode>,
        text?: string,
        elm?: Node,
        context?: Component,
        componentOptions?: VNodeComponentOptions
      ) {
        /*当前节点的标签名*/
        this.tag = tag
        /*当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息*/
        this.data = data
        /*当前节点的子节点，是一个数组*/
        this.children = children
        /*当前节点的文本*/
        this.text = text
        /*当前虚拟节点对应的真实dom节点*/
        this.elm = elm
        /*当前节点的名字空间*/
        this.ns = undefined
        /*编译作用域*/
        this.context = context
        /*函数化组件作用域*/
        this.functionalContext = undefined
        /*节点的key属性，被当作节点的标志，用以优化*/
        this.key = data && data.key
        /*组件的option选项*/
        this.componentOptions = componentOptions
        /*当前节点对应的组件的实例*/
        this.componentInstance = undefined
        /*当前节点的父节点*/
        this.parent = undefined
        /*简而言之就是是否为原生HTML或只是普通文本，innerHTML的时候为true，textContent的时候为false*/
        this.raw = false
        /*静态节点标志*/
        this.isStatic = false
        /*是否作为跟节点插入*/
        this.isRootInsert = true
        /*是否为注释节点*/
        this.isComment = false
        /*是否为克隆节点*/
        this.isCloned = false
        /*是否有v-once指令*/
        this.isOnce = false
      }
    
      // DEPRECATED: alias for componentInstance for backwards compat.
      /* istanbul ignore next */
      get child (): Component | void {
        return this.componentInstance
      }
    }
    
#### 渲染实例

    {
        tag: 'div'
        data: {
            class: 'test'
        },
        children: [
            {
                tag: 'span',
                data: {
                    class: 'demo'
                }
                text: 'hello,VNode'
            }
        ]
    }

渲染结果：

    <div class="test">
        <span class="demo">hello,VNode</span>
    </div>

更多内容参考：https://github.com/answershuto/learnVue/blob/master/docs/VNode%E8%8A%82%E7%82%B9.MarkDown
 
 


