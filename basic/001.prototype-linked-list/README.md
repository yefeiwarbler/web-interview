# 原型链

## 知识点

 1. 每个构造函数都有一个prototype属性，指向其原型；原型具有constructor属性，指向原型的构造函数：

```javascript
    function A(){}

    A.prototype; // 构造函数A的原型
    A.prototype.constructor === A; // true
```

 2. 构造函数的实例存在__proto__属性，指向构造函数的原型prototype，例如：

```javascript
    function A(){}
    
    const a = new A();
    a.__proto__ === A.prototype; // true
```

 3. 构造函数A的原型 A.prototype 存在属性 \_\_proto__ 指向 Object.prototype（\_\_proto__ 属性可以理解为链表的 next 指针）

```javascript
    function A(){}
    
    A.prototype.__proto__ === Object.prototype; // true
```

 4. Object.prototype是该原型链的最后一个节点

```javascript
    function A(){}
    
    A.prototype.__proto__; // Object.prototype
    A.prototype.__proto__.__proto__; // null
    // new A() -> A.prototype -> Object.prototype -> null
```
完整的原型链如下：

<div style="align:center;padding:1em 0 2em">
    <img src="/assets/images/001-1.png" />
</div>

5. 访问实例的属性、方法时，如果实例没有对应的属性、方法，将沿着原型链进行查找，直至访问到属性、方法或查找到原型链的末尾。但是，无法通过这种方式修改原型属性、方法，如果需要原型属性、方法，必须通过__proto__显示修改

```javascript
    function A(){

    }
    A.prototype.name = "a";
    
    // 可以通过"."运算符访问原型属性
    const a1 = new A();
    a1.name; // "a"

    // 直接通过"."运算符无法修改原型属性，而是为实例添加/修改自有属性
    a1.name = "a1";
    a1.name; // "a1"
    a1.__proto__.name; // "a"

    // 可以通过__proto__修改原型属性
    a1.__proto__.name = "A";
    a1.__proto__.name; // "A"
```

## 原型链的应用

1. 原型继承
    - 直接继承：新建一个父类的实例，将子类的原型指向该实例

    ```javascript
        // 父类
        function Parent(props){
            this.array = props.array || [];
        }
        Parent.prototype.log = function(s){ console.log(s) };

        // 子类
        function Child(props){
            Parent.call(this, props); // 继承父类的实例属性、方法
            this.name = props.name || "";
        }
        Child.prototype = new Parent({});

        let a = new Child({
            array: [1,2,3],
            name: "a"
        });
        a.log("Hello!"); // "Hello!"
        
        // 由于子类的原型是父类的实例，因此能通过原型访问到该父类实例的属性
        delete a.array;
        a.array; // []
    ```

    这种继承方式中，每个子类实例都能通过原型访问原型上的父类实例属性，子类本身也存在相同键值的自有属性，容易造成混淆。此时的原型链如下图所示：

    <div style="align:center;padding:1em 0 2em">
        <br/>
        <img src="/assets/images/001-2.png" />
        <br/>
        <br/>
    </div>

    - 借助空函数实现继承

    ```javascript
        // 父类
        function Parent(props){
            this.array = props.array || [];
        }
        Parent.prototype.log = function(s){ console.log(s) };

        // 空函数
        function M(){}
        M.prototype = Parent.prototype;

        // 子类
        function Child(props){
            Parent.call(this); // 继承父类的实例属性、方法
            this.name = props.name || "";
        }
        Child.prototype = new M();
        Child.prototype.constructor = Child;

        let a = new Child({
            array: [1,2,3],
            name: "a"
        });
        a.log("Hello!"); // "Hello!"
        
        delete a.array;
        a.array; // undefined
    ```

    原型链如下图所示：

    <div style="align:center;padding:1em 0 2em">
        <img src="/assets/images/001-3.png" />
    </div>

## 一些面试题
  1. 原型链基础考察

```javascript
    var A = function() {};
    A.prototype.n = 1;
    var b = new A();

    A.prototype = {
        n: 2,
        m: 3
    };
    var c = new A();
    // 输出？
    console.log(b.n);
    console.log(b.m);

    console.log(c.n);
    console.log(c.m);
```

生成实例时，实例的__proto__属性指向构造函数的ptototype，修改构造函数的prototype指向不会影响实例的__proto__指向。因此

```
    b.__proto__ = {n: 1, ...(constructor, __proto__等属性)}
    c.__proto__ = {n: 2, m: 3}
```

所以，该题的输出是

```javascript
    b.n; // 1
    b.m; // undefined
    c.n; // 2
    c.m; // 3
```

  2. 自有属性和原型继承属性

```javascript
    function A() {
        this.name = "a";
        this.color = ["green", "yellow"];
    }
    function B() {};
    B.prototype = new A();
    var b1 = new B();
    var b2 = new B();

    b1.name = "change";
    b1.color.push("black");

    console.log(b2.name);
    console.log(b2.color);
```

"."运算符可以通过原型链访问原型属性、方法，但无法为其直接赋值，直接赋值将被视为实例的属性、方法。

因此，该题输出如下：

```javascript
    b2.name; // "a"
    b2.color; // ["green", "yellow", "black"]
```

  3. new运算符
  使用 new 运算符调用函数，将执行以下步骤
      1. 创建一个新的空对象
      2. 将构造函数的作用域赋给新对象，因此this指向该对象
      3. 执行构造函数中的代码，为对象添加属性
      4. 返回新对象

```javascript
    function myNew(Fn, ...args){
        // 实例的__proto__指向Fn.prototype
        const obj = Object.create(Fn.prototype);
        // 将Fn的this指向新建的对象，调用Fn
        const res = Fn.call(obj, ...args);
        // 如果函数调用结果是对象则返回该对象，否则返回新对象
        return res instanceof Object ? res : obj;
    }

    // es5
    function myNewES5(Fn){
        var obj = {};
        obj.__proto__ = Fn.prototype;
        var args = Array.prototype.slice.call(arguments, 1);
        var res = Fn.apply(obj, args);
        return myInstanceof(res, Object) ? res : obj ;
    }

    function myInstanceof(Obj1, Obj2){
        var baseTypes = ["undefined", "number", "symbol", "bigint", "string"];
        if(Obj1 === null || baseTypes.indexOf(typeof Obj1) > -1){
            return false;
        }
        var proto2 = Obj2.prototype;
        while(Obj1.__proto__ != null){
            Obj1 = Obj1.__proto__;
            if(Obj1 === proto2){
                return true;
            }
        }
        return false;
    }
```