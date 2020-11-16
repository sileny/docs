# base64


- [btoa](#btoa)
- [atob](#atob)
- [abab按照rfc文档实现的方案](#abab)
- [ArrayBuffer转base64详解](#ArrayBuffer转base64详解)
- [base64转换成Uint8Array](#base64转换成Uint8Array)



## btoa

转换成 base64

- browser

浏览器环境下转换成 `base64`

```js
bota('HELLO'); // "SEVMTE8="
```

- node

node 环境下转换

```js
module.exports = function btoa(str) {
  return new Buffer(str).toString('base64')
};

bota('HELLO'); // "SEVMTE8="
```

## atob

- browser
```js
atob("SEVMTE8="); // "HELLO"
```

- node
```js
module.exports = function atob(str) {
  return Buffer.from(str, 'base64').toString('binary')
};

atob("SEVMTE8="); // "HELLO"
```


## abab

参考 rfc 文档实现的方案

- 实现 `btoa`
```js
// btoa.js

"use strict";

/**
 * btoa() as defined by the HTML and Infra specs, which mostly just references
 * RFC 4648.
 */
function btoa(s) {
  let i;
  // String conversion as required by Web IDL.
  s = `${s}`;
  // "The btoa() method must throw an "InvalidCharacterError" DOMException if
  // data contains any character whose code point is greater than U+00FF."
  for (i = 0; i < s.length; i++) {
    if (s.charCodeAt(i) > 255) {
      return null;
    }
  }
  let out = "";
  for (i = 0; i < s.length; i += 3) {
    const groupsOfSix = [undefined, undefined, undefined, undefined];
    groupsOfSix[0] = s.charCodeAt(i) >> 2;
    groupsOfSix[1] = (s.charCodeAt(i) & 0x03) << 4;
    if (s.length > i + 1) {
      groupsOfSix[1] |= s.charCodeAt(i + 1) >> 4;
      groupsOfSix[2] = (s.charCodeAt(i + 1) & 0x0f) << 2;
    }
    if (s.length > i + 2) {
      groupsOfSix[2] |= s.charCodeAt(i + 2) >> 6;
      groupsOfSix[3] = s.charCodeAt(i + 2) & 0x3f;
    }
    for (let j = 0; j < groupsOfSix.length; j++) {
      if (typeof groupsOfSix[j] === "undefined") {
        out += "=";
      } else {
        out += btoaLookup(groupsOfSix[j]);
      }
    }
  }
  return out;
}

/**
 * Lookup table for btoa(), which converts a six-bit number into the
 * corresponding ASCII character.
 */
const keystr =
  "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

function btoaLookup(index) {
  if (index >= 0 && index < 64) {
    return keystr[index];
  }

  // Throw INVALID_CHARACTER_ERR exception here -- won't be hit in the tests.
  return undefined;
}

module.exports = btoa;
```

- 实现 `atob`
```js
// atob.js

"use strict";

/**
 * Implementation of atob() according to the HTML and Infra specs, except that
 * instead of throwing INVALID_CHARACTER_ERR we return null.
 */
function atob(data) {
  // Web IDL requires DOMStrings to just be converted using ECMAScript
  // ToString, which in our case amounts to using a template literal.
  data = `${data}`;
  // "Remove all ASCII whitespace from data."
  data = data.replace(/[ \t\n\f\r]/g, "");
  // "If data's length divides by 4 leaving no remainder, then: if data ends
  // with one or two U+003D (=) code points, then remove them from data."
  if (data.length % 4 === 0) {
    data = data.replace(/==?$/, "");
  }
  // "If data's length divides by 4 leaving a remainder of 1, then return
  // failure."
  //
  // "If data contains a code point that is not one of
  //
  // U+002B (+)
  // U+002F (/)
  // ASCII alphanumeric
  //
  // then return failure."
  if (data.length % 4 === 1 || /[^+/0-9A-Za-z]/.test(data)) {
    return null;
  }
  // "Let output be an empty byte sequence."
  let output = "";
  // "Let buffer be an empty buffer that can have bits appended to it."
  //
  // We append bits via left-shift and or.  accumulatedBits is used to track
  // when we've gotten to 24 bits.
  let buffer = 0;
  let accumulatedBits = 0;
  // "Let position be a position variable for data, initially pointing at the
  // start of data."
  //
  // "While position does not point past the end of data:"
  for (let i = 0; i < data.length; i++) {
    // "Find the code point pointed to by position in the second column of
    // Table 1: The Base 64 Alphabet of RFC 4648. Let n be the number given in
    // the first cell of the same row.
    //
    // "Append to buffer the six bits corresponding to n, most significant bit
    // first."
    //
    // atobLookup() implements the table from RFC 4648.
    buffer <<= 6;
    buffer |= atobLookup(data[i]);
    accumulatedBits += 6;
    // "If buffer has accumulated 24 bits, interpret them as three 8-bit
    // big-endian numbers. Append three bytes with values equal to those
    // numbers to output, in the same order, and then empty buffer."
    if (accumulatedBits === 24) {
      output += String.fromCharCode((buffer & 0xff0000) >> 16);
      output += String.fromCharCode((buffer & 0xff00) >> 8);
      output += String.fromCharCode(buffer & 0xff);
      buffer = accumulatedBits = 0;
    }
    // "Advance position by 1."
  }
  // "If buffer is not empty, it contains either 12 or 18 bits. If it contains
  // 12 bits, then discard the last four and interpret the remaining eight as
  // an 8-bit big-endian number. If it contains 18 bits, then discard the last
  // two and interpret the remaining 16 as two 8-bit big-endian numbers. Append
  // the one or two bytes with values equal to those one or two numbers to
  // output, in the same order."
  if (accumulatedBits === 12) {
    buffer >>= 4;
    output += String.fromCharCode(buffer);
  } else if (accumulatedBits === 18) {
    buffer >>= 2;
    output += String.fromCharCode((buffer & 0xff00) >> 8);
    output += String.fromCharCode(buffer & 0xff);
  }
  // "Return output."
  return output;
}
/**
 * A lookup table for atob(), which converts an ASCII character to the
 * corresponding six-bit number.
 */

const keystr =
  "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

function atobLookup(chr) {
  const index = keystr.indexOf(chr);
  // Throw exception if character is not in the lookup string; should not be hit in tests
  return index < 0 ? undefined : index;
}

module.exports = atob;
```

先将方面的代码进行整合
```js
const abab = {
  atob(data) {
    // Web IDL requires DOMStrings to just be converted using ECMAScript
    // ToString, which in our case amounts to using a template literal.
    data = `${data}`;
    // "Remove all ASCII whitespace from data."
    data = data.replace(/[ \t\n\f\r]/g, "");
    // "If data's length divides by 4 leaving no remainder, then: if data ends
    // with one or two U+003D (=) code points, then remove them from data."
    if (data.length % 4 === 0) {
      data = data.replace(/==?$/, "");
    }
    // "If data's length divides by 4 leaving a remainder of 1, then return
    // failure."
    //
    // "If data contains a code point that is not one of
    //
    // U+002B (+)
    // U+002F (/)
    // ASCII alphanumeric
    //
    // then return failure."
    if (data.length % 4 === 1 || /[^+/0-9A-Za-z]/.test(data)) {
      return null;
    }
    // "Let output be an empty byte sequence."
    let output = "";
    // "Let buffer be an empty buffer that can have bits appended to it."
    //
    // We append bits via left-shift and or.  accumulatedBits is used to track
    // when we've gotten to 24 bits.
    let buffer = 0;
    let accumulatedBits = 0;
    // "Let position be a position variable for data, initially pointing at the
    // start of data."
    //
    // "While position does not point past the end of data:"
    for (let i = 0; i < data.length; i++) {
      // "Find the code point pointed to by position in the second column of
      // Table 1: The Base 64 Alphabet of RFC 4648. Let n be the number given in
      // the first cell of the same row.
      //
      // "Append to buffer the six bits corresponding to n, most significant bit
      // first."
      //
      // atobLookup() implements the table from RFC 4648.
      buffer <<= 6;
      buffer |= atobLookup(data[i]);
      accumulatedBits += 6;
      // "If buffer has accumulated 24 bits, interpret them as three 8-bit
      // big-endian numbers. Append three bytes with values equal to those
      // numbers to output, in the same order, and then empty buffer."
      if (accumulatedBits === 24) {
        output += String.fromCharCode((buffer & 0xff0000) >> 16);
        output += String.fromCharCode((buffer & 0xff00) >> 8);
        output += String.fromCharCode(buffer & 0xff);
        buffer = accumulatedBits = 0;
      }
      // "Advance position by 1."
    }
    // "If buffer is not empty, it contains either 12 or 18 bits. If it contains
    // 12 bits, then discard the last four and interpret the remaining eight as
    // an 8-bit big-endian number. If it contains 18 bits, then discard the last
    // two and interpret the remaining 16 as two 8-bit big-endian numbers. Append
    // the one or two bytes with values equal to those one or two numbers to
    // output, in the same order."
    if (accumulatedBits === 12) {
      buffer >>= 4;
      output += String.fromCharCode(buffer);
    } else if (accumulatedBits === 18) {
      buffer >>= 2;
      output += String.fromCharCode((buffer & 0xff00) >> 8);
      output += String.fromCharCode(buffer & 0xff);
    }
    // "Return output."
    return output;
  },
  btoa(s) {
    let i;
    // String conversion as required by Web IDL.
    s = `${s}`;
    // "The btoa() method must throw an "InvalidCharacterError" DOMException if
    // data contains any character whose code point is greater than U+00FF."
    for (i = 0; i < s.length; i++) {
      if (s.charCodeAt(i) > 255) {
        return null;
      }
    }
    let out = "";
    for (i = 0; i < s.length; i += 3) {
      const groupsOfSix = [undefined, undefined, undefined, undefined];
      groupsOfSix[0] = s.charCodeAt(i) >> 2;
      groupsOfSix[1] = (s.charCodeAt(i) & 0x03) << 4;
      if (s.length > i + 1) {
        groupsOfSix[1] |= s.charCodeAt(i + 1) >> 4;
        groupsOfSix[2] = (s.charCodeAt(i + 1) & 0x0f) << 2;
      }
      if (s.length > i + 2) {
        groupsOfSix[2] |= s.charCodeAt(i + 2) >> 6;
        groupsOfSix[3] = s.charCodeAt(i + 2) & 0x3f;
      }
      for (let j = 0; j < groupsOfSix.length; j++) {
        if (typeof groupsOfSix[j] === "undefined") {
          out += "=";
        } else {
          out += btoaLookup(groupsOfSix[j]);
        }
      }
    }
    return out;
  }
}
```

做了代码性能对比
```
console.time()
abab.btoa('HELLO');
console.timeEnd();
default: 0.129150390625 ms

console.time()
window.btoa('HELLO');
console.timeEnd();
default: 0.009033203125 ms

console.time()
abab.btoa('HELLO');
console.timeEnd();
default: 0.02880859375 ms

console.time()
window.btoa('HELLO');
console.timeEnd();
```


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


