## 学习笔记

- Set 和 Map 在 ES5 中的模拟实现  

```js
let set = Object.create(null);
set.foo = true;

if (set.foo) {// 检查属性是否存在
// do something
}

let map = Object.create(null);
map.foo = "bar";

let value = map.foo;// 提取属性值
console.log(value); // "bar"
```

- 模拟实现的问题  

```
键会转化为字符串
数字类型， key 5，'5'，是一样的  
对象类型，都是 [object Object]

```

- Set In ES6   

```
new Set()、add、size、has、delete、clear
Set 值不会做类型转换
Set 构造函数接受任何可迭代的对象作为属性  
Set.forEach，回调函数三个参数 1、
```

```js  
let set = new Set();
let key1 = {}, key2 = {};
set.add(5);
set.add("5");
set.add(5); // 忽略
set.add(key1);
set.add(key2); // key1,key2 都添加了，是不同 key

console.log(set.size) // size = 4 
```

- Map In ES6



# Sets and Maps

JavaScript 在绝大部分历史时期内只有一种集合类型，那就是数组（可能有人会质疑所有的非类数组对象都是键值对的集合，但其实它们的用途和数组有根本上的区别）。数组在 JavaScript 中的使用方式和其它语言很相似，但是其它集合类型的缺乏导致数组也经常被当作队列（queues）和栈（stacks）来使用。因为数组的索引只能是数字类型，当开发者觉得非数字类型的索引是必要的时候会使用非数组对象。这项用法促进了以非类数组对象为基础的 set 和 map 集合类型的实现。

set 是非重复值的集合。你一般不会像在数组中那样来访问 set 中的某个值；相反，它常被用来检查某个值是否存在。map 则是包含键和对应值的集合，所以 map 中的每个元素都有两块数据，当指定键的时候对应的值会被读取。map 常用作缓存以便之后需要的时候能快速提取数据。虽然 ECMAScript 5 没有正式支持 set 和 map，开发者使用非类数组对象对它们进行了模拟。

ECMAScript 6 正式为 JavaScript 添加了 set 和 map，本章介绍了这两种集合类型你所需要了解的的全部信息。

首先，我会讲解开发者在 ECMAScript 6 之前是使用了什么样的解决方案来实现 set 和 map，而且为什么这些方案是有问题的。在这些重要的背景浮出水面之后，我会介绍 ECMAScript 6 中 set 和 map 的工作机制。



## Sets and Maps in ECMAScript 5

在 ECMAScript 5 中，开发者使用对象属性来模拟 set 和 map，像这样：

```js
let set = Object.create(null);

set.foo = true;

// 检查属性是否存在
if (set.foo) {

    // do something
}
```

本例中的 set 对象没有原型，确保它不会继承任何属性。在 ECMAScript 5 中使用对象属性来检查唯一值的用法十分普遍。当给 set 对象添加属性并设置值为 true 之后，条件判断语句（如本例中的 if 语句）可以轻松地检查某个值是否存在。

使用对象模拟的 set 和 map 之间唯一真正的区别是键值的类型。例如，下面的例子将对象做为 map 使用：

```js
let map = Object.create(null);

map.foo = "bar";

// 提取属性值
let value = map.foo;

console.log(value);         // "bar"
```

该段代码将字符串 "bar" 赋值给了 foo 键。和 set 不同，map 大部分情况下被用来提取数据，而不是验证键是否存在


## Problems with Workarounds

虽然在简单的情况下使用对象模拟的 set 和 map 没有太大的问题，不过当条件变得复杂时对象属性的限制很快就会暴露出来。例如，既然对象属性的类型必须为字符串，你必须保证键存储的值是唯一的。考虑如下的代码：

```js
let map = Object.create(null);

map[5] = "foo";

console.log(map["5"]);      // "foo"
```

该例将 "foo" 值赋值给 5 这个数字类型键。{% em %}在内部，数字类型的键会被转化为字符串，所以 map["5"] 和 map[5] 引用了相同的属性。当你想同时使用数字和字符串类型的键时，该内部实现是制造问题的根源。{% endem %}同样，当使用对象作为键的时候也会出现麻烦，例如：

```js
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);     // "foo"
```

