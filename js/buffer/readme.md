- # buffer

- [string-to-array-buffer](#string-to-array-buffer)
- []


## string-to-array-buffer

```js
function str2ab(str) {
  var buf = new ArrayBuffer(str.length * 2); // 每个字符占用2个字节
  var bufView = new Uint16Array(buf);
  for (var i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

上面的 `new ArrayBuffer(str.length * 2)` 在内存中指定的字节数，表水创建了一个 `str.length * 2` byte 的 `ArrayBuffer`

打印
```
str2ab('hello')
```

控制台输出
```
ArrayBuffer(10) {}
[[Int8Array]]: Int8Array(10) [104, 0, 101, 0, 108, 0, 108, 0, 111, 0]
[[Int16Array]]: Int16Array(5) [104, 101, 108, 108, 111]
[[Uint8Array]]: Uint8Array(10) [104, 0, 101, 0, 108, 0, 108, 0, 111, 0]
byteLength: 10
```

`hello` 从左到右，分别对应的是 `[104, 101, 108, 108, 111]` 里的几个数字

打印

```js
str2ab('HELLO')
```

输出如下
```
ArrayBuffer(10) {}
[[Int8Array]]: Int8Array(10) [72, 0, 69, 0, 76, 0, 76, 0, 79, 0]
[[Int16Array]]: Int16Array(5) [72, 69, 76, 76, 79]
[[Uint8Array]]: Uint8Array(10) [72, 0, 69, 0, 76, 0, 76, 0, 79, 0]
byteLength: 10
```

`HELLO` 从左到右，分别对应的是 `[72, 69, 76, 76, 79]` 里的几个数字


**如何判定数字与字母的对应关系**

```js
String.fromCharCode(69); // 'E'

'E'.charCodeAt(); // `E`对应的编码为69
'E'.codePointAt(); // `E`对应的编码为69
```


来看下面的例子
```js
ab2str(str2ab('hello')); // "h e l l o "
```
可以看到，一个字符对应了两个位置

将 `Uint16Array` 改为 `Uint8Array`，现在的方法体如下
```js
function str2ab(str) {
  var buf = new ArrayBuffer(str.length * 2); // 每个字符占用2个字节
  var bufView = new Uint8Array(buf);
  for (var i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

打印 `ab2str(str2ab('hello'))`，输出如下，
```
"hello      "
```

现在，将 `ArrayBuffer` 的空间尺寸改小一些
```js
function str2ab(str) {
  var buf = new ArrayBuffer(str.length); // 每个字符占用2个字节
  var bufView = new Uint8Array(buf);
  for (var i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

打印 `ab2str(str2ab('hello'))`，输出如下，
```
ab2str(str2ab('hello'))
```

为啥有区别，看下面
```
// 创建一个8-byte的ArrayBuffer
var b = new ArrayBuffer(8);
 
// 创建一个b的引用，类型是Int32，起始位置在0，结束位置为缓冲区尾部
var v1 = new Int32Array(b);
 
// 创建一个b的引用，类型是Uint8，起始位置在2，结束位置为缓冲区尾部
var v2 = new Uint8Array(b, 2);
 
// 创建一个b的引用，类型是Int16，起始位置在2，总长度为2
var v3 = new Int16Array(b, 2, 2);
```

`ArrayBuffer` 可以存放多种类型的数据，而不同类型的存储方式就是 `DataView`

**数组缓冲器视图DataView**

- 使用 `ArrayBuffer`（数组缓冲器类型）的一种特别的方式就是：用 `ArrayBuffer` 来创建数组缓冲器视图
- 最常见的视图是 `DataView`，通过它可以选择 `ArrayBuffer`中的一小段字节
- 可在创建 `DataView` 实例的时候传入一个 `ArrayBuffer`、一个可选的字节偏移量（从该字节开始选择）和一个可选的要选择的字节数
- `ArrayBuffer` 对象并没有提供任何读写内存的方法，而是允许在其上方建立 `视图`，从而插入与读取内存中的数据

||	ArrayBuffer |	Uint8Array数组
---|---|---
存储内容 |	放0和1组成的二进制数据 |	放数字、字符串、布尔值以及对象和数组等
存储位置 |	数据放在栈中（所以取数据较快） |	放在堆中
存储大小 |	初始化后固定大小 |	数组则可以自由增减


__类型化视图__ 一般也被称为 __类型化数组__，原因在于，它们除了元素必须是某种特定的数据类型外，与常规的数组无异。而且它们都继承了 `DataView`


```js
var arraybuffer = new ArrayBuffer(8); //实例化时，仅需传入字节数，表示，在内存中开辟了8byte的ArrayBuffer
ArrayBuffer.length;  // 1
arraybuffer.byteLength;  // 8
 
var int8a = new Int8Array(arraybuffer);   // 在ArrayBuffer上建立视图
ArrayBuffer.isView(int8a)  //return true  // 类方法ArrayBuffer.isView()判断某对象是否为视图
```



`DataView`类型如下：
- Int8Array：8位有符号整数，长度1个字节。
- Uint8Array：8位无符号整数，长度1个字节。
- Int16Array：16位有符号整数，长度2个字节。
- Uint16Array：16位无符号整数，长度2个字节。
- Int32Array：32位有符号整数，长度4个字节。
- Uint32Array：32位无符号整数，长度4个字节。
- Float32Array：32位浮点数，长度4个字节。
- Float64Array：64位浮点数，长度8个字节



