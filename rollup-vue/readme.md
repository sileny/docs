# rollup-vue


## rollup编译vue所需依赖

`rollup` 编译 `vue` 组件文件

- `@babel/core` 转换源代码，你可以对源代码进行修改
- `@babel/preset-env` 负责转换新语法

- `@rollup/plugin-babel` 转换指定的文件，默认转换的文件为 `['.js', '.jsx', '.es6', '.es', '.mjs']`
- `@rollup/plugin-commonjs` 转换 `commonjs` 模块为 `es6` 语法可以导入的模块。`npm` 中的大多数包都是以 `CommonJS` 模块的形式出现的，这个包可以在 `rollup.conifig.js` 里直接使用 `es6` 的 `import` 语句

- `@rollup/plugin-node-resolve` 用来查找模块，比如查找 `node_modules` 下的某个模块，默认查找的模块为 `['.mjs', '.js', '.json', '.node']`

- `rollup-plugin-terser` 压缩代码

- `vue-template-compiler` 编译 `vue` 文件。一个 `vue` 文件通常包含 `template`、`script`、`style` 三部分。会将 `template` 内部的代码编译成 `render` 函数，最终该函数会被注入到 `script` 标签里。`vue` 文件里的 `script` 会作为一个 `ES Module`来执行，通过 `export` 显示指定输出。注意，`vue-template-compiler` 版本和 `vue` 版本要一致


# 案例

- 需求：
  - 编写一个 `vue` 组件
  - 支持 `Vue.use` 插件的写法

- 实现：

1、项目结构
```
- src/
-   index.js
-   unity.vue
```

2、源码

`index.js`
```js
import unity from "./unity.vue";

unity.install = (Vue) => {
  Vue.component("VueUnity", unity);
};

export default unity;

```

`unity.vue`
```vue
<template>
  <div class="unity-wrapper">
    <div :id="containerId" :style="{ width: width + 'px', height: height + 'px' }"></div>
    <div class="unity-loader" v-if="!loaded">
      <div class="unity-loader-bar">
        <div class="unity-loader-fill" :style="{ width: progress * 100 + '%' }"></div>
        <div class="unity-loader-text text-center">loading...</div>
      </div>
    </div>
    <div class="unity-button" v-if="hideFullButton !== true" @click="fullscreen">Fullscreen</div>
  </div>
</template>

<script>
import Vue from 'vue';

export default {
  props: ['src', 'module', 'width', 'height', 'externalProgress', 'unityLoader', 'hideFullButton'],
  name: 'a-unity',
  data() {
    return {
      containerId: 'unity-container-' + Number(Math.random().toString().substring(3, length) + Date.now()).toString(36),
      unityInstance: null,
      loaded: false,
      progress: 0,
      error: null
    };
  },
  methods: {
    fullscreen() {
      this.unityInstance.SetFullscreen(1);
    },
    message(gameObject, method, param) {
      if (param === null) {
        param = '';
      }
      if (this.unityInstance !== null) {
        this.unityInstance.SendMessage(gameObject, method, param);
      } else {
        console.warn("vue-unity-webgl: you have sent a message to the Unity content, but it wasn't instantiated yet.");
      }
    }
  },
  beforeMount() {
    if (!this.eventBus) {
      this.eventBus = new Vue({
        data: {
          ready: false,
          load: false
        }
      });
    }
    if (typeof UnityLoader === 'undefined' && this.unityLoader && !this.eventBus.load) {
      const script = document.createElement('SCRIPT');
      script.setAttribute('src', this.unityLoader);
      script.setAttribute('async', '');
      script.setAttribute('defer', '');
      document.body.appendChild(script);
      this.eventBus.load = true;
      script.onload = () => {
        this.eventBus.ready = true;
        this.eventBus.$emit('onload');
      };
    } else {
      this.eventBus.ready = true;
      this.eventBus.load = true;
    }
  },
  mounted() {
    const instantiate = () => {
      if (typeof UnityLoader === 'undefined') {
        const error =
          'The UnityLoader was not defined, please add the script tag ' +
          'to the base html and embed the UnityLoader.js file Unity exported or use "unityLoader" attribute for path to UnityLoader.js.';
        console.error(error);
        this.error = error;
        return;
      }
      if (this.src == null) {
        const error = 'Please provide a path to a valid JSON in the "src" attribute.';
        console.error(error);
        this.error = error;
        return;
      }
      const params = {};
      if (this.externalProgress) {
        params.onProgress = this.externalProgress;
      } else {
        params.onProgress = (unityInstance, progress) => {
          const loaded = progress === 1;
          this.loaded = loaded;
          this.progress = progress;
          this.$emit('progress', progress, unityInstance);
          loaded && this.$emit('loaded', loaded, unityInstance);
        };
      }
      if (this.module) {
        params.Module = this.module;
      }
      // eslint-disable-next-line no-undef
      this.unityInstance = UnityLoader.instantiate(this.containerId, this.src, params);
    };
    if (this.eventBus.ready) {
      instantiate();
    } else {
      this.eventBus.$on('onload', () => {
        instantiate();
      });
    }
  }
};
</script>

```

项目代码在[这里](https://github.com/sileny/unity)

3、进一步优化

```
<div class="unity-loader" v-if="!loaded">
  <div class="unity-loader-bar">
    <div class="unity-loader-fill" :style="{ width: progress * 100 + '%' }"></div>
    <div class="unity-loader-text text-center">loading...</div>
  </div>
</div>
```

这部分代码嵌入到了组件内部，不方便实现纯粹的自定义的进度加载

进行如下改写

直接将上述的代码替换为 `<slot></slot>`

最终 `template` 代码如下
```
<template>
  <div class="unity-wrapper">
    <div :id="containerId" :style="{ width: width + 'px', height: height + 'px' }"></div>
    <slot></slot>
    <div class="unity-button" v-if="hideFullButton !== true" @click="fullscreen">Fullscreen</div>
  </div>
</template>
```
