# shape

绘制特殊形状

- [四分之一扇形](#sector)
- [圆点环绕](#circle-points)
- [米字型](#union-jack)
- [八圆角形](#8radius)
- [九大行星](#9stars)
- [一个圆8等分](#8partion)
- [消除chrome浏览器的记录密码填充颜色](#autofill)

## svg

- [通过img引入的svg如何改颜色?](./svg)

## sector

四分之一扇形
```css
.sector {
    box-sizing: border-box;
    width: 100px;
    height: 100px;
    border-top: 50px solid rgba(0, 0, 0, 0);
    border-right: 50px solid rgba(255, 0, 0, 1);
    border-bottom: 50px solid rgba(0, 0, 0, 0);
    border-left: 50px solid rgba(0, 0, 0, 0);
    border-radius: 50%;
}
```

## circle-points

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    body {
      padding-top: 200px;
    }

    ul {
      width: 200px;
      height: 200px;
      margin: 0 auto;
      position: relative;
    }

    li {
      position: absolute;
      width: 20px;
      background: #000;
    }
  </style>
</head>
<body>
<script>
  const doc = document.createDocumentFragment();
  for (let i = 0; i < 8; i++) {
    const li = document.createElement('li');
    li.className = 'li-' + (i % 2 === 0 ? "1" : "2");
    li.style.transform = `rotate(${ i * 0.125 }turn)`;
    doc.appendChild(li);
  }
  const ul = document.createElement('ul');
  ul.appendChild(doc);
  document.body.appendChild(ul);
</script>
</body>
</html>

```

## union-jack

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    body {
      padding-top: 200px;
    }

    ul {
      width: 200px;
      height: 200px;
      margin: 0 auto;
      position: relative;
    }

    li {
      position: absolute;
      width: 60px;
      background: #000;
    }
  </style>
</head>
<body>
<script>
  const doc = document.createDocumentFragment();
  for (let i = 0; i < 8; i++) {
    const li = document.createElement('li');
    li.className = 'li-' + (i % 2 === 0 ? "1" : "2");
    li.style.transform = `rotate(${ i * 0.125 }turn)`;
    doc.appendChild(li);
  }
  const ul = document.createElement('ul');
  ul.appendChild(doc);
  document.body.appendChild(ul);
</script>
</body>
</html>

```

## 8radius

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    body {
      padding-top: 200px;
    }

    ul {
      width: 200px;
      height: 200px;
      margin: 0 auto;
      position: relative;
    }

    li {
      position: absolute;
      width: 60px;
      background: #000;
      -webkit-border-radius: 50%;
      -moz-border-radius: 50%;
      border-radius: 50%;
    }
  </style>
</head>
<body>
<script>
  const doc = document.createDocumentFragment();
  for (let i = 0; i < 8; i++) {
    const li = document.createElement('li');
    li.className = 'li-' + (i % 2 === 0 ? "1" : "2");
    li.style.transform = `rotate(${ i * 0.125 }turn)`;
    doc.appendChild(li);
  }
  const ul = document.createElement('ul');
  ul.appendChild(doc);
  document.body.appendChild(ul);
</script>
</body>
</html>

```

## 9stars

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    body {
      padding-top: 200px;
    }

    ul {
      width: 200px;
      height: 200px;
      margin: 0 auto;
      position: relative;
    }

    li {
      position: absolute;
      width: 60px;
      height: 60px;
      background: #000;
      -webkit-border-radius: 50%;
      -moz-border-radius: 50%;
      border-radius: 50%;
    }
  </style>
</head>
<body>
<script>
  const doc = document.createDocumentFragment();
  for (let i = 0; i < 8; i++) {
    const li = document.createElement('li');
    li.className = 'li-' + (i % 2 === 0 ? "1" : "2");
    li.style.transform = `rotate(${ i * 0.125 }turn)`;
    doc.appendChild(li);
  }
  const ul = document.createElement('ul');
  ul.appendChild(doc);
  document.body.appendChild(ul);
</script>
</body>
</html>

```

## 8partion

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>CSS等分圆</title>
  <meta charset="utf-8">
  <style type="text/css">
    .circle-left {
      width: 100px;
      height: 200px;
      border-radius: 0px 100px 100px 0px;
      position: absolute;
      right: 0;
      transform-origin: 0 50%;
    }

    .circle-left:hover,
    .circle-right:hover {
      cursor: pointer;
      background-color: #000!important;
    }

    .circle-right {
      width: 100px;
      height: 200px;
      border-radius: 100px 0px 0px 100px;
      position: absolute;
      right: 0;
      transform-origin: 100% 50%;
    }

    #circle0 {
      width: 200px;
      height: 200px;
      border-radius: 100px;
      background-color: yellowgreen;
      position: relative;
    }

    #circle2 {
      background-color: #70f3ff;
    }

    #circle3 {
      background-color: #ff461f;
      transform: rotate(45deg);
    }

    #circle4 {
      background-color: #bce672;
      transform: rotate(90deg);
    }

    #circle5 {
      background-color: #ffffff;
      transform: rotate(135deg);
    }

    #circle6 {
      background-color: #3b2e7e;
      transform: rotate(180deg);
    }

    #circle7 {
      background-color: #ff2121;
      transform: rotate(225deg);
    }

    #circle8 {
      background-color: #16a951;
      transform: rotate(270deg);
    }

    #circle9 {
      background-color: #e0eee8;
      transform: rotate(315deg);
    }

    #circle10 {
      background-color: #00f;
      transform: rotate(360deg);
    }

    #left {
      clip: rect(0px 100px 200px 0px);
      position: absolute;
      right: 0px;
      width: 100px;
      height: 200px;
      overflow: hidden;
    }

    #right {
      clip: rect(0px 100px 200px 0px);
      position: absolute;
      left: 0px;
      width: 100px;
      height: 200px;
      overflow: hidden;
    }
  </style>
</head>
<body>
<div id="circle0">
  <div id="left">
    <div class="circle-left" id="circle2"></div>
    <div class="circle-left" id="circle3"></div>
    <div class="circle-left" id="circle4"></div>
    <div class="circle-left" id="circle5"></div>
  </div>
  <div id="right">
    <div class="circle-right" id="circle10"></div>
    <div class="circle-right" id="circle9"></div>
    <div class="circle-right" id="circle8"></div>
    <div class="circle-right" id="circle7"></div>
    <div class="circle-right" id="circle6"></div>
  </div>
</div>
</body>
</html>

```

