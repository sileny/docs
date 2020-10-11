# dom

```html
<div class="test" id="test" title="title">
  test for div
  <p class="test">lorem</p>
</div>
```

使用js解析dom对象，即虚拟dom

```javascript
const Div = {
  tagName: 'div',
  attrs: {
    id: 'test',
    className: 'test',
    title: 'title'
  },
  children: [
    'test for div',
    {
      tagName: 'p',
      attrs: {
        className: 'test'
      },
      children: [
        'lorem'
      ]
    }
  ]
}
```
