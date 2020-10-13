# setState

- 传入对象

更改状态
```js
this.setState({
  proppertyName: 'proppertyName'
});
```

- 传入第二参数：回调

```js
console.log('outer', this.state.proppertyName);

this.setState({
  proppertyName: 'proppertyName'
}, () => {
  console.log(this.state.proppertyName);
});

console.log('outer', this.state.proppertyName);
```

`outer` 外部的 `console.log` 会先输出，但是，拿不到最新的数据，在 `setState` 的第二参数里可以拿到最新的数据状态