在这里，map[key2] 和 map[key1] 引用了相同的值。{% em %}对象中的 key1 和 key2 被转化为字符串是因为对象的属性只能为该类型。既然对象的字符串表达形式是 "[object Object]"，那么 key1 和 key2 也不例外。开发者在一般的思维下都会自然认为不同的键名就代表不同的键，因此这里会造成难以察觉的错误。{% endem %}

默认的字符串转化使得对象很难被当作键来使用（该情况同样存在于 set）。

当键的值为假的情况下也会有一些问题。在条件判断中期待接收布尔值的位置，任何假值都会被自动转换为 false，比如 if 语句。只要你对要用的值稍加注意，一般这并不是个大的问题。例如，下面的代码：

```js
let map = Object.create(null);

map.count = 1;

// 检查 "count" 是否存在或该值是否为假？
if (map.count) {
    // ...
}
```

该例中 map.count 的用法存在歧义。该语句的目的到底是检查 map.count 是否存在还是它的值是否为假。if 中的代码会执行是因为 1 被视为真值。然而 map.count 如果不存在或它的值为假则代码都不会被执行。

在大型应用的调试过程中这些都是棘手的问题，这也是 ECMAScript 6 添加 Set 和 Map 类型的主要原因之一。

<br />

> JavaScript 有 in 操作符可以在不读取值的情况下检查某个属性是否在对象中存在，如果是的话则返回 true。不过，该操作符还会检查对象的原型，这就使得该操作只有在对象不存在原型的条件下才是可靠的。即使这样，很多开发者都使用了上例中不当的方式而没有使用 in 。

## Sets in ECMAScript 6

ECMAScript 6 中的 set 类型是一个包含无重复元素的有序列表。Set 允许对内部某元素是否存在进行快速检查，使得元素的追踪操作效率更高。


### Creating Sets and Adding Items

set 由 new Set() 语句创建并通过调用 add() 方法来向 set 中添加项。你还可以查看 set 的 size 属性来获取项的数目：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size);    // 2
```

{% em %}set 在比较值是否相等的时候不会做强制类型转换。这意味着在 set 可以同时包含数字 5 和 字符串 "5"（在 set 内部，该比较使用了第四章讨论过的 Object.is() 方法来决定两者的值是否相等）。{% endem %}你可以向 set 内部添加多个对象，它们的值会被认作是不相等的：

```js
let set = new Set(),
    key1 = {},
    key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);    // 2
```

因为 key1 和 key2 不会转换为字符串，所以它们 set 认为两者都是唯一的（记住，如果它们被转换为字符串，那么值都是 "[object Object]"）。

如果 add() 方法由同一个参数调用了多次，那么首次之后的调用将会被忽略：

```js
let set = new Set();
set.add(5);
set.add("5");
set.add(5);     // duplicate - this is ignored

console.log(set.size);    // 2
```

你可以使用数组来初始化一个 set，而且 Set 构造函数会确保使用数组中唯一存在的元素。例如：

```js
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);    // 5
```

在该例中，带有重复元素的数组被用来初始化 set 。虽然在数组中数字 5 出现了 4 次，但 set 中只将它存入一次。这项功能可以方便的转换 JSON 结构中已存在的数据。

<br />

> {% em %}Set 构造函数实际上可以接收任何可迭代（iterable）对象作为属性。{% endem %}数组是默认的可迭代类型，所以它可以作为 Set 的参数。同样 set 和 map 也都是可迭代类型。Set 构造函数使用迭代器从参数中提取相应值。（可迭代类型与迭代器将在第八章讨论）

<br />

你可以使用 has() 方法来查看某个值是否在 set 中，像这样：


```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true
console.log(set.has(6));    // false
```

在这里，因为 set 中不包含 6 ，所以 set.has(6) 会返回 false 。


### Removing Values

将 set 中的值移除也是可以做到的。你可以使用 delete() 方法来销毁单个值，或者调用 clear() 方法来清空整个 set。下面的代码演示了这些操作：

```js
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true

set.delete(5);

console.log(set.has(5));    // false
console.log(set.size);      // 1

set.clear();

