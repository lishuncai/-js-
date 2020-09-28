# js设计模式 

### 相关概念

#### 多态

同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结 果。换句话说，给不同的对象发送同一个消息的时候，这些对象会根据这个消息分别给出不同的 反馈。 

-------

### 设计模式

#### 单例模式

《javascript设计模式与开发实践》
    
- 定义： 保证一个类仅有一个实例，并提供一个访问它的全局访问。
    
- 描述： 单例模式是一种常用的模式，有一些对象我们往往只需要一个，比如线程池、全局缓存、浏 览器中的 window 对象等。在 JavaScript开发中，单例模式的用途同样非常广泛。
    试想一下，当我 们单击登录按钮的时候，页面中会出现一个登录浮窗，而这个登录浮窗是唯一的，无论单击多少 次登录按钮，这个浮窗都只会被创建一次，那么这个登录浮窗就适合用单例模式来创建。

- 涉及相关：闭包、高阶

```
/* 常规 */

var createLoginLayer = function () {
    var div = document.createElement('div');
    div.innerHTML = '我是登录浮窗';
    div.style.display = 'none';
    document.body.appendChild(div);
    return div;
};

document.getElementById('loginBtn').onclick = function () {
    var loginLayer = createLoginLayer();
    loginLayer.style.display = 'block';
};

```

```
/* 用代理实现单例 */
var creatediv = function(html){
    this.html = html;
    this.init();
}
creatediv.prototype.init = function() {
    var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
}
var ProxyCreater = (function() {
    var instance;
    return function(html) {
        if (!instance) {
            instance = new creatediv(html);
        }
        return instance;
    }
})();
var a = new ProxyCreater('svent')
var b = new ProxyCreater('svent2')
```
```
/* 惰性、单一 */
/* em1. */
var Creater = function(html) {
    this.html = html;
    this.init();
}
Creater.prototype.init = function() {
    var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div)
}
var createrProxy = function(fn) {
    var instance;
    return function() {
        console.log(arguments);
        return instance || (instance = fn.apply(this, arguments))
    }
}
var result = createrProxy(function(html, id){
    return new Creater(html, id)
})
document.body.click = function() {
    result('divText', 1);
}

/* em2. */
var getSingle = function (fn) {
    var result;
    return function () {
        return result || (result = fn.apply(this, arguments));
    }
}
var bindEvent = getSingle(function () {
    document.getElementById('div1').onclick = function () {
        alert('click');
    }
    return true;
});
var render = function () {
    console.log('开始渲染列表');
    bindEvent();
};

render();
render();
render();
```

------

#### 策略模式

《javascript设计模式与开发实践》
    
- 定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。
    
- 描述：在程序设计中，我们也常常遇到类似的情况，要实现某一个功能有多种方案可以选择。比如 一个压缩文件的程序，既可以选择 zip算法，也可以选择 gzip算法。 这些算法灵活多样，而且可以随意互相替换。这种解决方案就是本章将要介绍的策略模式。

- 描述： 一个基于策略模式的程序至少由两部分组成。第一个部分是一组策略类，策略类封装了具体 的算法，并负责具体的计算过程。 第二个部分是环境类 Context，Context接受客户的请求，随后 把请求委托给某一个策略类。要做到这点，说明 Context中要维持对某个策略对象的引用。

- 涉及相关： 多态

- 优点：  
  1. 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。   
  2. 策略模式提供了对开放—封闭原则的完美支持，将算法封装在独立的 strategy 中，使得它 们易于切换，易于理解，易于扩展。   
  3. 策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。    
  4. 在策略模式中利用组合和委托来让 Context拥有执行算法的能力，这也是继承的一种更轻 便的替代方案。 

##### 案例： 很多公司的年终奖是根据员工的工资基数和年底绩效情况来发放的。例如，绩效为S的人年 终奖有 4倍工资，绩效为 A的人年终奖有 3倍工资，而绩效为 B的人年终奖是 2倍工资。假设财 务部要求我们提供一段代码，来方便他们计算员工的年终奖。 

```
/* 常规代码 */
var calculateBonus = function (performanceLevel, salary) {

    if (performanceLevel === 'S') {
        return salary * 4;
    }

    if (performanceLevel === 'A') {
        return salary * 3;
    }

    if (performanceLevel === 'B') {
        return salary * 2;
    }

};

calculateBonus('B', 20000); // 输出：40000 calculateBonus( 'S', 6000 );      // 输出：24000 

```
代码缺点：  
    1. calculateBonus 函数比较庞大，包含了很多 if-else 语句，这些语句需要覆盖所有的逻辑 分支。
    2. calculateBonus 函数缺乏弹性，如果增加了一种新的绩效等级 C，或者想把绩效 S 的奖金 系数改为 5，那我们必须深入 calculateBonus 函数的内部实现，这是违反开放封闭原则的。
    3. 算法的复用性差，如果在程序的其他地方需要重用这些计算奖金的算法呢？我们的选择 只有复制和粘贴。 

策略模式
```
/* em1 */
var performanceS = function(){}; 
 
performanceS.prototype.calculate = function( salary ){     return salary * 4; }; 
 
var performanceA = function(){}; 
 
performanceA.prototype.calculate = function( salary ){     return salary * 3; }; 
 
var performanceB = function(){}; 
 
performanceB.prototype.calculate = function( salary ){     return salary * 2; }; 

var Bonus = function () {
    this.salary = null; // 原始工资    
    this.strategy = null; // 绩效等级对应的策略对象 
};

var Bonus = function () {
    this.salary = null; // 原始工资    
    this.strategy = null; // 绩效等级对应的策略对象 
};

Bonus.prototype.setSalary = function (salary) {
    this.salary = salary; // 设置员工的原始工资 }; 

    Bonus.prototype.setStrategy = function (strategy) {
        this.strategy = strategy; // 设置员工绩效等级对应的策略对象 
    };

    Bonus.prototype.getBonus = function () { // 取得奖金数额 
        return this.strategy.calculate(this.salary); // 把计算奖金的操作委托给对应的策略对象 
    }
}

var bonus = new Bonus();

bonus.setSalary(10000);
bonus.setStrategy(new performanceS()); // 设置策略对象 

console.log(bonus.getBonus()); // 输出：40000     

bonus.setStrategy(new performanceA()); // 设置策略对象 
console.log( bonus.getBonus() );    // 输出：30000  

```

```
/* 策略2 */
/* 使用js的 */
var strategies = {
    "S": function (salary) {
        return salary * 4;
    },
    "A": function (salary) {
        return salary * 3;
    },
    "B": function (salary) {
        return salary * 2;
    }
};

var calculateBonus = function (level, salary) {
    return strategies[level](salary);
};

console.log(calculateBonus('S', 20000)); // 输出：80000 
console.log(calculateBonus('A', 10000)); // 输出：30000

```
