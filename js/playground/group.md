# group

需要分组的数据

```js
const list = [
  { controllable: true, groupId: 1, name: 1, value: 1 },
  { controllable: true, groupId: 1, name: 2, value: 2 },
  { controllable: false, groupId: 2, name: 3, value: 3 },
  { controllable: false, groupId: 3, name: 4, value: 4 },
  { controllable: false, groupId: 4, name: 5, value: 5 }
];

// 需要添加到 `list` 每一条数据里的属性值
const sn = 2020;
```

对数据进行分组
```js
class DyForm {
  constructor(array = []) {
    // 为array添加sn
    this.origin = array;
    this.groups = null;
    this.initialized = false;
    this.init();
  }

  init() {
    if (this.initialized) throw new Error(`You have already initialized. this.initialized: ${this.initialized}`);
    this.initialized = true;

    this.group();
  }

  group() {
    const group = {};

    for (let i = 0; i < this.origin.length; i++) {
      const item = this.origin[i];
      item.sn = sn;
      const { controllable, groupId } = item;
      const key = controllable + '-' + groupId;
      if (!group[key]) group[key] = [];
      group[key].push(item);
    }

    // console.log(group);
    this.groups = group;
  }

  /**
   * 所有的分组
   * @return { Object }
   * ```js
   * // 返回值：{true-1: Array(2), false-2: Array(1), false-3: Array(1), false-4: Array(1)}
   * ```
   */
  getGroups() {
    return this.groups;
  }

  /**
   * 获取可控制的分组
   * @return {Array}
   * ```js
   * // 返回值：Array
   * ```
   */
  getGroupControllable() {
    for (const p in this.groups) {
      const list = p.split('-');
      if (list[0] === 'true') {
        return this.groups[p];
      }
    }
    return [];
  }

  /**
   * 获取非控制类型的组数据--以数组形式返回
   * @return {Array}
   * ```js
   * // 返回值：Array
   * ```
   */
  getGroupRests() {
    const res = [];
    for (const p in this.groups) {
      const list = p.split('-');
      if (list[0] === 'false') {
        res.push(...this.groups[p]);
      }
    }
    return res;
  }

  /**
   * 获取非控制类型的组数据--以对象形式返回
   * @return { Object }
   * ```js
   * // 返回值：{false-2: Array(1), false-3: Array(1), false-4: Array(1)}
   * ```
   */
  getGroupRest() {
    const res = {};
    for (const p in this.groups) {
      const list = p.split('-');
      if (list[0] === 'false') {
        res[p] = this.groups[p];
      }
    }
    return res;
  }

  clear() {
    this.origin = [];
    this.groups = null;
    this.initialized = null;
  }
}
```

用法
```js
const df = new DyForm(list);
console.log('getGroupControllable() ', df.getGroupControllable());
console.log('getGroupRest()         ', df.getGroupRest());
console.log('getGroupRests()        ', df.getGroupRests());
console.log('getGroups()            ', df.getGroups());
```