console.log(set.has("5"));  // false
console.log(set.size);      // 0
```

在调用 delete() 之后，只有 5 被移除；而 clear() 方法的执行使得 set 内部被清空。

这些方法提供了一个方便的机制用来追踪有序的唯一值。不过，给 set 添加项之后该怎样遍历它们呢？forEcah() 方法解决了这个问题。


### The forEach() Method for Sets

如果你曾处理过数组，那么 forEach() 方法你可能非常熟悉。ECMAScript 5 给数组添加了 forEach() 使得遍历操作每一项变得更加方便，不再需要循环。开发者在实践中享受到了好处并逐渐推广了它，于是 set 也顺势添加了该方法并且功能保持不变。

forEach() 方法接收一个含有三个参数的回调函数：

1. 下一位置的值
2. 和首个参数的值相同
3. 操作的 set 本身

比较奇怪的是 set 实现的 forEach() 方法和数组存在的差异在于接收的前两个参数是相同的。虽然这看来不大对劲，但却有意而为之的。

其它包含 forEach() 方法的对象（数组和 map）在该方法的回调函数中也会接收三个参数。前两个参数分别为下一位置的值和键（其中数组版本中的键为数字索引）。

但是 set 并不包含键。制定 ECMAScript 6 标准的相关人员本可以在 set 中设定 forEach() 的回调函数只接受两个参数。不过他们却另辟蹊径的找到了统一回调函数的办法：set 中的每一项既是键也是值。于是 set 为了和数组与 map 中的 forEach() 方法保持一致，将回调函数中的前两个参数设为相同。


除了参数个数的差异外，set 版本的 forEach() 和 数组基本相同。以下是一些代码来展示该方法是怎样工作的：

```js
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
    console.log(key + " " + value);
    console.log(ownerSet === set);
});
```

该段代码将数组中的每一项添加到 set 中然后将值传递给 forEach() 的回调函数。回调函数每一次执行时，key 和 value 都是相同的，同时 ownerSet 则等同于 set。下面是输出结果：

```js
1 1
true
2 2
true
```

结果和使用数组是相同的，你可以给 forEach() 传入第二个参数 this 以便你在该方法中使用它。

```js
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach(function(value) {
            this.output(value);
        }, this);
    }
};

processor.process(set);
```

在上例中，processor.process() 方法在 set 上调用 forEach() 并将 this 值传递给回调函数。这对于 this.output() 会正确调用 processor.ouput() 是不可或缺的。forEach() 中的回调函数只需要首个参数， value，其它的都会忽略掉。你也可以使用箭头函数来替代作为第二个参数传递的 this，像这样：

```js
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach((value) => this.output(value));
    }
};

processor.process(set);
```

该例中的箭头函数会读取包含它的 process() 函数中的 this 值，所以 this.output() 会正确的调用 processor.output() 。

需要留心的是，虽然 set 可以很好的进行值的跟踪而且 forEach() 可以顺序处理其中的每一项，但是你无法使用下标来访问其中的值。如果你想这么做，最佳的方案还是将它转化为数组。


### Converting a Set to an Array

将数组转化为 set 相当容易，你只需将数组传递给 Set 构造函数。使用扩展运算符将 set 转化为数组也并不复杂。第三章中介绍的扩展运算符（...）可以拆分数组中的项并传递给函数参数。你同样可以在可迭代对象上使用扩展运算符，例如 set，并将它转化为数组。例如：

```js
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

以上，set 在初始时读取了一个包含重复值的数组。set 清除掉了重复值之后，使用了扩展操作符将 set 中的项传入了一个新的数组当中。set 仍然包含创建时的项（1，2，3，4 和 5）。他们只是被复制到了新的数组中。

该实践在遇到清除已创建数组中的重复值的需求时特别好用。例如：

```js
function eliminateDuplicates(items) {
    return [...new Set(items)];
}

let numbers = [1, 2, 3, 3, 3, 4, 5],
    noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);      // [1,2,3,4,5]
```

在 eliminateDuplicates() 函数中，set 只是被当作临时的代理以便在创建新的数组之前过滤掉重复项。


### Weak Sets

set 类型根据它存储对象的方式，也被称为 strong set。一个对象存储在 set 内部或存储于一个变量在效果上是等同的。只要对该 Set 实例的引用存在，那么存储的对象在垃圾回收以释放内存的时候无法被销毁，例如：

