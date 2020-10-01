# typescript

- [apply](#apply)
- [setTimeout](#settimeout)

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
