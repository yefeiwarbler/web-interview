# var、let和const的区别

  1. let 和 const 声明的变量存在暂时性死区，在声明前访问会报错；var 声明变量会将声明提升至当前作用域的最上方，声明前可以访问，值为 undefined
```javascript
    a; // 报错：Uncaught ReferenceError: Cannot access 'a' before initialization
    b; // 报错：Uncaught ReferenceError: Cannot access 'b' before initialization
    c; // undefined

    let a = 1;
    const b = 2;
    var c = 3;

    // 声明后正常访问
    a; // 1
    b; // 2
    c; // 3
```

  2. 全局作用域下，使用var声明的变量将挂载在全局对象上，而使用 let 和 const 声明的变量不会

```javascript
    // 浏览器环境
    let a = 1;
    const b = 2;
    var c = 3;

    console.log(window.a); // undefined
    console.log(window.b); // undefined
    console.log(window.c); // 3
```

  3. let 和 const 声明的变量存在块级作用域和函数作用域，var 声明的变量只有函数作用域，没有块级作用域

```javascript
    (function(){
        let a = 1;
        const b = 2;
        var c = 3;
    })();

    // 均存在函数作用域
    console.log(a); // Uncaught ReferenceError: a is not defined
    console.log(b); // Uncaught ReferenceError: b is not defined
    console.log(c); // Uncaught ReferenceError: c is not defined

    {
        let a = 1;
        const b = 2;
        var c = 3;
    }
    // var 声明的变量不存在块级作用域
    console.log(a); // Uncaught ReferenceError: a is not defined
    console.log(b); // Uncaught ReferenceError: b is not defined
    console.log(c); // 3
```

  4. const 声明的变量必须赋予初始值，且声明后不能再

```javascript
    const a; // Uncaught SyntaxError: Missing initializer in const declaration

    const m = 1;
    m = 2; // Uncaught TypeError: Assignment to constant variable.
```

  5. 同一作用域下，var 可以重复声明同名变量，而 let 和 const 不能
```javascript
    let a = 1;
    const b = 2;
    var c = 3;

    let a = 4; // Uncaught SyntaxError: Identifier 'a' has already been declared
    var a = 4; // Uncaught SyntaxError: Identifier 'a' has already been declared

    const b = 5; // Uncaught SyntaxError: Identifier 'b' has already been declared
    
    var c = 6; // 正常赋值
    console.log(c); // 6

    var c;
    console.log(c); // 6  声明被忽略
```

## 面试题