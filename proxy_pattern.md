## 代理模式（proxy_pattern）
### 1.定义
代理模式是为一个对象提供一个代替品或占位符，以便控制对这个对象的访问。

生活中其实有很多使用了代理概念的场景。比如明星都有经纪人，可以为他们筛选出一些好的演出和剧本；公司的高管一般也会有秘书来帮忙过滤掉一些不重要的信息。

### 2.保护代理
保护代理用于**控制不同权限**的对象对目标的访问。

场景：小明征婚，将自己的择偶条件交给了婚姻介绍所，婚姻介绍所会替他筛选合适的人。
```js
const xiaoming={
  addALady:(lady)=>{
    console.log(`have a date with ${lady.name}`);
  }
}

const laobao={
  addALady:(lady)=>{
    //帮助小明筛选符合条件的候选人
    if(lady.age>=0 && lady.age<30){
      xiaoming.add(lady);
    }else{
      console.log('sorry');
    }
  }
}

const xiaohong = {age:26,name:'xiaohong'};
laobao.addALady(xiaohong);
```  

### 3.虚拟代理
虚拟代理会将一些开销很大的操作延迟到正在需要的时候才做，而且是由代理对象来做。

#### 虚拟代理实现图片的预加载
```js
const myImage=(function(){
  const imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return function(src){
    imgNode.src = src;
  }
})();

const imageProxy=(function(){
  const img = new Image;
  img.onload=function(){
    myImage(this.src);
  }
  return function(src){
    myImage('loading.gif');
    img.src = src; //开始下载真正的图片
  }
})();

imageProxy('flower.jpg');
```  

### 4.代理的意义
#### 单一职责原理
单一职责原理提倡的是，**一个类应该只有一个引起它变化的原因**。如果一个对象承担了多个职责，那么就意味着引起它变化的原因也会变多。面向对象设计鼓励将行为分布到细粒度的对象中，如果一个对象的职责过多，会导致职责之间很容易出现耦合。

代理的意义是在于在**不影响目标对象的情况下，通过增加代理对象附加逻辑去控制最终的输出**。对于目标对象和使用者来说，这些逻辑都是透明的（**黑盒**），它们不需要关心是否有代理层，而只聚焦与自己需要的结果。

所以需要尽量保证代理对象暴露出来的接口和目标对象暴露的**接口是一致**的。这样就可以在任何使用目标对象的地方使用代理对象去代替，而不需要修改其他的东西。

### 5.代理的使用场景
#### 代理模式合并HTTP请求
在频繁网络请求的应用中，合并请求是一种常用的优化手段，可以减少网络请求的开销。
**
节流**
```js
const postAjax=(params)=>{
  console.log('发起请求，参数为'+params);
}

const proxyPostAjax=(function(){
  const cache=[];
  let timer;
  return function(param){
    cache.push(param);
    if(timer) return;
    timer = setTimeout(function(){
      postAjax(cache);
      clearTimeout(timer);
      timer = null;
      cache.length=0;
    },2000);
  }
})();

for(let i=0;i<100;i++){
  proxyPostAjax(i);
}
```  

#### 代理模式实现缓存代理
在js中，常常使用高阶函数函数的形式来实现代理模式，动态为目标对象增加一层代理。
```js
const createCacheProxy=(fn)=>{
  const cache={};
  return (arr)=>{
    if(cache[arr]){
      return cache[arr];
    }else{
      return cache[arr] = fn(arr);
    }
  }
}

const add=(arr)=>{
  return arr.reduce((count,n)=>{
    return count+n;
  },0);
}

const proxyAdd = createCacheProxy(add);
proxyAdd([1,2,3]); //6
proxyAdd([1,2,3]); //6
```  

### 5.小结
虽然代理模式很有用，但是我们在写逻辑的时候，不需要预先考虑去使用这种模式，等出现了不方便直接访问某个对象的时候，我们再用代理模式不迟。