```js
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);      // 1

// 销毁引用
key = null;

console.log(set.size);      // 1

// 重新获得了引用
key = [...set][0];
```

在本例中，将 key 设置为 null 只是清除了对 key 对象的一个引用，但是其它引用还存于 set 内部。你仍然可以使用扩展运算符将 set 转化为 数组后获取它。在大部分编程中这个结果是可以接受的，但某些时候，当其它引用解除之后 set 内部能自动解除相关引用是再好不过的。例如，当在一个网页中使用 JavaScript 追踪一些可能在之后会被销毁 DOM 元素，你不希望有任何残留的 DOM 元素引用存在。（这种情况称其为内存泄漏）

为了减少这些情况的出现，ECMAScript 6 同时引入了 weak set 。该类型不允许存储原始值而专门存储弱对象引用。由于弱引用不会被当做剩余存在的引用，所以它不会阻止垃圾回收。


#### Creating a Weak Set

weak set 由 WeakSet 构造函数创建并包含 add()，has() 和 delete() 方法。下面的例子使用了这些方法：

```js
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

set.delete(key);

console.log(set.has(key));      // false
```

weak set 的用法和一般的 set 类似。你可以添加，移除，查看引用，也可以给构造函数传入一个可迭代类型：

```js
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2]);

console.log(set.has(key1));     // true
console.log(set.has(key2));     // true
```

本例中一个数组被传给了 WeakSet 构造函数。因为数组中包含了两个对象，于是它们被添加到了 weak set 中。需要注意的是若数组中包含了非对象元素，一个错误会被抛出，因为 WeakSet 不接受原始值。


#### Key Differences Between Set Types

weak set 和 一般 set 的最大区别是前者存储的是弱对象引用。下面的例子解释了两者的差异：

```js
let set = new WeakSet(),
    key = {};

// 将对象添加给 set
set.add(key);

console.log(set.has(key));      // true

// 销毁了最后剩余的强引用，weak set 中的引用也随即消失
key = null;
```

当该段代码执行后，weak set 中的引用就无法访问了。不过要验证它是不可能的，因为 has() 方法需要传递一个对象的引用（weak set 之外的引用已不存在）。虽然这让包含 weak set 的测试变得有些困惑，但至少你可以确定该引用已经彻底不存在于 JavaScript 引擎当中。

These examples show that weak sets share some characteristics with regular sets, but there are some key differences. Those are:

这些例子演示了 weak set 和一般 set 的相同特称，但是它们有一些关键的差异，例如：

1. 当调用 add()，has() 或 delete() 方法传入了一个非对象参数时，一个错误会被抛出。
2. weak set 不是可迭代类型，因此不能被用在 for-of 循环中。
3. weak set 无法暴露出自身的迭代器（例如 keys() 和 values() 方法），所以没有任何编程手段来确定 weak set 中的内容
4. weak set 没有 forEach() 方法。
5. weak set 没有 size 属性。


这些针对 weak set 显而易见的功能限制对于正确的内存操作来说必不可少。一般情况下，如果你只想追踪对象的引用，你应该是用 weak set 而不是 set 。

set 给了你处理一系列值的新方式，不过若想给这些值添加附加信息则显得捉襟见肘。ECMAScript 6 因此添加了 map 。


## Maps in ECMAScript 6

ECMAScript 6 中的 map 类型包含一组有序的键值对，其中键和值可以是任何类型。键的比较结果由 Object.is() 来决定，所以你可以同时使用 5 和 "5" 做为键来存储，因为它们是不同的类型。这和使用对象属性做为值的方法大相径庭，因为对象的属性会被强制转换为字符串类型。

你可以使用 set() 方法给 map 添加键和对应的值，并在之后调用 get() 方法传入键名来提取值。例如：

```js
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);

console.log(map.get("title"));      // "Understanding ES6"
console.log(map.get("year"));       // 2016
```

该例中存储了两个键值对。"title" 键保存了一个字符串，"year" 键保存了一个数字类型。get() 方法在之后被调用并提取出了两者的值。如果键在 map 中不存在，get() 方法会返回 undefined 。

你也可以将对象用作键，这也是使用对象属性创建 map 时无法做到的。查看下例：

