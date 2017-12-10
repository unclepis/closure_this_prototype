# closure_this_prototype
《你不知道js》读书笔记

## 1. 闭包 closure
### 1-1 背景：js的作用域包括函数作用域和全局作用域

### 1-2 效果：
- 在函数内部不可以访问到全局作用域的属性和方法
- 但是在函数作用域内部可以访问到全局作用域中的属性和方法

### 1-3 问题：如何在函数外部获取到函数内部的属性？

### 1-4 解决方案：
在函数内部再写一个函数讲需要再全局访问的属性return出去

	function test(){
		var needVariable = 'test name';
		function closureFunction(){
			return needVariable;
		}
		return closureFunction;
	}
	var result = test();
	result();
- 在上面的例子中，test就是一个定义在全局的函数，因为在函数内部定义了局部变量needVariable,在test函数外面的全局环境中拿不到needVariable，所以在test函数里定义了closureFuncton把需要的局部变量return出去。

### 上面的closureFunction就是一个闭包

## 2. this的指向
### 2-1 背景：
- 在函数中，为了使用方便，常常有用this自带当前函数运行环境的对象，但是this的指向不是在函数定义阶段就绑定的，所以this的指向在函数运行时会发生变化引发问题

### 2-2 基本的规则
- 在函数执行的时候，谁调用这个函数this就指向这个对象。

### 2-3 常见场景
- 纯函数this为全局window对象
		
		var global = 1; 
		function test(){
			var global = 2;
			return this.global;
		}	
		test(); //1
		window.test(); // 1
	    window.test === test // true
- 当函数是对象的方法，this指向该对象
		
		var name = 'uncle';
		var a = {
			name:'pis',
			showName:function(){
				return this.name;
			}
		};	
		a.showName(); // pis	
- 通过构造函数new的实例，this指向实例对象
		
		var name = 'eric';
		function People(name){
			this.name = name;
			this.showName = function(){
				return this.name
			}
		}
		var uncle = new People('pis');
		uncle.showName(); // pis
- 通过apply方法对函数调用this进行绑定

		var name = 'uncle';
		var a = {
			name:'pis',
			showName:function(){
				return this.name;
			}
		};	
		a.showName.apply(a); // pis
		a.showName.apply(); // 默认绑定在window，所以输出uncle
		
		
## 3.构造函数和原型链
### 3.1 背景

#### 为什么不使用class类的概念
- js设计的时候因为没有设计class类的概念，js实现面向对象的方式就是通过构造函数constructor把属性和方法封装在一个对象里面

#### 构造函数是什么？作用？
- 构造函数相当于提供了一个生成实例对象的模板
- 构造函数：其实就是一个普通函数，但是有一些特殊特性：

	> 1.内部使用了this变量，而且this指向生成的实例对象
	
	> 2.对构造函数使用new操作，就可以通过构造函数的模板生成实例
	
	> 3.一般构造函数的名字都是首字母大写，驼峰体的写法
		
			function HumanBeings(nationality,gender){
				this.nationality = nationality;
				this.gender= gender;
			}

### 3.2 面向对象设计模式
#### 3.2.1 生成实例对象的原始模式
	1.定义一个实例对象的规格(什么样子)
		　　var Cat ={
		　　	name:'',
		　　	color:''
		　　};
	2.大家都按照这个规格定义实例对象
		　var cat1 = {}; // 创建一个空对象			 cat1.name = "大毛"; // 按照原型对象的属 性赋值
		　cat1.color = "黄色";
		　var cat2 = {};
		　cat2.name = "二毛";
		　cat2.color = "黑色";
	3.问题：
	1）cat1，cat2虽然都是按照Cat定义的规格生成的，cat1和cat2没有任何关系，而且每次定义都要重复
	2）看不出来实例对象和原型对象的关系

#### 3.2.2 原始模式的改进
	1.定义一个函数避免重复代码
	function Cat(name,color){
		return {
			name:name,
			color:color
		}
	}
	2.使用函数
	var cat1 = Cat('eric','red');
	var cat2 = Cat('cindy','blue');
	3.问题：cat1和cat2没有任何关系且实例对象和原型对象之间没有关系
	
