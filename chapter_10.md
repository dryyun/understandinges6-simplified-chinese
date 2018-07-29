## 学习笔记

- [Array.of()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/of) 和 [Array.from()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
- [Array.find()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find) 和 [Array.findIndex()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)
- [fill()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/fill) 和 [copyWithin()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin)
- [Typed Array ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 没细看，就了解了一下

# 改进的数组功能


数组是 JS 中的一种基本对象。 JS 的其他方面都随着时间的推移在进化，而数组却基本保持不变，直到 ES5 才添加了几个相关的方法让数组更易使用。 ES6 也添加了很多功能来继续强化数组，例如新的创建方法、几个有用的便捷方法，还增加了创建类型化数组（typed array）的能力。

##  创建数组

在 ES6 之前创建数组主要存在两种方式： `Array` 构造器与数组字面量写法。这两种方式都需要将数组的项分别列出，并且还要受到其他限制。将“类数组对象”（即：拥有数值类型索引与长度属性的对象）转换为数组也并不自由，经常需要书写额外的代码。为了使数组更易创建， ES6 新增了 `Array.of()` 与 `Array.from()` 方法。

###  Array.of() 方法

ES6 为数组新增创建方法的目的之一，是帮助开发者在使用 `Array` 构造器时避开 JS 语言的一个怪异点。调用 `new Array()` 构造器时，根据传入参数的类型与数量的不同，实际上会导致一些不同的结果。例如

```js
let items = new Array(2);
console.log(items.length);          // 2
console.log(items[0]);              // undefined
console.log(items[1]);              // undefined

items = new Array("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2"

items = new Array(1, 2);
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = new Array(3, "2");
console.log(items.length);          // 2
console.log(items[0]);              // 3
console.log(items[1]);              // "2"
```

当使用单个数值参数来调用 `Array` 构造器时，数组的长度属性会被设置为该参数；而如果使用单个的非数值型参数来调用，该参数就会成为目标数组的唯一项；如果使用多个参数（无论是否为数值类型）来调用，这些参数也会成为目标数组的项。数组的这种行为既混乱又有风险，因为有时可能不会留意所传参数的类型。

ES6 引入了 `Array.of` 方法来解决这个问题。该方法的作用非常类似 Array 构造器，但在使用单个数值参数的时候并不会导致特殊结果。 `Array.of()` 方法总会创建一个包含所有传入参数的数组，而不管参数的数量与类型。下面几个例子演示了 `Array.of()` 的用法：

```js
let items = Array.of(1, 2);
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = Array.of(2);
console.log(items.length);          // 1
console.log(items[0]);              // 2

items = Array.of("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2"
```

在使用 `Array.of()` 方法创建数组时，只需将想要包含在数组内的值作为参数传入。第一个例子创建了一个包含两个项的数组，第二个数组只包含了单个数值项，而最后一个数组则包含了单个字符串项。这个结果类似于使用数组字面量写法，通常你都可以在原生数组上使用字面量写法来代替 Array.of() ，但若想向函数传递参数，使用 Array.of() 而非 Array 构造器能够确保行为一致。例如

```js
function createArray(arrayCreator, value) {
    return arrayCreator(value);
}

let items = createArray(Array.of, value);
```

此代码中的 `createArray()` 函数接受两个参数，一个数组创建器与一个值，并会将后者插入到目标数组中。你应该想 createArray() 函数传递 Array.of() 作为第一个参数来创建新数组；相反，若传递 `Array` 构造器则会有危险，因为你无法保证第二个参数类型。

> Array.of() 方法并没有使用 Symbol.species 属性（参阅第九章）来决定返回值的类型，而是使用了当前的构造器（即 of() 方法内部的 this ）来做决定。

###  Array.from() 方法

在 JS 中将非数组对象转换为真正的数组总是很麻烦。例如，若想将类数组的 `arguments` 对象当做数组来使用，那么你首先需要对其进行转换。在 ES5 中，进行这种转换需要编写一个函数，类似

```js
function makeArray(arrayLike) {
    var result = [];

    for (var i = 0, len = arrayLike.length; i < len; i++) {
        result.push(arrayLike[i]);
    }

    return result;
}

function doSomething() {
    var args = makeArray(arguments);

    // use args
}
```

该方式手动创建了一个 `result` 数组，并将 arguments 对象的所有项复制到该数组中。这种方式虽然有效，却为一个简单操作书写了过多的代码。开发者最终发现他们可以调用数组原生的 `slice()` 方法来减少代码量，类似：

```js
function makeArray(arrayLike) {
    return Array.prototype.slice.call(arrayLike);
}

function doSomething() {
    var args = makeArray(arguments);

    // use args
}
```

这段代码的功能与前一段代码等效。它能正常工作是因为将 slice() 方法的 this 设置为类数组对象，slice() 只需要有数值类型的索引与长度属性就能正常工作，而类数组对象能满足这些要求。

尽管这种技巧所用的代码量更少，但调用 `Array.prototype.slice.call(arrayLike)` 并没有明确体现出“要将类数组对象转换为数组”的目的。幸运的是，ES6 新增了 Array.from() 方法来提供一种明确清晰的方式以解决这方面的需求。

将可迭代对象或者类数组对象作为第一个参数传入，`Array.from()` 就能返回一个数组。这里有个简单的例子：

```js
function doSomething() {
    var args = Array.from(arguments);

    // use args
}
```

这里调用 `Array.from()` 方法，使用 `arguments` 对象创建一个新数组 `args`，他是一个数组实例，包含 `arguments` 对象的所有项，同时保持项的顺序。

>  Array.from() 同样使用 this 来决定返回什么类型的数组

##### 映射转换


如果你想实行进一步的数组转换，你可以向 Array.from() 方法传递一个映射用的函数作为第二个参数。此函数会将类数组对象的每一个值转换为目标形式，并将其存储在目标数组的对应位置上。例如：

```js
function translate() {
    return Array.from(arguments, (value) => value + 1);
}

let numbers = translate(1, 2, 3);

console.log(numbers);               // 2,3,4
```

此代码将 (value) => value + 1 作为映射函数传递给了 Array.from() 方法，对每个项进行了一次 +1 处理。如果映射函数需要在对象上工作，你可以手动传递第三个参数给 Array.from() 方法，从而指定映射函数内部的 this 值。

```js
let helper = {
    diff: 1,

    add(value) {
        return value + this.diff;
    }
};

function translate() {
    return Array.from(arguments, helper.add, helper);
}

let numbers = translate(1, 2, 3);

console.log(numbers);               // 2,3,4
```

这个例子使用了 `helper.add` 作为映射函数。由于该函数使用了 `this.diff` 属性，你必须向 Array.from() 方法传递第三个参数用于指定 this ，借助这个参数， Array.from() 就可以方便地进行数据转换，而无须调用 bind() 方法或用其他方式去指定 this 值。

##### 在可迭代对象上使用


Array.from() 方法不仅可用于类数组对象，也可用于可迭代对象，这意味着该方法可以将任意包含 `Symbol.iterator` 属性的对象转换为数组。例如：

```js
let numbers = {
    *[Symbol.iterator]() {
        yield 1;
        yield 2;
        yield 3;
    }
};

let numbers2 = Array.from(numbers, (value) => value + 1);

console.log(numbers2);              // 2,3,4
```

由于代码中的 numbers 对象是一个可迭代对象，你可以把它直接传递给 Array.from() 方法，从而将它包含的值转换为数组。映射函数对每个数都进行了 +1 处理，因此目标数组的内容就是 2 、 3 、 4 ，而不是 1 、 2 、 3 。

> 如果一个对象既是类数组对象，又是可迭代对象，那么迭代器就会使用  Array.from 方法来决定需要转换的值。


##  数组的新方法


ES6 延续了 ES5 的工作，为数组增加了几个新方法。 `find()` 与 `findIndex()` 方法是为了让开发者能够处理包含任意值的数组，而 `fill()` 与 `copyWithin()` 方法则是受到了类型化数组（ typed arrays ）的启发。类型化数组是在 ES6 中引入的，只允许包含数值类型的值。


### find() 和 findIndex() 方法


在 ES5 之前，检索数组是件麻烦事，因为没有对应的原生方法可用。 ES5 增加了 `indexOf()` 与 `lastIndexOf()` 方法，从而允许开发者在数组中查找特定值。这虽然是很大的进步，但依然受到了一些限制，因为你每次只能用它们来查找某个特定值。例如，若想在一系列的数中间查找第一个偶数，你必须自己写代码来实现这个意图。而 ES6 引入了 find() 与 findIndex() 方法，从而解决了这方面的问题。

find() 与 findIndex() 方法均接受两个参数：一个回调函数、一个可选值用于指定回调函 数内部的 this 。该回调函数可接收三个参数：数组的某个元素、该元素对应的索引位置、 以及该数组自身，这与 `map()` 和 `forEach()` 方法的回调函数所用的参数一致。该回调函数应当在给定的元素满足你定义的条件时返回 `true`，而 find() 与 findIndex() 会在回调函数第一次返回 true 时停止查找。

二者唯一的区别是：find() 方法会返回匹配的值，而 findeIndex() 方法则会返回匹配位置的索引，示例：

```js
let numbers = [25, 30, 35, 40, 45];

console.log(numbers.find(n => n > 33));         // 35
console.log(numbers.findIndex(n => n > 33));    // 2
```

这段代码使用了 find() 与 findIndex() 方法在 numbers 数组中查找第一个大于 33 的元 素，前者返回 35，后者返回 2 ，也就是 35 的索引值。

find() 与 findIndex() 方法在查找特定条件的数组元素时非常有用。但若想查找特定值，则只用 indexOf() 与 lastIndexOf () 方法会是更好的选择。

###  fill() 方法


fill() 方法能使用特定值填充数组中的一个或多个元素 。当只使用一个参数的时候，该方法会用该参数的值填充整个数组，例如：

```js
let numbers = [1, 2, 3, 4];

numbers.fill(1);

console.log(numbers.toString());    // 1,1,1,1
```

此代码中的 `numbers.fill(1)` 调用将 numbers 数组中的所有元素都填充为 1 。若你不想改变数组中的所有元素，而只想改变其中一部分，那么可以使用可选的起始位置参数与结束位置参数（不包括结束位置的那个元素），想这样：

```js
let numbers = [1, 2, 3, 4];

numbers.fill(1, 2);

console.log(numbers.toString());    // 1,2,1,1

numbers.fill(0, 1, 3);

console.log(numbers.toString());    // 1,0,0,1
```

当进行 numbers.fill(1,2) 调用时，第二个参数 2 指定从数组索引值为 2 的元素（即数组的第 3 个元素）开始填充，而此时没有指定第三个参数，因此结束位置默认为 numbers 数组的长度，意味着该数组的最后两个元素会被填充为 1 。而 numbers.fill(0, 1, 3) 调用则将该数组索引值为 1 与 2 的元素填充为 0 。在调用 fill() 方法时指定第二个和第三个参数，允许你一次性填充数组中多个元素，避免改写整个数组。

> 如果提供的起始位置或结束位置为负数，则它们会被加上数组的长度来算出最终的位置。例如：将起始位置指定为 -1 ，就等于是 array.length - 1 ，这里的 array 指的是 fill() 方法所要处理的数组。


###  copyWithin() 方法

copyWithin() 方法与 fill() 类似，可以一次性修改数组的多个元素。不过，与 fill() 使用单个值来填充数组不同，copyWithin() 方法允许你在数组内部复制自身元素。为此你需要传递两个参数给 copyWithin() 方法：从什么位置开始进行填充，以及被用来复制的数据的其实位置索引。

例如，将数组的前两个元素复制到数组的最后两个位置，你可以这么做：

```js
let numbers = [1, 2, 3, 4];

// paste values into array starting at index 2
// copy values from array starting at index 0
numbers.copyWithin(2, 0);

console.log(numbers.toString());    // 1,2,1,2
```

这段代码从 numbers 数组索引值为 2 的元素开始进行填充，因此索引值为 2 与 3 的元素都会被覆盖；调用 copyWithin() 方法时将第二个参数指定为 0 ，表示被复制的数据从索引值为 0 的元素开始，一直到没有元素可供复制为止。

默认情况下， copyWithin() 方法总是会一直复制到数组末尾，不过你还可以提供一个可选参数来限制到底有多少元素会被覆盖。这第三个参数指定了复制停止的位置（不包含该位置自身），例子：

```js
let numbers = [1, 2, 3, 4];

// paste values into array starting at index 2
// copy values from array starting at index 0
// stop copying values when you hit index 1
numbers.copyWithin(2, 0, 1);

console.log(numbers.toString());    // 1,2,1,4
```

在这个例子中，因为可选的结束位置参数被指定为 1 ，于是只有索引值为 0 的元素被复制了，而该数组的最后一个元素并没有被修改。


> 类似于 fill() 方法，如果你向 copyWithin() 方法传递负数参数，数组的长度会自动 被加到该参数的值上，以便算出正确的索引位置。

 fill() 和 copyWithin() 方法初看起来不是那么有用，因为它们起源于类型化数组的需求，而出于功能一致性的目的才被添加到常规数组上。不过，接下来的小节你就会学到如何用类 型化数组来按位操作数值，此时这两个方法就会变得非常有用了。

## 类型数组


Typed arrays are special-purpose arrays designed to work with numeric types (not all types, as the name might imply). The origin of typed arrays can be traced to WebGL, a port of Open GL ES 2.0 designed for use in web pages with the <canvas> element. Typed arrays were created as part of the port to provide fast bitwise arithmetic in JavaScript.

Arithmetic on native JavaScript numbers was too slow for WebGL because the numbers were stored in a 64-bit floating-point format and converted to 32-bit integers as needed. Typed arrays were introduced to circumvent this limitation and provide better performance for arithmetic operations. The concept is that any single number can be treated like an array of bits and thus can use the familiar methods available on JavaScript arrays.

ECMAScript 6 adopted typed arrays as a formal part of the language to ensure better compatibility across JavaScript engines and interoperability with JavaScript arrays. While the ECMAScript 6 version of typed arrays is not exactly the same as the WebGL version, there are enough similarities to make the ECMAScript 6 version an evolution of the WebGL version rather than a different approach.

<br />

### 数值数据类型


JavaScript numbers are stored in IEEE 754 format, which uses 64 bits to store a floating-point representation of the number. This format represents both integers and floats in JavaScript, with conversion between the two formats happening frequently as numbers change. Typed arrays allow the storage and manipulation of eight different numeric types:

1. Signed 8-bit integer (int8)
2. Unsigned 8-bit integer (uint8)
3. Signed 16-bit integer (int16)
4. Unsigned 16-bit integer (uint16)
5. Signed 32-bit integer (int32)
6. Unsigned 32-bit integer (uint32)
7. 32-bit float (float32)
8. 64-bit float (float64)

If you represent a number that fits in an int8 as a normal JavaScript number, you’ll waste 56 bits. Those bits might better be used to store additional int8 values or any other number that requires less than 56 bits. Using bits more efficiently is one of the use cases typed arrays address.

All of the operations and objects related to typed arrays are centered around these eight data types. In order to use them, though, you’ll need to create an array buffer to store the data.

<br />

> In this book, I will refer to these types by the abbreviations I showed in parentheses. Those abbreviations don’t appear in actual JavaScript code; they’re just a shorthand for the much longer descriptions.



### 数据缓冲区


The foundation for all typed arrays is an array buffer, which is a memory location that can contain a specified number of bytes. Creating an array buffer is akin to calling malloc() in C to allocate memory without specifying what the memory block contains. You can create an array buffer by using the ArrayBuffer constructor as follows:

```js
let buffer = new ArrayBuffer(10);   // allocate 10 bytes
```

Just pass the number of bytes the array buffer should contain when you call the constructor. This let statement creates an array buffer 10 bytes long. Once an array buffer is created, you can retrieve the number of bytes in it by checking the byteLength property:

```js
let buffer = new ArrayBuffer(10);   // allocate 10 bytes
console.log(buffer.byteLength);     // 10
```

You can also use the slice() method to create a new array buffer that contains part of an existing array buffer. The slice() method works like the slice() method on arrays: you pass it the start index and end index as arguments, and it returns a new ArrayBuffer instance comprised of those elements from the original. For example:

```js
let buffer = new ArrayBuffer(10);   // allocate 10 bytes


let buffer2 = buffer.slice(4, 6);
console.log(buffer2.byteLength);    // 2
```

In this code, buffer2 is created by extracting the bytes at indices 4 and 5. Just like when you call the array version of this method, the second argument to slice() is exclusive.

Of course, creating a storage location isn’t very helpful without being able to write data into that location. To do so, you’ll need to create a view.

<br />

> An array buffer always represents the exact number of bytes specified when it was created. You can change the data contained within an array buffer, but never the size of the array buffer itself.



### 使用视图操作数组缓冲区


Array buffers represent memory locations, and views are the interfaces you’ll use to manipulate that memory. A view operates on an array buffer or a subset of an array buffer’s bytes, reading and writing data in one of the numeric data types. The DataView type is a generic view on an array buffer that allows you to operate on all eight numeric data types.

To use a DataView, first create an instance of ArrayBuffer and use it to create a new DataView. Here’s an example:

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer);
```

The view object in this example has access to all 10 bytes in buffer. You can also create a view over just a portion of a buffer. Just provide a byte offset and, optionally, the number of bytes to include from that offset. When a number of bytes isn’t included, theDataView will go from the offset to the end of the buffer by default. For example:

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer, 5, 2);      // cover bytes 5 and 6
```

Here, view operates only on the bytes at indices 5 and 6. This approach allows you to create several views over the same array buffer, which can be useful if you want to use a single memory location for an entire application rather than dynamically allocating space as needed.

<br />

##### Retrieving View Information


You can retrieve information about a view by fetching the following read-only properties:

* buffer - The array buffer that the view is tied to
* byteOffset - The second argument to the DataView constructor, if provided (0 by default)
* byteLength - The third argument to the DataView constructor, if provided (the buffer’s byteLength by default)

Using these properties, you can inspect exactly where a view is operating, like this:

```js
let buffer = new ArrayBuffer(10),
    view1 = new DataView(buffer),           // cover all bytes
    view2 = new DataView(buffer, 5, 2);     // cover bytes 5 and 6

console.log(view1.buffer === buffer);       // true
console.log(view2.buffer === buffer);       // true
console.log(view1.byteOffset);              // 0
console.log(view2.byteOffset);              // 5
console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2
```

This code creates view1, a view over the entire array buffer, and view2, which operates on a small section of the array buffer. These views have equivalent buffer properties because both work on the same array buffer. The byteOffset and byteLength are different for each view, however. They reflect the portion of the array buffer where each view operates.

Of course, reading information about memory isn’t very useful on its own. You need to write data into and read data out of that memory to get any benefit.

<br />

##### Reading and Writing Data


For each of JavaScript’s eight numeric data types, the DataView prototype has a method to write data and a method to read data from an array buffer. The method names all begin with either “set” or “get” and are followed by the data type abbreviation. For instance, here’s a list of the read and write methods that can operate on int8 and uint8 values:

* getInt8(byteOffset, littleEndian) - Read an int8 starting at byteOffset
* setInt8(byteOffset, value, littleEndian) - Write an int8 starting at byteOffset
* getUint8(byteOffset, littleEndian) - Read an uint8 starting at byteOffset
* setUint8(byteOffset, value, littleEndian) - Write an uint8 starting at byteOffset

The “get” methods accept two arguments: the byte offset to read from and an optional boolean indicating whether the value should be read as little-endian. (Little-endian means the least significant byte is at byte 0, instead of in the last byte.) The “set” methods accept three arguments: the byte offset to write at, the value to write, and an optional boolean indicating whether the value should be stored in little-endian format.

Though I’ve only shown the methods you can use with 8-bit values, the same methods exist for operating on 16- and 32-bit values. Just replace the 8 in each name with 16 or 32. Alongside all those integer methods, DataView also has the following read and write methods for floating point numbers:

* getFloat32(byteOffset, littleEndian) - Read a float32 starting at byteOffset
* setFloat32(byteOffset, value, littleEndian) - Write a float32 starting at byteOffset
* getFloat64(byteOffset, littleEndian) - Read a float64 starting at byteOffset
* setFloat64(byteOffset, value, littleEndian) - Write a float64 starting at byteOffset

To see a “set” and a “get” method in action, consider the following example:

```js
let buffer = new ArrayBuffer(2),
    view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt8(0));       // 5
console.log(view.getInt8(1));       // -1
```

This code uses a two-byte array buffer to store two int8 values. The first value is set at offset 0 and the second is at offset 1, reflecting that each value spans a full byte (8 bits). Those values are later retrieved from their positions with the getInt8() method. While this example uses int8 values, you can use any of the eight numeric types with their corresponding methods.

Views are interesting because they allow you to read and write in any format at any point in time, regardless of how data was previously stored. For instance, writing two int8 values and reading the buffer with an int16 method works just fine, as in this example:

```js
let buffer = new ArrayBuffer(2),
    view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt16(0));      // 1535
console.log(view.getInt8(0));       // 5
console.log(view.getInt8(1));       // -1
```

The call to view.getInt16(0) reads all bytes in the view and interprets those bytes as the number 1535. To understand why this happens, take a look at Figure 10-1, which shows what each setInt8() line does to the array buffer.

```js
new ArrayBuffer(2)      0000000000000000
view.setInt8(0, 5);     0000010100000000
view.setInt8(1, -1);    0000010111111111
```

The array buffer starts with 16 bits that are all zero. Writing 5 to the first byte with setInt8() introduces a couple of 1s (in 8-bit representation, 5 is 00000101). Writing -1 to the second byte sets all bits in that byte to 1, which is the two’s complement representation of -1. After the second setInt8() call, the array buffer contains 16 bits, and getInt16() reads those bits as a single 16-bit integer, which is 1535 in decimal.

The DataView object is perfect for use cases that mix different data types in this way. However, if you’re only using one specific data type, then the type-specific views are a better choice.

<br />

##### Typed Arrays Are Views


ECMAScript 6 typed arrays are actually type-specific views for array buffers. Instead of using a generic DataView object to operate on an array buffer, you can use objects that enforce specific data types. There are eight type-specific views corresponding to the eight numeric data types, plus an additional option for uint8 values.

Table 10-1 shows an abbreviated version of the complete list of type-specific views from section 22.2 of the ECMAScript 6 specification.

|  Constructor Name  |  Element Size (in bytes)  |  Description                                  |  Equivalent C Type  |
|  :--------------:  |  :---------------------:  |  :-----------------------------------------:  |  :---------------:  |
|  Int8Array         |  1                        |  8-bit two’s complement signed integer        |  signed char        |
|  Uint8Array        |  1                        |  8-bit unsigned integer                       |  unsigned char      |
|  Uint8ClampedArray |  1                        |  8-bit unsigned integer (clamped conversion)  |  unsigned char      |
|  Int16Array        |  2                        |  16-bit two’s complement signed integer       |  short              |
|  Uint16Array       |  2                        |  16-bit unsigned integer                      |  unsigned short     |
|  Int32Array        |  4                        |  32-bit two’s complement signed integer       |  int                |
|  Uint32Array       |  4                        |  32-bit unsigned integer                      |  int                |
|  Float32Array      |  4                        |  32-bit IEEE floating point                   |  float              |
|  Float64Array      |  8                        |  64-bit IEEE floating point                   |  double             |

The left column lists the typed array constructors, and the other columns describe the data each typed array can contain. A Uint8ClampedArray is the same as a Uint8Array unless values in the array buffer are less than 0 or greater than 255. A Uint8ClampedArray converts values lower than 0 to 0 (-1 becomes 0, for instance) and converts values higher than 255 to 255 (so 300 becomes 255).

Typed array operations only work on a particular type of data. For example, all operations on Int8Array use int8 values. The size of an element in a typed array also depends on the type of array. While an element in an Int8Array is a single byte long, Float64Array uses eight bytes per element. Fortunately, the elements are accessed using numeric indices just like regular arrays, allowing you to avoid the somewhat awkward calls to the “set” and “get” methods of DataView.

<br />

> #### Element Size

> Each typed array is made up of a number of elements, and the element size is the number of bytes each element represents. This value is stored on a BYTES_PER_ELEMENT property on each constructor and each instance, so you can easily query the element size:

```js
console.log(UInt8Array.BYTES_PER_ELEMENT);      // 1
console.log(UInt16Array.BYTES_PER_ELEMENT);     // 2

let ints = new Int8Array(5);
console.log(ints.BYTES_PER_ELEMENT);            // 1
```

<br />

##### Creating Type-Specific Views


Typed array constructors accept multiple types of arguments, so there are a few ways to create typed arrays. First, you can create a new typed array by passing the same arguments DataView takes (an array buffer, an optional byte offset, and an optional byte length). For example:

```js
let buffer = new ArrayBuffer(10),
    view1 = new Int8Array(buffer),
    view2 = new Int8Array(buffer, 5, 2);

console.log(view1.buffer === buffer);       // true
console.log(view2.buffer === buffer);       // true
console.log(view1.byteOffset);              // 0
console.log(view2.byteOffset);              // 5
console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2
```

In this code, the two views are both Int8Array instances that use buffer. Both view1 and view2 have the same buffer, byteOffset, and byteLength properties that exist on DataView instances. It’s easy to switch to using a typed array wherever you use a DataView so long as you only work with one numeric type.

The second way to create a typed array is to pass a single number to the constructor. That number represents the number of elements (not bytes) to allocate to the array. The constructor will create a new buffer with the correct number of bytes to represent that number of array elements, and you can access the number of elements in the array by using the length property. For example:

```js
let ints = new Int16Array(2),
    floats = new Float32Array(5);

console.log(ints.byteLength);       // 4
console.log(ints.length);           // 2

console.log(floats.byteLength);     // 20
console.log(floats.length);         // 5
```

The ints array is created with space for two elements. Each 16-bit integer requires two bytes per value, so the array is allocated four bytes. The floats array is created to hold five elements, so the number of bytes required is 20 (four bytes per element). In both cases, a new buffer is created and can be accessed using the buffer property if necessary.

<br />

> **NOTE**: If no argument is passed to a typed array constructor, the constructor acts as if 0 was passed. This creates a typed array that cannot hold data because zero bytes are allocated to the buffer.

<br />

The third way to create a typed array is to pass an object as the only argument to the constructor. The object can be any of the following:

* **A Typed Array** - Each element is copied into a new element on the new typed array. For example, if you pass an int8 to the Int16Array constructor, the int8 values would be copied into an int16 array. The new typed array has a different array buffer than the one that was passed in.

* **An Iterable** - The object’s iterator is called to retrieve the items to insert into the typed array. The constructor will throw an error if any elements are invalid for the view type.

* **An Array** - The elements of the array are copied into a new typed array. The constructor will throw an error if any elements are invalid for the type.

* **An Array-Like Object** - Behaves the same as an array.

In each of these cases, a new typed array is created with the data from the source object. This can be especially useful when you want to initialize a typed array with some values, like this:

```js
let ints1 = new Int16Array([25, 50]),
    ints2 = new Int32Array(ints1);

console.log(ints1.buffer === ints2.buffer);     // false

console.log(ints1.byteLength);      // 4
console.log(ints1.length);          // 2
console.log(ints1[0]);              // 25
console.log(ints1[1]);              // 50

console.log(ints2.byteLength);      // 8
console.log(ints2.length);          // 2
console.log(ints2[0]);              // 25
console.log(ints2[1]);              // 50
```

This example creates an Int16Array and initializes it with an array of two values. Then, an Int32Array is created and passed the Int16Array. The values 25 and 50 are copied from ints1 into ints2 as the two typed arrays have completely separate buffers. The same numbers are represented in both typed arrays, but ints2 has eight bytes to represent the data while ints1 has only four.


## 类型数组与普通数组的相似之处


Typed arrays and regular arrays are similar in several ways, and as you’ve already seen in this chapter, typed arrays can be used like regular arrays in many situations. For instance, you can check how many elements are in a typed array using the length property, and you can access a typed array’s elements directly using numeric indices. For example:

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length);          // 2
console.log(ints[0]);              // 25
console.log(ints[1]);              // 50