```js
let map = new Map(),
    key1 = {},
    key2 = {};

map.set(key1, 5);
map.set(key2, 42);

console.log(map.get(key1));         // 5
console.log(map.get(key2));         // 42
```

该段代码使用了 key1 和 key2 对象做为键并在 map 中存储了两个值。因为这些键不会被强制转换成其它类型，所以每个对象都是唯一的。这允许你给对象添加额外信息但不需要操作该对象本身。


### Map Methods

map 的一些方法效果和 set 相同。这是有意而为的，允许你使用熟悉的方式来和 map 与 set 交互。以下三个方法 map 和 set 都能使用：

* has(key) - 判断给定的 key 是否在 map 中存在
* delete(key) - 移除 map 中的 key 及对应的值
* clear() - 移除 map 中所有的键值对

map 同样包含 size 属性指明它包含了多少键值对。下面的代码用不同的方式演示了上述三种方法和 size 属性：

```js
let map = new Map();
map.set("name", "Nicholas");
map.set("age", 25);

console.log(map.size);          // 2

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"

console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25

map.delete("name");
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.size);          // 1

map.clear();
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.has("age"));    // false
console.log(map.get("age"));    // undefined
console.log(map.size);          // 0
```

和 set 一样，map 的 size 属性总是反映出包含键值对的数目。在向本例中的 map 实例添加了 "name" 和 "age" 键之后，调用 has() 并传入已存在的键名自然会返回 true。在 "name" 键被 delete() 方法移除后，has() 方法在传入 "name" 之后调用会返回 false，同时 size 属性的值会减 1 。调用 clear() 方法移除了剩下的所有键值对，所以 has() 和 size 分别返回 false 与 0 。

clear() 方法能极快的将大量的数据从 map 中移除，反之 map 也有一种方法能使大量的数据迅速入驻。


### Map Initialization

和 set 类似，你可以将数据存入数组并传给 Map 构造函数来初始化它。数组中的每一项必须也是数组，后者包含的两项中前者作为键，后者为对应值。因此整个 map 被带有两个项的数组所填充。

```js
let map = new Map([["name", "Nicholas"], ["age", 25]]);

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.has("age"));    // true
console.log(map.get("age"));    // 25
console.log(map.size);          // 2
```

"name" 和 "age" 作为键在调用构造函数时传入并初始化了该 map 。虽然数组中包含数组看起来有些奇怪，不过这对键的精准描述时必须的，因为键可以是任何类型。将键放入数组是唯一可以在添加给 map 之前不被强制执行类型转换的办法。


### The forEach Method on Maps

map 的 forEach() 方法类似于 set 和 数组，它同样接收一个含有三个参数的回调函数：

1. map 中下一位置的值
2. 该值对应的键
3. 正在读取的 map 本身


这些回调函数参数的行为更接近数组中的 forEach() 方法，前两个参数分别是值和键（数组中是值的数字索引）。这里有个示例：

```js
let map = new Map([ ["name", "Nicholas"], ["age", 25]]);

map.forEach(function(value, key, ownerMap) {
    console.log(key + " " + value);
    console.log(ownerMap === map);
});
```

forEach() 的回调函数输出了传给它的信息。其中 value 与 key 被直接输出，ownerMap 用来和 map 比较是否相等。以下为输出结果：

```js
name Nicholas
true
age 25
true
```

键值对按照存入 map 中的顺序依次传给 forEach() 中的回调函数。这里和数组有些不同，后者 forEach() 中的回调函数是按照数组中数字索引的顺序来读取每一项。

<br />

> 你也可以给 forEach() 传入第二个参数来制定回调函数中的 this 值，其行为和 set 中的 forEach() 一致。


### Weak Maps

weak map 和 map 的关系就像 weak set 和 set 一样：前者都是一种存储弱对象引用的方式。在 weak map 中，所有的键必须是对象（否则会抛出错误），而且这些对象都是弱引用，不会干扰垃圾回收。当 weak map 中的键在 weak map 之外不存在引用的时候，该键及对应的值会被移除。

weak map 的最佳实践是创建一个对象并和网页中的特定 DOM 元素关联。例如，某些作用于网页的 JavaScript 库会为每一个绑定的 DOM 元素维护一个由该库中衍生的自定义对象，并将其映射到作为缓存的对象内部。

