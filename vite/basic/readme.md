# api

- [reactive](#reactive)
- [ref](#ref)
- [isRef](#isref)
- [toRefs](#torefs)
- [computed](#computed)
- [watch](#watch)

## reactive

Vue2里的 `Vue.observable()` 等价于 `vite3` 里的 `reactive` 函数，用来创建响应式的对象，基本代码如下：

```js
import { reactive } from '@vue/composition-api'

export default {
  setup() {
    const state = reactive({ count: 0 })
    console.log(state)

    state.count += 1
    console.log(state.count)

    return state // return出去，template里可以访问
  }
}
```

## ref

`ref`：创建响应式的数据对象，这对象实际上只包含一个 `value` 属性

推荐使用 `ref` 创建响应式的数据对象，不直接推荐使用 `reactive`，因为，`reactive` 创建的数据对象是只读的值，不会变化

```js
import { ref } from '@vue/composition-api'

export default {
  setup() {
    // 设置count的初始value为0
    const count = ref(0)
    console.log(count)

    count.value++
    console.log(count.value)
    return count
  }
}
```

## isRef

`isRef`：判断某个对象是否通过 `ref()` 创建出来的

```js
const unwrapped = isRef(foo) ? foo.value : null
```

## toRefs

`toRefs`：该函数将 `reactive()` 创建的响应式的对象转换成普通的对象，只不过，这个对象上的每个属性节点都是 `refs()` 函数的响应式的数据

```js
import { toRefs } from '@vue/composition-api'

export default {
  setup() {
    const state = reactive({ count: 0 })

    const increment = () => state.count++

    // return { ...state, increment } // 此时的state是只读的值，不会变化
    return { ...toRefs(state), increment }
  }
}
```

```html
<p>{{ count }}</p>
```

## computed

1. `computed`：返回 `ref` 的实例

```js
import { computed } from '@vue/composition-api'

export default {
  const count = refs(0)

  const plusOne = computed(() => count.value + 1)

  // plusOne.value++ //error，computed只读的
}
```

2. `computed` 定义可读可取的数据对象

```js
import { computed } from '@vue/composition-api'

export default {
  const count = refs(0)

  const plusOne = computed({
    get() { return count.value + 1 },
    set(val) { count.value = val - 1 }
  })

  plusOne.value = 9
}
```

## watch

1. `watch`: 监听数据

```js
import { watch } from '@vue/composition-api'

export default {
  setup() {
    // const state = ref(0)
    const state = reactive({count: 0})

    watch(() => state.count, (val, old) => { console.log(val, old) })

    setTimeout(() => state.count++, 2000)
  }
}
```

2. `watch`: 监听异步的数据（即，组件创建时不执行 `watch`）

```js
import { watch } from '@vue/composition-api'

export default {
  setup() {
    const state = reactive({count: 0})

    watch(
      () => state.count,
      (val, old) => { console.log(val, old) },
      { lazy: true } // 组件创建的时候不执行
    )

    setTimeout(() => state.count++, 2000)
  }
}
```

3. `watch`: 监听 `ref` 创建的数据对象

```js
import { ref, watch } from '@vue/composition-api'

export default {
  setup() {
    const count = ref(0)

    watch(() => count, (val, old) => { console.log(val, old) })

    setTimeout(() => count.value++, 2000)
  }
}
```

4. `watch`: 监听 `reactive` 创建的多个数据对象属性

```js
import { watch, reactive } from '@vue/composition-api'

export default {
  setup() {
    const state = reactive({count: 0, name: 'a'})

    watch(
      [() => state.count, () => state.name],
      ([count, name], [oldCount, oldName]) => {
        console.log(count, name, oldCount, oldName)
      },
      { lazy: true } // 创建组件的时候不执行watch
    )

    setTimeout(() => {
      state.count++
      state.name = 'xa'
    }, 2000)
  }
}
```

5. `watch`: 监听 `ref` 创建的多个数据对象属性

```js
import { watch, ref } from '@vue/composition-api'

export default {
  setup() {
    const count = ref(0)
    const name = ref('dax')

    watch(
      [count, name],
      ([count, name], [oldCount, oldName]) => {
        console.log(count, name, oldCount, oldName)
      },
      { lazy: true } // 创建组件的时候不执行watch
    )

    setTimeout(() => {
      count.value++
      name.value = 'cxa'
    }, 2000)
  }
}
```

6. 清理 `watch` 监听：

`watch` 会在组件销毁的时候自动停止，如果想明确地停止 `watch`，可以直接调用 `watch` 的返回值

```js
const stop = watch(() => {}, (val, old) => {})

stop(); // 调用
```

7. 清除掉无效的 `watch`

```js
import { ref, watch } form '@vue/composition-api'

export default {
  setup() {
    const kw = ref('')

    // 异步执行的
    const print = val => { console.log(val) }

    watch(kw, (val, old, clean) => {
      const timerId = print(val)

      clean(() => clearTimeout(timerId))
    }, { lazy: true })

    return { kw }
  }
}
```

html部分依旧是旧的写法
```html
<input v-model="kw" />
```


