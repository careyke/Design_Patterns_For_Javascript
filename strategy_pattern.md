### 策略模式
定义一系列的算法，把它们一个个封装起来，并且使他们可以相互替换。

#### 1.没有使用策略模式的代码
```js
//这是一段根据绩效计算年终奖的代码
const getMoney=(level,salary)=>{
  if(level === 's'){
    return salary * 5;
  }
  if(level === 'a'){
    return salary * 4;
  }
  if(level === 'b'){
    return salary * 3;
  }
}

console.log(getMoney('s',2000));  //10000
```  
可以看到这段代码存在很明显的问题：
- if-else语句太多，代码像拉面一样。
- getMoney方法缺乏弹性，如果新增一种类型，需要去修改函数的内部实现，违反开放-封闭原则。

#### 2.使用策略模式进行改进
策略模式的目的是将算法的使用和算法的实现分离开来。
一个基于策略模式的程序至少有两个部分组成：
1. 一组策略类——策略类中封装了具体的算法，负责算法的实现部分。
2. 环境类Context——Context负责对接用户的请求，然后委托给对应的策略。要做到这一点，说用Context中必须要有策略类的引用。

实现代码：
```js
const strategies = {
  's':(salary) => salary*5,
  'a':(salary) => salary*4,
  'b':(salary) => salary*3
}

//Context
const getMoney=(level,salary)=>{
  return strategies[level](salary);
}

console.log(getMoney('s',2000));  //10000
```  

#### 3.策略模式的实际使用场景
策略模式看起来就是用来封装算法的，然后在合适的时候使用合适的策略解决问题。但是在实际的应用中，我们将算法的定义扩大话，使用策略模式封装一些业务规则。只用这些业务规则的目标一致，就可以替换使用。

##### 常用的场景——表单验证
###### 1.最简单的表单验证
```js
<form action='' id='registerForm' method='post'>
  <input type='text' name='userName'/>
  <input type='password' name='password'/>
  <input type='number' name='phone'/>
  <button>submit</button>
</form>

//script
const registerForm = document.getElementById('registerForm');
registerForm.onsubmit=()=>{
  if(registerForm.userName.value === ''){
    console.log('用户名不能为空');
    return false;
  }
  if(registerForm.password.value.length < 6){
    console.log('密码长度不少于6位');
    return false;
  }
  if(!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.password.phone)){
    console.log('手机号码格式不正确');
    return false;
  }
}
```  
这就和开始没有用策略模式的年终奖计算方法一样，代码难以扩展也难以维护，也不够优雅。

###### 2.策略模式实现表达验证
```js
const strategies={
  isNonEmpty:(value,errorMsg)=>{
    if(value === ''){
      return errorMsg;
    }
  },
  minLength:(value,length,errorMsg)=>{
    if(value.length<length){
      return errorMsg;
    }
  },
  isMobile:(value,errorMsg)=>{
    if(!/(^1[3|5|8][0-9]{9}$)/.test(value)){
      return errorMsg;
    }
  }
}

class Validator{
  constructor(){
    this.cache=[];
  }
  add=(dom,rule,errorMsg)=>{
    const arr=rule.split(':');
    this.cache.push(()=>{
      const strategy = arr.shift();
      arr.unshift(dom.value);
      arr.push(errorMsg);
      return strategies[strategy].apply(dom,arr);
    })
  }
  start=()=>{
    this.cache.forEach((validFun)=>{
      const msg = validFun();
      if(msg){
        return msg;
      }
    })
  }
}

//Context
const registerForm = document.getElementById('registerForm');
const validataFunc=()=>{
  const validator = new Validator();
  validator.add(registerForm.userName,'isNonEmpty','用户名不能为空');
  validator.add(registerForm.password,'minLength:6','密码长度不能少于6位');
  validator.add(registerForm.phone,'isMobile','手机号码不符合规范');
  return validator.start();
}
registerForm.onsubmit=()=>{
  const errorMsg = validataFunc();
  if(errorMsg){
    console.log(errorMsg);
    return false;
  }
}
```  
在使用策略模式重构之后：
1. 代码的复用性增强，这些验证的逻辑可以被用在任何表单而不需要重写。
2. 修改某一个验证规则的还是很方便，通过配置的形式就可以完成。

#### 4.策略模式的优缺点
##### 优点
1. 策略模式利用组合、委托和多态的思想，可以有效的避免写多重if-else的拉面代码
2. 策略模式完美支持开放-封闭原则，将算法实现独立封装在strategy中，易于切换，易于扩展。
3. 策略模式利用组合和委托的方式使Context具有执行strategy的能力，这也是继承的一种替代方案。

##### 缺点
1. 策略模式会增加一个策略类，但是实际上这个是一种好的方式
2. 使用策略模式就必须要了解所有的strategy，这个有点违反最少知识原则。