该实现棘手的部分在于，当 DOM 元素不存在于网页中之后要销毁该元素关联的对象。否则，库就会保持一个无效的 DOM 元素引用并造成潜在的内存泄漏影响。若使用 weak map，库依然可以将对象与每个 DOM 元素相关联，而且在该 元素不存在的时候还能自动销毁相关联对象。

<br />

> 必须注意的是，弱引用指的是键的弱引用，而不只是值的弱引用。将对象存储为 weak map 中的值仍然会在其余引用不存在的情况下妨碍垃圾回收。


##### Using Weak Maps


ECMAScript 6 中的 WeakMap 类型是无序键值对的集合，其中键必须是非 null 的对象，值可以是任意类型。WeakMap 和 Map 相似的地方在于都能使用 set() 和 get() 来分别添加及移除数据：

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

let value = map.get(element);
console.log(value);             // "Original"

// 移除该元素
element.parentNode.removeChild(element);
element = null;

// 现在 weak map 的内部为空
```

该例中存储了一对键值。element 键为一个 DOM 元素并存储了一个对应的字符串值。当传递该 DOM 元素给 get() 方法之后对应的值会被提取。当 DOM 元素从文档（document）中移除并将变量设置为 null 后，相应的数据就会在 weak map 中销毁。

和 weak set 类似的是，没有任何办法可以确认 weak map 是否为空，因为它不存在 size 属性。由于没有剩余的引用存在，你无法通过给 get() 方法传入引用来提取相应的值，而且该值也已经被清除了。当垃圾回收器运行时占有的内存会被自动释放。



##### Weak Map Initialization


若想初始化 weak map，只需将数组传递给 WeakMap 构造函数。和一般的 map 初始化相同，数组内部的元素必须是包含两项的数组，前一项是个非 null 的对象作键，后一项则是值（任意类型）。例如：

```js
let key1 = {},
    key2 = {},
    map = new WeakMap([[key1, "Hello"], [key2, 42]]);

console.log(map.has(key1));     // true
console.log(map.get(key1));     // "Hello"
console.log(map.has(key2));     // true
console.log(map.get(key2));     // 42
```

key1 和 key2 对象作为键传给了 weak map，get() 和 has() 方法可以访问它们。若 WeakMap 接收的键值对中存在非对象的键，那么一个错误会被抛出。

<br />

##### Weak Map Methods


weak map 只有两种方法和键值对交互。has() 方法可以由给定的键来判断其是否存在于 weak map 内部，delete() 方法用来删除指定的键值对。clear() 方法并不存在，因为它需要对键进行枚举。和 weak set 同理，weak map 是不可能做到的。下面的例子同时用到了 has() 和 delete() 方法：

```js
let map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

console.log(map.has(element));   // true
console.log(map.get(element));   // "Original"

map.delete(element);
console.log(map.has(element));   // false
console.log(map.get(element));   // undefined
```

在这里，一个 DOM 元素再次作为键添加给了 weak map。为了查看 weak map 中是否有该引用作为键，has() 正是派上用场的时候，不过要注意键必须是非 null 的引用类型。delete() 方法会强制将传入的键从 weak map 中移除，导致再次传入该键给 has() 或 get() 会分别返回 false 和 undefined 。


##### Private Object Data


虽然大多数开发者认为 weak map 的主要用途是让 DOM 元素和相关数据进行协作，实际上其它的用途还有很多（毋须置疑的是，还有很多用法尚待发掘）。例如其中一种就是存储对象内部的私有数据。在 ECMAScript 6 中所有的对象属性都是公有的，所以你就需要创造一些方法来让数据只在对象内部有效而对外封闭。考虑下面的示例：

```js
function Person(name) {
    this._name = name;
}

