## 单例模式
### 定义
保证一个类**仅有一个实例**，并提供一个访问它的全局访问点。

### 使用场景
线程池，全局缓存和浏览器的window对象都是采用的单例模式，共用一个实例。

### 1.单例模式-不透明模式
```js
function SingletonUser(name){
  this.name = name;
  this.instance = null;
}
//实例方法
SingletonUser.prototype.getName=()=>{
 console.log(this.name);
}
//类方法
SingletonUser.getInstance=function(name){
  if(!this.instance){
    this.instance = new SingletonUser(name);
  }
  return this.instance;
}

const a = SingletonUser.getInstance('user1');
const b = SingletonUser.getInstance('user2');

assert(a===b); //断言
```  
这种实现方式相对简单，但是问题也十分明显，它需要使用其他的方式[getInstance]实例化这个类才能达到单例的效果。违背了类的通用模式，提高了使用成本，使用者在使用之前得判断这是不是一个单例类。

### 2.单例模式-透明模式
```js
//立即执行函数
let CreateSingleUser = (function(){
  let instance;
  function CreateSingleUser(name){
    if(!instance){
      instance = this.create(name);
    }
    return instance;
  }
  
  CreateSingleUser.prototype.create=function(name){
    this.name = name;
    return this;
  }
  return CreateSingleUser;
})()

const a = new CreateSingleUser('user1');
const b = new CreateSingleUser('user2');

assert(a===b);
```  
这种方式虽然保证了透明性，但是仍有以下缺点：
1. 其中使用了立即函数和闭包，读起来很不舒服
2. CreateSingleUser类中负责了两件事，一是创建实例，而是保证只实例化一次，违背了**单一职责原则**。如果有一天需要创建很多实例，或者这两种方式并存的时候，都难以解决。

### 3.惰性单例-使用的时候才会初始化
```js
//保证单例的逻辑
function singletonProxy(fn){
  let instance = null;
  //使用闭包
  return function(){
    return instance || (instance = fn.apply(this,arguments));
  }
}

//创建实例的逻辑
const createSingleDiv = singletonProxy(()=>{
  let div = document.createElement('div');
  div.innerHTML = 'hehe';
  document.body.appendChild(div);
  return div;
})

const a = createSingleDiv();
const b = createSingleDiv();

assert(a === b);
```  
这种实现方式是js中通用的单例模式实现方式，遵守单一职责规则，而且js中是没有类的概念的，类都是用函数来实现的。所以用函数来实现更符合js的编程习惯。