ints[0] = 1;
ints[1] = 2;

console.log(ints[0]);              // 1
console.log(ints[1]);              // 2
```

In this code, a new Int16Array with two items is created. The items are read from and written to using their numeric indices, and those values are automatically stored and converted into int16 values as part of the operation. The similarities don’t end there, though.

<br />

> Unlike regular arrays, you cannot change the size of a typed array using the length property. The length property is not writable, so any attempt to change it is ignored in non-strict mode and throws an error in strict mode.



### 通用方法


Typed arrays also include a large number of methods that are functionally equivalent to regular array methods. You can use the following array methods on typed arrays:

* copyWithin()
* entries()
* fill()
* filter()
* find()
* findIndex()
* forEach()
* indexOf()
* join()
* keys()
* lastIndexOf()
* map()
* reduce()
* reduceRight()
* reverse()
* slice()
* some()
* sort()
* values()

Keep in mind that while these methods act like their counterparts on Array.prototype, they are not exactly the same. The typed array methods have additional checks for numeric type safety and, when an array is returned, will return a typed array instead of a regular array (due to Symbol.species). Here’s a simple example to demonstrate the difference:

```js
let ints = new Int16Array([25, 50]),
    mapped = ints.map(v => v * 2);

console.log(mapped.length);        // 2
console.log(mapped[0]);            // 50
console.log(mapped[1]);            // 100

