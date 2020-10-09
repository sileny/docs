# css

- [svg](#svg)
- [四分之一扇形](#sector)
- [圆点环绕](#circle-points)
- [米字型](#union-jack)
- [八圆角形](#8radius)
- [九大行星](#9stars)

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
