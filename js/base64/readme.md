# base64


- [ArrayBuffer转base64详解](#ArrayBuffer转base64详解)
- [base64转换成Uint8Array](#base64转换成Uint8Array)

## ArrayBuffer转base64详解

先贴代码：

```js
const base64String = window.btoa(String.fromCharCode(...new Uint8Array(buffer)))
```
看起来非常的简洁，优美。那么我们今天就来捋一捋，这个看似简单的背后，蕴含了怎样的原理


在JavaScript中，有一个很常用的引用数据类型 `Array`,你可以在里面放字符串、数字、对象、布尔值等等等等。它存放在堆中，可以自由增减。

而 `ArrayBuffer` 我们叫它类型化数组，它的诞生就是为了解决一个问题：**操作二进制数据**。

只由0和1组成的二进制数据往往是非常巨大的，上千个字节可以说司空见惯，传统的Array这时候处理起二进制数据起来就显得非常低效，所以 `ArrayBuffer` 出现了，它作为一块专用的内存区域存放在栈中，取数据非常快


通过new ArrayBuffer(10)初始化一个buffer实例
```js
let buffer = new ArrayBuffer(10);
console.log(buffer);

// 打印内容如下
// ArrayBuffer(10) {}
// [[Int8Array]]: Int8Array(10) [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
// [[Int16Array]]: Int16Array(5) [0, 0, 0, 0, 0]
// [[Uint8Array]]: Uint8Array(10) [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
// byteLength: 10
```

可以看到在ArrayBuffer中，主要存放了几个“视图”，`Int8Array` 表示8位有符号整数数组，`Int16Array` 表示16位有符号整数数组，`Uint8Array` 则表示8位无符号整数数组

当然，如果比如说我们想取出 `Int8Array` 这个数组来，是不能直接通过 `buffer.Int8Array` 来取的。这是因为 `ArrayBuffer` 不能直接通过下标去读写，需要把它转成一个类型化数组（`TypedArray`）

```js
const myTypedArray = new Uint8Array(buffer)
```

转化完之后，我们我们不仅可以通过下标去对类型化数组进行索引，也可以获取其 `length`

当然 `TypedArray` 仍与普通的 `Array` 存在细微的区别，那就是假设我们用超出边界的索引语法去获取数组元素时，TypedArray并不会去原型链中进行查找


拿到了这个类型化数组，可以把它转换成字符串，
```js
const str = String.fromCharCode(...myTypedArray)
```

用拓展运算符 `...` 把类型数组的代码单元解出来，一次性转完，得到一个普通的字符串

最后，借助一个 `window` 对象的方法，也就是 `btoa` 方法，它的作用是：把一个普通字符串编码成 `base64` 格式的字符串

后台把图片资源通过 `ArrayBuffer` 传给前端，这时候为了能正常显示，我们还需要在转化的 `base64` 字符串前面拼接上 `data:image/jpeg;base64,`

最终，可以得出这样一个函数：

```js
const arrayBufferToBase64Img = (buffer) => {
  const str = String.fromCharCode(...new Uint8Array(buffer));
  return `data:image/jpeg;base64,${window.btoa(str)}`;
}
```

>思路：得到一个ArrayBuffer ---> 转成类型化数组以正常读取 --> 转成普通字符串 --> 转成base64字符串


## base64转换成Uint8Array

```js
function base64oUint8Array(base64String) {
    let padding = '='.repeat((4 - base64String.length % 4) % 4);
    let base64 = (base64String + padding)
        .replace(/\-/g, '+')
        .replace(/_/g, '/');

    let rawData = window.atob(base64);
    let outputArray = new Uint8Array(rawData.length);

    for (var i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
}
```