console.log(mapped instanceof Int16Array);  // true
```

This code uses the map() method to create a new array based on the values in ints. The mapping function doubles each value in the array and returns a new Int16Array.



### 相同的迭代器


Typed arrays have the same three iterators as regular arrays, too. Those are the entries() method, the keys() method, and the values() method. That means you can use the spread operator and for-of loops with typed arrays just like you would with regular arrays. For example:

```js
let ints = new Int16Array([25, 50]),
    intsArray = [...ints];

console.log(intsArray instanceof Array);    // true
console.log(intsArray[0]);                  // 25
console.log(intsArray[1]);                  // 50
```

This code creates a new array called intsArray containing the same data as the typed array ints. As with other iterables, the spread operator makes converting typed arrays into regular arrays easy.

<br />

### of() 和 from() 方法


Lastly, all typed arrays have static of() and from() methods that work like the Array.of() and Array.from() methods. The difference is that the methods on typed arrays return a typed array instead of a regular array. Here are some examples that use these methods to create typed arrays:

```js
let ints = Int16Array.of(25, 50),
    floats = Float32Array.from([1.5, 2.5]);

console.log(ints instanceof Int16Array);        // true
console.log(floats instanceof Float32Array);    // true

console.log(ints.length);       // 2
console.log(ints[0]);           // 25
console.log(ints[1]);           // 50

