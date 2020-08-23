# Js 继承总结

## 基于原型的继承

```javascript
function father(){
  	this.name = 'father';
		this.list = ['111','222']
}
father.prototype.getName = function(){
		console.log(`father name == ${this.name}`)
}
function child(){
		this.cname = 'child';
}
child.prototype = new father();
child.prototype.constructor = child;
child.prototype.getCname = function(){
		console.log(`child name == ${this.cname}`);
}
var c1 = new child();
c1.list.push('333')
// ['111','222','333']
var c2 = new child()
console.log(c2.list)
// ['111','222','333']
```

可以看到这种基于原型的继承对于父类上的**引用类型数据**，所有子类对象会共享。(c1.list 和 c2.list)



## 组合式继承

```javascript
function father(){
  	this.name = 'father';
		this.list = ['111','222']
}
father.prototype.getName = function(){
		console.log(`father name == ${this.name}`)
}
function child(){
		this.cname = 'child';
  	father.apply(this, [])            // 这个函数只比上面多了这一行
}
child.prototype = new father();
child.prototype.constructor = child;
child.prototype.getCname = function(){
		console.log(`child name == ${this.cname}`);
}

var ch1 = new child();
console.log(ch1)
// child {cname: "child", name: "father", list: Array(2)}
```

 可以看到组合式继承只比基于原型的继承多了一个父类构造函数在子类上的调用，这样会造成一个后果，即**子类上 会有多余的不必要属性**, name 和 list 属性其实都是父类的属性，在子类上还会多存储一次。并且父类的构造函数会被调用两次。



## 寄生组合实现继承

```javascript
function father(){
		this.name = 'father';
}
father.prototype.getName = function(){
		console.log(this.name)
}
function child(){
		this.chname = 'child';
}
function object(o){
		function F(){};
		F.prototype = o;
		return new F();
}
var prototype = object(father.prototype)
prototype.constructor = child;
child.prototype = prototype;


var c1 = new child();
```

组合式的继承已经很完美了，但是美中不足就是调用了两次父类构造函数，寄生组合式继承可以保证只调用一次。并且不会在子对象上创建不必要的，多余的属性

