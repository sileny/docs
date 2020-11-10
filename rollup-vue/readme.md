# rollup-vue

`rollup` 编译 `vue` 组件文件

- `@babel/core` 转换源代码，你可以对源代码进行修改
- `@babel/preset-env` 负责转换新语法

- `@rollup/plugin-babel` 转换指定的文件，默认转换的文件为 `['.js', '.jsx', '.es6', '.es', '.mjs']`
- `@rollup/plugin-commonjs` 转换 `commonjs` 模块为 `es6` 语法可以导入的模块。`npm` 中的大多数包都是以 `CommonJS` 模块的形式出现的，这个包可以在 `rollup.conifig.js` 里直接使用 `es6` 的 `import` 语句

- `@rollup/plugin-node-resolve` 用来查找模块，比如查找 `node_modules` 下的某个模块，默认查找的模块为 `['.mjs', '.js', '.json', '.node']`

- `rollup-plugin-terser` 压缩代码

- ``vue-template-compiler` 编译 `vue` 文件。一个 `vue` 文件通常包含 `template`、`script`、`style` 三部分。会将 `template` 内部的代码编译成 `render` 函数，最终该函数会被注入到 `script` 标签里。`vue` 文件里的 `script` 会作为一个 `ES Module`来执行，通过 `export` 显示指定输出