console.log(floats.length);     // 2
console.log(floats[0]);         // 1.5
console.log(floats[1]);         // 2.5
```

The of() and from() methods in this example are used to create an Int16Array and a Float32Array, respectively. These methods ensure that typed arrays can be created just as easily as regular arrays.



## 类型数组与普通数组的差异


The most importance difference between typed arrays and regular arrays is that typed arrays are not regular arrays. Typed arrays don’t inherit from Array and Array.isArray() returns false when passed a typed array. For example:

```js
let ints = new Int16Array([25, 50]);

console.log(ints instanceof Array);     // false
console.log(Array.isArray(ints));       // false
```

Since the ints variable is a typed array, it isn’t an instance of Array and cannot otherwise be identified as an array. This distinction is important because while typed arrays and regular arrays are similar, there are many ways in which typed arrays behave differently.



### 行为差异


While regular arrays can grow and shrink as you interact with them, typed arrays always remain the same size. You cannot assign a value to a nonexistent numeric index in a typed array like you can with regular arrays, as typed arrays ignore the operation. Here’s an example:

```js
let ints = new Int16Array([25, 50]);

console.log(ints.length);          // 2
console.log(ints[0]);              // 25
console.log(ints[1]);              // 50

ints[2] = 5;

console.log(ints.length);          // 2
console.log(ints[2]);              // undefined
```

Despite assigning 5 to the numeric index 2 in this example, the ints array does not grow at all. The length remains the same and the value is thrown away.

Typed arrays also have checks to ensure that only valid data types are used. Zero is used in place of any invalid values. For example:

```js
let ints = new Int16Array(["hi"]);

