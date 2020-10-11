# start react

```ecmascript 6
// src/App.js

import React from 'react';

export default class React.Component {
  render() {
    return (<h1>app test</h1>)
  }
}
```

```ecmascript 6
// src/index.js

import React from 'react';
import ReactDom from 'react-dom';
import App from './App';

ReactDom.render(<App/>, document.getElementById('app'));
```

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>index</title>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

等待编译完成后再查看页面
