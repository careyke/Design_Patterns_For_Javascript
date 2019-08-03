## 发布-订阅模式
### 定义
发布订阅又称观察者模式，它定义了对象间一对多的依赖关系，**当对象的状态发生变化的时候，所有依赖于它的对象都将得到通知**。

### 使用场景
发布订阅模式的使用场景很丰富，DOM事件，现代的MVC,MVVM框架中都有运用到这种设计模式。

### 发布订阅模式的通用设计思路
1. 事件缓存对象 —— 通常会定义一个**全局**的对象，用来缓存事件以及提供发布、订阅和删除事件的接口。建议是一个项目只需要定义一个发布订阅模式的消息模块。
2. 事件订阅者 —— 调用订阅接口，缓存回调事件处理函数，不关注消息的来源。
3. 事件发布者 —— 调用发布接口，发布消息，不关注消息的去处。

### eventBus简单实现代码
eventBus就是典型的发布订阅的模式，而且使用也比较广泛。
```js
const EventBus = (function() {
  const listenerCache = {};
  const subscribe = (topic, channel, method, callback) => {
    const key = `${topic}_${channel}_${method}`;
    if (!listenerCache[key]) {
      listenerCache[key] = [];
    }
    listenerCache[key].push(callback);
    return {
      remove: () => {
        listenerCache[key] = listenerCache[key].filter(cb => cb !== callback);
      }
    };
  };
  const publish = (topic, channel, method, params) => {
    const key = `${topic}_${channel}_${method}`;
    listenerCache[key].forEach(callback => {
      //这里执行回调函数的时候一定要catch错误，防止一个中断，全部挂掉
      try {
        callback(params);
      } catch (error) {
        console.error(error);
      }
    });
  };
  const unsubcribe = (topic, channel, method, callback) => {
    const key = `${topic}_${channel}_${method}`;
    if (callback === undefined) {
      listenerCache[key] = [];
    } else {
      listenerCache[key] = listenerCache[key].filter(cb => cb !== callback);
    }
  };

  return {
    sub: subscribe,
    pub: publish,
    unsub: unsubcribe
  };
})();
```  
需要注意的点：
1. eventBus一般是一种即时通讯的机制，调用处理函数的时候是同步调用,异步会导致接收消息延时。
2. 在调用回调函数的时候，需要使用try...catch。防止由于个别失误导致整个程序中断。
3. 在程序销毁的时候，记得先销毁订阅，避免出现内存泄漏

### 发布订阅的优势与劣势
#### 优势
1. 降低对象之间的耦合程度，降低代码复杂度
2. 可以使任意两个对象之间进行通讯，这个运用在React这种强数据流向顺序的框架中是可以解决很多问题的。（有一些无需保存状态的数据之间的交互可以直接使用eventBus）

#### 劣势
1. 需要额外的内部去缓存事件回调列表
2. 发布订阅的模式虽然可以解耦对象，但是凡事**过犹不及**，如果过度依赖这种模式，会将对象之间的关联关系完全隐藏在背后，会导致代码晦涩难懂。

### 离线重传——后订阅先发布
可能在极少数的场景下，可以能会出现先发布后订阅的情况，需要在下次订阅的时候，立马执行前面发布的消息。

实现思路：可以先建立一个对象来用**存储未订阅已发布的消息**，然后如果一个消息先发布但是没有订阅就先存在对象中，等待某一次有相同事件被订阅的时候，立马将所有消息全部重新发布一遍，然后清空暂存队列。

脑补代码