console.log(ints.length);       // 1
console.log(ints[0]);           // 0
```

This code attempts to use the string value "hi" in an Int16Array. Of course, strings are invalid data types in typed arrays, so the value is inserted as 0 instead. The length of the array is still one, and even though the ints[0] slot exists, it just contains 0.

All methods that modify values in a typed array enforce the same restriction. For example, if the function passed to map() returns an invalid value for the typed array, then 0 is used instead:

```js
let ints = new Int16Array([25, 50]),
    mapped = ints.map(v => "hi");

console.log(mapped.length);        // 2
console.log(mapped[0]);            // 0
console.log(mapped[1]);            // 0

console.log(mapped instanceof Int16Array);  // true
console.log(mapped instanceof Array);       // false
```

Since the string value "hi" isn’t a 16-bit integer, it’s replaced with 0 in the resulting array. Thanks to this error correction behavior, typed array methods don’t have to worry about throwing errors when invalid data is present, because there will never be invalid data in the array.



### 缺失的方法


While typed arrays do have many of the same methods as regular arrays, they also lack several array methods. The following methods are not available on typed arrays:

* concat()
* pop()
* push()
* shift()
* splice()
* unshift()

Except for the concat() method, the methods in this list can change the size of an array. Typed arrays can’t change size, which is why these aren’t available for typed arrays. The concat() method isn’t available because the result of concatenating two typed arrays (especially if they deal with different data types) would be uncertain, and that would go against the reason for using typed arrays in the first place.



### 附加的方法


Finally, typed arrays methods have two methods not present on regular arrays: the set() and subarray() methods. These two methods are opposites in that set() copies another array into an existing typed array, whereas subarray() extracts part of an existing typed array into a new typed array.

The set() method accepts an array (either typed or regular) and an optional offset at which to insert the data; if you pass nothing, the offset defaults to zero. The data from the array argument is copied into the destination typed array while ensuring only valid data types are used. Here’s an example:

```js
let ints = new Int16Array(4);