Person.prototype.getName = function() {
    return this._name;
};
```

该段代码使用了下划线这种广泛表示私有属性的写法来表明该成员只能在内部使用而不能被对象实例所修改，意图只能使用 getName() 来访问但不能修改它。然而，没有任何办法能阻止其它人重新给 _name 属性赋值，所以它依然可以在内部或意外地被重写。

In ECMAScript 5, it’s possible to get close to having truly private data, by creating an object using a pattern such as this:

在 ECMAScript 5 中，创造真正的私有数据并不是遥不可及的，如使用下面的方式来创建对象：

```js
var Person = (function() {

    var privateData = {},
        privateId = 0;

    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });

        privateData[this._id] = {
            name: name
        };
    }

    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };

    return Person;
}());
```

在本例中 Person 的定义被 IIFE 包裹的同时拥有两个私有变量：privateData 和 privateId ，它们分别用来存储每个实例的信息和生成的唯一 ID 。当 Person 构造函数被调用后，一个无法枚举（nonenumerable），无法修改（nonconfigurable）且无法赋值（nonwritable）的属性 _id 会被添加。

之后，为了能使用给定的 ID 获取 name 的信息，一个由实例访问 privateData 对象的入口被创建，即接下来的 getName() 函数通过 this._id 作为键来访问 privateData 。因为 privateData 在 IIFE 的外部不可见，即使它可以由暴露给公共的 this._id 来访问，该对象仍然是安全的。

这种方式最大的问题在于当对象实例销毁时没有获知 privateData 内部相关数据的办法，于是该对象会驻留在内存中并包含额外的数据。这个问题可以由 weak map 来解决，如下所示：

```js
let Person = (function() {

    let privateData = new WeakMap();

    function Person(name) {
        privateData.set(this, { name: name });
    }

    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };

    return Person;
}());
```

该版本示例中 Person 使用 weak map 替换了对象来存储私有数据。Person 对象的实例本身可用作键，所以一个分离的 ID 就没有必要了。当 Person 构造函数被调用时，this 的值和一个带有私有信息对象分别作为键和值添加到了 weak map 中。在本例中，传入的对象仅包含 name 。getName() 函数通过调用 prviateData.get() 方法来提取相关的私有信息并可以访问 name 属性。该方案下私有信息仍然是私有的，而且在实例被销毁的同时该信息也会随即消失。


##### Weak Map Uses and Limitations

每当纠结于 map 或 weak map 的选择时，首要考虑的因素是你是否只想使用对象作为键。如果答案为是，那么最好的选择就是 weak map 。因为它能优化内存的占用并通过在对象销毁之后删除额外的信息来防止内存泄漏。

需要注意的是 weak map 稍微减少了自身内容的可见度，所以你不能使用 forEach() 方法，size 属性或 clear() 方法来管理集合中的项。如果你需要较强的操控能力，那么一般的 map 会是个更好的选择，不过要留意内存的占用。

当然，如果你想使用非对象类型作键，那么 map 就是你唯一的选择。


##  Summary

ECMAScript 6 正式在 JavaScript 中引入了 set 和 map。在这之前，开发者经常使用对象来模拟它们，不过由于对象属性自身的限制总会导致一些问题。

set 是包含 ~~无序~~ 有序非重复项的集合。其中的项是否唯一由 Object.is() 方法来判断。set 会自动移除重复的项，所以你可以使用它来过滤数组中的重复元素并返回结果。set 并不是数组的子集，所以你不能随意的去访问 set 中的值。相反，你可以使用 has() 方法来判断某个值是否存在于 set 中或通过 size 属性来查看项的数目。set 类型还包含 forEach() 方法用来对每一项进行操作。

weak set 是只包含对象的特殊 set 。这些对象被当作弱引用存储，意味着在它们当中如果有的项是仅存的某个对象的引用，那么该项不会阻止垃圾回收。

map 是包含键值对的 ~~无序~~ 有序集合。和 set 类似，键是否重复由内部调用 Object.is() 决定，这意味着 map 中数字类型 5 和 字符串类型 "5" 可以分别作为键共存。任何类型的值可以通过调用 set() 方法和对应的键一同添加到 map 中，之后还可以使用 get() 方法来提取该值。map 还包括 size 属性与 forEach() 方法来方便访问集合中的项。

weak map 是只包含对象键的特殊 map 。和 weak set 类似，键的是弱对象引用，因此当其为仅存的某个对象的引用时，垃圾回收不会被阻止。当键被垃圾回收器清理之后，所关联的值也一并销毁。当想要将额外的信息附加到生命周期可由外部代码控制的对象上时，带有内存管理的 weak map 类型是唯一适合的。

