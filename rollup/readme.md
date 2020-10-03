# rollup

- [plugin](#plugin)

## plugin

如何写一个 `rollup` 插件

### what-Plugin

首先看下该插件是如何定义的

```ts
export interface Plugin extends Partial<PluginHooks>, Partial<OutputPluginValueHooks> {
	name: string;
}
```

`Plugin` 继承了 `Partial`，`Partial`定义了属性规范，也就是定义了能传递哪些属性，`Partial` 具体实现如下，

```ts
/**
 * Make all properties in T optional
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

单单指定的 `name` 属性是不能实现功能强大的 `Plugin` 的，继续往 `extends` 里的接口定义里面看，先看 `PluginHooks` 的具体模型，

### what-PluginHooks

```ts
export interface PluginHooks extends OutputPluginHooks {
	buildEnd: (this: PluginContext, err?: Error) => Promise<void> | void;
	buildStart: (this: PluginContext, options: NormalizedInputOptions) => Promise<void> | void;
	load: LoadHook;
	options: (this: MinimalPluginContext, options: InputOptions) => InputOptions | null | undefined;
	resolveDynamicImport: ResolveDynamicImportHook;
	resolveId: ResolveIdHook;
	transform: TransformHook; // transform(code: 原代码, id: 该文件完整路径)
	watchChange: (id: string) => void;
}
```

这里定义了 `PluginHooks` 可以传递的钩子函数：`buildEnd`、`buildStart`、`load`、`options`、`resolveDynamicImport`、`resolveId`、`transform`、`watchChange`

其中，`buildEnd`、`buildStart`、`load`、`resolveDynamicImport`、`resolveId`、`transform` 返回值都是 `Promise` 类型，`watchChange` 监听文件变化

`options` 是 `roullup` 的可选参数对象定义，看[inputoptions-object](https://www.rollupjs.org/guide/en/#inputoptions-object)，完整的清单在[big-list-of-options](https://www.rollupjs.org/guide/en/#big-list-of-options)

__注意__：这里需要特别说明的是，`buildEnd` 返回的是 `Pormise` 类型，因此，在此处要访问构建好的文件会报错，因为打包文件还没被输出到磁盘里

### what-OutputPluginHooks

要想访问打包好的文件，需要了解 `OutputPluginHooks`，这里定义了输出的钩子函数

```ts
interface OutputPluginHooks {
	augmentChunkHash: (this: PluginContext, chunk: PreRenderedChunk) => string | void;
	generateBundle: (
		this: PluginContext,
		options: NormalizedOutputOptions,
		bundle: OutputBundle,
		isWrite: boolean
	) => void | Promise<void>;
	outputOptions: (this: PluginContext, options: OutputOptions) => OutputOptions | null | undefined;
	renderChunk: RenderChunkHook;
	renderDynamicImport: (
		this: PluginContext,
		options: {
			customResolution: string | null;
			format: InternalModuleFormat;
			moduleId: string;
			targetModuleId: string | null;
		}
	) => { left: string; right: string } | null | undefined;
	renderError: (this: PluginContext, err?: Error) => Promise<void> | void;
	renderStart: (
		this: PluginContext,
		outputOptions: NormalizedOutputOptions,
		inputOptions: NormalizedInputOptions
	) => Promise<void> | void;
	/** @deprecated Use `resolveFileUrl` instead */
	resolveAssetUrl: ResolveAssetUrlHook;
	resolveFileUrl: ResolveFileUrlHook;
	resolveImportMeta: ResolveImportMetaHook;
	writeBundle: (
		this: PluginContext,
		options: NormalizedOutputOptions,
		bundle: OutputBundle
	) => void | Promise<void>;
}
```

1. [`augmentChunkHash` 单独使用似乎无效](https://www.rollupjs.org/guide/en/#augmentchunkhash) 为每一块增加哈希值，以便于输出的文件之间相互调用的时候使用。
2. `generateBundle` 生成代码块。在 `bundle.generate()` 之后或在将文件写入 `bundle.write` 写入磁盘之前调用。**要在写入文件后修改文件，要使用 `writeBundle` 挂钩**

文件输出定义，也支持命令的部分看[command-line-flags](https://www.rollupjs.org/guide/en/#command-line-flags)
```
options：NormalizedOutputOptions {
  amd: { define: 'define' },
  assetFileNames: 'assets/[name]-[hash][extname]', // 文件格式输出
  banner: [Function], // 打包banner注释
  chunkFileNames: '[name]-[hash].js', // 打包分包后的文件格式
  compact: false, // 压缩代码
  dir: undefined,
  dynamicImportFunction: undefined, // 当输出格式为 ES Bundle时，可以将动态加载的函数重命名，当然也可以使用原来的名称，详情参考这里https://github.com/uupaa/dynamic-import-polyfill
  entryFileNames: '[name].js', // output后的入口文件名
  esModule: true, // 表示导出的值是ES模块的名称空间，并且此模块的默认导出对应于导出对象的.default属性
  exports: 'auto', // 指定导出模式（自动，默认，命名，无）
  extend: false, // 对全局对象进行扩展。是否以umd或iife格式扩展由name选项定义的全局变量。 如果为true，则全局变量将定义为（global.name = global.name || {}）。 如果为false，则将按名称定义的全局名称将被覆盖（global.name = {}）
  externalLiveBindings: false, // 当引入外部代码的时候是否使用 `Object.defineProperty` 对 `exports` 进行属性扩展，使用 `Object.defineProperty` 的代码较多；`false`，则是使用 `exports.属性名` 的方式扩展 `exports`
  file: 'dist/index.min.js', // 打包后的文件
  footer: [Function], // 与banner类似，在尾部生成内容
  format: 'umd', // 输出规范
  freeze: true, // 封装命名空间
  globals: { jquery: '$' },
  hoistTransitiveImports: true, // 提升导入
  indent: true, // 缩进
  inlineDynamicImports: false, // 如果内联了动态导入，则将立即执行仅动态导入的模块
  interop: [Function], // 兼容 ADM,CommonJS,IIFE
  intro: [Function], // 和banner、footer类型，只能放在非代码块内
  manualChunks: {}, // 公共代码块。比如将`loadsh`作为公共代码块，https://www.rollupjs.org/guide/en/#outputmanualchunks
  minifyInternalExports: false, // 是否使用as语法导入或者导出，`export { importantValue as i };`、`import { i as importantValue } from './main-5532def0.js';`
  name: 'index', // 文件名
  namespaceToStringTag: false, // 为命名空间的方法创建合适的 `toString` 方法，比如，`import * as yourName from './file.js'; console.log(String(yourName));`
  noConflict: false, // 冲突解决。UMD格式的包将会生成额外的noConflict，在IIFE环境调用时候，会使用代码的exports
  outro: [Function], // 和banner、footer类型，只能放在非代码块内
  paths: {}, // 将外部的代码块映射到paths里。可以在代码内部直接使用。适用于CDN场景，具体的看https://www.rollupjs.org/guide/en/#outputpaths
  plugins: [], // 定义的插件，可以自己传入
  preferConst: false, // Use `const` instead of `var` for exports
  preserveModules: false, // 此模式将使用原始模块名称作为文件名为所有模块创建单独的块，需要结合dir属性使用
  sourcemap: false, // 是否输出映射文件
  sourcemapExcludeSources: false,
  sourcemapFile: undefined,
  sourcemapPathTransform: undefined,
  strict: true, // 严格模式
  systemNullSetters: false
}
```
第二参数 `bundle`，是 `OutputBundle` 类型，关于 `rollup.rollup` 构建看[这里](https://www.rollupjs.org/guide/en/#rolluprollup)
```
bundle: OutputBundle {
  'index.min.js': { // 输出的文件名
    exports: [ 'isArrayBuffer' ], // 暴露的方法名
    facadeModuleId: 'D:\\code\\xxx\\src\\index.ts', // 源码目录
    isDynamicEntry: false, // 是否动态入口
    isEntry: true, // 是否为入口文件
    isImplicitEntry: false, // 是否在别的代码块加载完之后加载（模块依赖的时候可能会为true）
    modules: [Object: null prototype] { // 模块
      'D:\\code\\xxx\\src\\index.ts': [Object]
    },
    name: [Getter], // extends RenderedChunk extends PreRenderedChunk
    type: 'chunk', // extends PreRenderedChunk
    code: '/*!\n' + // extends PreRenderedChunk 生成的代码，不一定输出到磁盘，输出到磁盘要看 `writeBundle`
      '  * @silen/type v1.0.0\n' +
      '  * (c) 2020\n' +
      '  * author: sunsilent\n' +
      '  * email: sunsilently@outlook.com\n' +
      '  * @license MIT\n' +
      '  */\n' +
      '!function(e,t){"object"==typeof exports&&"undefined"!=typeof module?t(exports):"function"==typeof define&&define.amd?define(["exports"],t):t((e="undefined"!=typeof globalThis?globalThis:e||self).index={})}(this,(function(e){"use strict";var t=Object.prototype.toString;e.isArrayBuffer=function(e){return"[object ArrayBuffer]"===t.call(e)},Object.defineProperty(e,"__esModule",{value:!0})}));\n',
    dynamicImports: [], // 动态加载的代码块
    fileName: 'index.min.js', // 输出的文件名
    implicitlyLoadedBefore: [], // 在该代码块之后加载的模块
    importedBindings: {}, // importedBindings: {[imported: string]: string[]} 每个模块导入的依赖
    imports: [], // 导入的外部静态模块
    map: null, // map映射是否展示
    referencedFiles: [] // 通过`import.meta.ROLLUP_FILE_URL_<id>`引用的文件
  },
  'index.d.ts': { // 类型文件相关
    fileName: 'index.d.ts',
    name: undefined,
    isAsset: [Getter],
    source: 'declare function isArrayBuffer(val: any): boolean;\r\n' +
      'export { isArrayBuffer };\r\n',
    type: 'asset'
  }
}
```

3. `writeBundle` 和 `generateBundle` 的参数定义类似，原理也类似

### do-plugin

0. 目前的痛点

- 写代码
- build
- copy到另外一个目录下（备注：将一个大的代码包拆分成多个小的代码包，需要根据方法名动态生成文件夹和文件）
- 之后将动态生成的代码推送到git仓库
- 发布到npm仓库

1. 实现

1.1 写业务代码
```
const toStr = Object.prototype.toString;

/**
 * Determine if a value is an ArrayBuffer
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is an ArrayBuffer, otherwise false
 */
function isArrayBuffer(val) {
  return toStr.call(val) === '[object ArrayBuffer]';
}

export {
  isArrayBuffer
};
```

1.2 build

1.2.1 修改配置

生产环境下执行的插件业务
```rollup.config.js
if (process.env.NODE_ENV === 'production') {
  packageFormats.forEach(format => {
    if ('umd' === format) {
      packageConfigs.push(createMinifiedConfig(format))
    }
  })
}

function createMinifiedConfig(format) {
  const { terser } = require('rollup-plugin-terser');
  return createConfig(
    format,
    {
      file: outputConfigs[format].file.replace(/\.js$/, '.min.js'),
      format: outputConfigs[format].format, // 输出umd/amd/iife...格式
      name: 'index' // umd必须
    },
    [terser(), { // 插件就是一个对象，该对象下有一些特定的方法，这些在会运行时执行的方法就是钩子函数
      writeBundle(options, bundle) {
        const { execFile } = require('child_process');
        execFile('node', ['D:/code/package/auto/copy-code.js', 'isArrayBuffer'], (error, stdout, stderr) => {
          console.log(error, stdout, stderr);
        });
      }
    }]
  )
}
```

1.2.2 build

```
npm run build
```

1.2.3 business

在 `D:/code/package/auto/copy-code.js` 实现 `copy`

然后，实现`git push`、`npm publish`