ints.set([25, 50]);
ints.set([75, 100], 2);

console.log(ints.toString());   // 25,50,75,100
```

This code creates an Int16Array with four elements. The first call to set() copies two values to the first and second elements in the array. The second call to set() uses an offset of 2 to indicate that the values should be placed in the array starting at the third element.

The subarray() method accepts an optional start and end index (the end index is exclusive, as in the slice() method) and returns a new typed array. You can also omit both arguments to create a clone of the typed array. For example:

```js
let ints = new Int16Array([25, 50, 75, 100]),
    subints1 = ints.subarray(),
    subints2 = ints.subarray(2),
    subints3 = ints.subarray(1, 3);

console.log(subints1.toString());   // 25,50,75,100
console.log(subints2.toString());   // 75,100
console.log(subints3.toString());   // 50,75
```

Three typed arrays are created from the original ints array in this example. The subints1 array is a clone of ints that contains the same information. Since the subints2 array copies data starting from index 2, it only contains the last two elements of the ints array (75 and 100). The subints3 array contains only the middle two elements of the ints array, as subarray() was called with both a start and an end index.


## 总结

ECMAScript 6 continues the work of ECMAScript 5 by making arrays more useful. There are two more ways to create arrays: the Array.of() and Array.from() methods. The Array.from() method can also convert iterables and array-like objects into arrays. Both methods are inherited by derived array classes and use the Symbol.species property to determine what type of value should be returned (other inherited methods also use Symbol.species when returning an array).

There are also several new methods on arrays. The fill() and copyWithin() methods allow you to alter array elements in-place. The find() and findIndex() methods are useful for finding the first element in an array that matches some criteria. The former returns the first element that fits the criteria, and the latter returns the element’s index.

Typed arrays are not technically arrays, as they do not inherit from Array, but they do look and behave a lot like arrays. Typed arrays contain one of eight different numeric data types and are built upon ArrayBuffer objects that represent the underlying bits of a number or series of numbers. Typed arrays are a more efficient way of doing bitwise arithmetic because the values are not converted back and forth between formats, as is the case with the JavaScript number type.

