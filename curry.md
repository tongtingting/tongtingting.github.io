# curry

### curry 的概念

只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

```javascript
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

这是一个入门的例子，可能有人会觉得柯里化没什么明显的好处，好像把简单问题复杂化了，没错，他本来就不是解决简单函数的，来上一组复杂函数看看。

### 来看一组问题

#### 1. 写一个函数, 可以连接字符数组, 如 `javascript f(['1','2']) => '12'`

不用柯里化, 怎么写? --reduce

```javascript
var concatArray = function(chars) {
  return chars.reduce(function(a, b) {
    return a.concat(b);
  });
};
concat(['1', '2', '3']); // => '123'
```

#### 2. 现在我要其中所有数字加 1, 然后在连接

```javascript
var concatArray = function(chars, inc) {
  return chars
    .map(function(char) {
      return +char + inc + '';
    })
    .reduce(function(a, b) {
      return a.concat(b);
    });
};
console.log(concatArray(['1', '2', '3'], 1)); // => '234'
```

#### 3. 所有数字乘以 2, 再重构试试看

```javascript
var multiple = function(a, b) {
  return +a * b + '';
};
var concatArray = function(chars, inc) {
  return chars
    .map(function(char) {
      return multiple(char, inc);
    })
    .reduce(function(a, b) {
      return a.concat(b);
    });
};
console.log(concatArray(['1', '2', '3'], 2)); // => '246'
```

是不是已经看出问题了呢? 如果我在需要每个数字都减 2,是不是很麻烦呢.需要将 map 参数匿名函数中的 multiple 函数换掉. 这样一来 concatArray 就不能同时处理加, 乘和减? 那么怎么能把他提取出来呢? 来对比下柯里化的解法.

### 柯里化解法

```javascript
var multiple = function(a) {
  return function(b) {
    return +b * a + '';
  };
};

var plus = function(a) {
  return function(b) {
    return +b + a + '';
  };
};
var concatArray = function(chars, stylishChar) {
  return chars.map(stylishChar).reduce(function(a, b) {
    return a.concat(b);
  });
};
console.log(concatArray(['1', '2', '3'], multiple(2)));
console.log(concatArray(['1', '2', '3'], plus(2)));
```

好处：

1.  处理数组中字符的函数被提取出来, 作为参数传入
2.  提取成柯里化的函数, 部分配置好后传入,函数更清晰

### 典型应用：Function.prototype.bind

bind 方法 将第一个参数设置为函数执行的上下文，其他参数依次传递给调用方法（函数的主体本身不执行，可以看成是延迟执行），并动态创建返回一个新的函数。

```javascript
var foo = { name: ttt };

var bar = function() {
  console.log(this.name);
}.bind(foo); // 绑定

bar(); // ttt
```

JS 中为了防止函数“丢失”自己原本的 this 绑定，有一种`硬绑定`的实现方法：

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
};

var bar = function() {
  foo.call(obj);
};

bar(); // 2
setTimeout(bar, 100); // 2

// `bar` 将 `foo` 的 `this` 硬绑定到 `obj`
// 所以它不可以被覆盖
bar.call(window); // 2
```

我们创建了一个函数 bar()，在它的内部手动调用 foo.call(obj)，由此强制 this 绑定到 obj 并调用 foo。无论你过后怎样调用函数 bar，它总是手动使用 obj 调用 foo。这种绑定即明确又坚定，所以我们称之为 `硬绑定（hard binding）`。

用 `硬绑定` 将一个函数包装起来的最典型的方法，是为所有传入的参数和传出的返回值创建一个通道：

```javascript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}

var obj = {
  a: 2,
};

var bar = function() {
  return foo.apply(obj, arguments);
};

var b = bar(3); // 2 3
console.log(b); // 5
```

另一种表达这种模式的方法是创建一个可复用的帮助函数：

```javascript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}

// 简单的 `bind` 帮助函数
function bind(fn, obj) {
  return function() {
    return fn.apply(obj, arguments);
  };
}

var obj = {
  a: 2,
};

var bar = bind(foo, obj);

var b = bar(3); // 2 3
console.log(b); // 5
```

当然，原生 bind 的实现没有这么简单，有兴趣的可以自己研究一下～