#### －－－－－－－－－－关键点－－－－－－－－－－－－
#### 3.2.3 构造函数模式
    1.定义一个函数避免重复代码
	function Cat(name,colour){
		this.name = name;
		this.colour= colour;
	}
	2.使用函数
	var cat1 = new Cat('Eric','red');
	var cat2 = new Cat('Cindy','blue');
	3.原型对象和实例对象的关系：
	1）实例对象的constructor方法指向实例的构造函数
	 cat1.constructor === Cat // true
    cat2.constructor === Cat // true
    2)instanceof
    cat1 instantceof Cat // true
    cat2 instantceof Cat // true
    4.问题：
    实例共有的方法需要重复定义，造成存储和代码上的资源浪费
    function Cat(name,colour){
		this.name = name;
		this.colour= colour;
		this.spice = 'animals';
	}
#### 3.2.4 原型链模式
    1.每一个构造函数都有一个prototype属性，指向另一个对象，这个对象的所有属性和方法都会被构造函数生成的实例对象所继承
    function Cat(name,colour){
        this.name = name;
        this.colour = colour;
    }
    Cat.prototype = {
        spice:'animals'
    };
    var cat1 = new Cat('Eric','red');
	var cat2 = new Cat('Cindy','blue');
	cat1.spice; // 'animals' 

### ------------------重点------------------------
### 4.原型链继承
        
        function Cat(name,colour){
            this.name = name;
            this.colour = colour;
        }
        
          function Animals(name){
            this.name = name;
            this.barke = function(){
                return this.name
            };
        }
        
        var cat1 = new Cat('pis','red');
        var cat2 = new Animals('uncle');
        
#### 问题：如何让Cat类继承Animals类的方法？

### 4.1 通过构造函数apply绑定
            
        function Animals(name){
            this.name = name;
            this.barke = function(){
                return this.name
            };
        }
        
          function Cat(name,colour){
            Animals.apply(this)；// 把Animals构造函数绑定到Cat构造函数上
            this.name = name;
            this.colour = colour;
        }
            var cat1 = new Cat('pis','red');
            cat1.name // pis
            cat1.barke() // pis
        
    1)通过把Animals的构造函数绑定在Cat的构造函数上，这样当Cat生成的实例对象访问barke属性的时候，就会掉用Animals对象的barke方法
    2)需要注意的是，如果animals和cat类中有相同的属性或者方法会被覆盖，所以在本例中，是先绑定了animals的构造函数是的cat有了barke的方法，然后在对cat的静态方法进行get，set赋值，因为name属性会被覆盖
    
         function Cat(name,colour){
            this.name = name;
            this.colour = colour;
            Animals.apply(this)；// 这个写到下面
        }
            var cat1 = new Cat('pis','red');
            cat1.name // undefined
            cat1.barke() // undefined
            
### 4.2 prototype模式

#### 4.2.1 背景知识铺垫
- 构造函数都有一个prototype对象，通过new操作生成的实例对象都能继承prototype上的属性和方法
- 在new操作生成实例的时候，先将prototype对象给实例对象，再将构造函数上定义的静态或者本地属性和方法给实例对象
    
        // 构造函数
        function TestClass(attr){
            this.attr = attr;
            this.method = function(){};
            prototype:{
                constructor:TestClass // 指向构造函数
                otherAttr:'',
                otherMethd:function(){}
            }
        }
        
        // 实例对象
        var test1 = new TestClass(attr);
        
所以  
         
         1)它相当于完全删除了prototype 对象原先的值，然后赋予一个新值。
         Cat.prototype = new Animal(); 
         2)Cat.prototype.constructor是指向Cat的；加了这一行以后，Cat.prototype.constructor指向Animal
         Cat.prototype.constructor = Cat;
         3)var cat1 = new Cat("大毛","黄色");
         4)alert(cat1.species); // 动物
