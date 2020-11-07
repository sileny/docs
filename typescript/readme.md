# typescript

- [apply](#apply)
- [setTimeout](#settimeout)
- [可索引类型](#可索引类型)
- [interface和type关键字](#interface和type关键字)
- [Objects/Functions](#Objects-Functions)
- [其他数据类型](#其他数据类型)
- [Extend](#Extend)
- [interface extends interface](#interface-extends-interface)
- [type extends type](#type-extends-type)
- [interface extends type](#interface-extends-type)
- [type extends interface](#type-extends-interface)
- [implements](#implements)
- [声明合并](#声明合并)
- [only interface can](#only-interface-can)
- [& 和 | 联合操作符](#联合操作符)
- [交叉类型](#交叉类型)
- [extends](#extends)
- [keyof](#keyof)
- [泛型](#泛型)
- [小试牛刀](#小试牛刀)
- [infer](#infer)
- [工具泛型](#工具泛型)
- [Partial](#Partial)
- [Required](#Required)
- [Readonly](#Readonly)
- [Record](#Record)
- [Pick](#Pick)
- [Exclude](#Exclude)
- [Extract](#Extract)
- [Omit](#Omit)
- [类型断言](#类型断言)
- [函数重载](#函数重载)



## apply

```ts
fn.apply(this, arguments);
```

上面的会报 `TS2683: 'this' implicitly has type 'any' because it does not have a type annotation.` 这个错误

正确写法如下
```typescript
fn.apply(null, arguments);
```

## setTimeout

```ts
// 防抖
function debounce(fn: Function, time: number) {
  let timeout: number;
  return function () {
    clearTimeout(timeout);
    timeout = setTimeout(function () {
      fn.apply(null, arguments);
    }, time);
  }
}
```

上面的报错信息为 `TS2322: Type 'Timeout' is not assignable to type 'number'.`

正确写法如下，
```ts
function debounce(fn: Function, time: number) {
  let timeout: number;
  return function () {
    clearTimeout(timeout);
    timeout = window.setTimeout(function () {
      fn.apply(null, arguments);
    }, time);
  }
}
```

## 可索引类型
## interface和type关键字
## Objects Functions
## 其他数据类型
## Extend
## interface extends interface
## type extends type
## interface extends type
## type extends interface
## implements
## 声明合并
## only interface can
## 联合
## extends
## keyof
## 泛型
## 小试牛刀
## infer
## Partial
## Required
## Readonly
## Record
## Pick
## Exclude
## Extract
## Omit
## 类型断言
## 函数重载

