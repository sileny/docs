# rollup

- [plugin](./plugin)
- [环境变量注入](#env)


## env

如果打包里用到了环境变量，则需要将环境变量提取出来，保证能在生产环境下引用，否则会报如下错，
```
Uncaught ReferenceError: process is not defined
```

提取变量
```
import replace from '@rollup/plugin-replace';

const mode = process.env.NODE_ENV;

export default {
  plugins: [
    replace({
      'process.env.NODE_ENV': JSON.stringify(mode)
    })
  ]
}
```

