# js设计模式 

#### 单例模式

《javascript设计模式与开发实践》
    
- 定义： 保证一个类仅有一个实例，并提供一个访问它的全局访问。
    
- 描述： 单例模式是一种常用的模式，有一些对象我们往往只需要一个，比如线程池、全局缓存、浏 览器中的 window 对象等。在 JavaScript开发中，单例模式的用途同样非常广泛。
    试想一下，当我 们单击登录按钮的时候，页面中会出现一个登录浮窗，而这个登录浮窗是唯一的，无论单击多少 次登录按钮，这个浮窗都只会被创建一次，那么这个登录浮窗就适合用单例模式来创建。
    

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
