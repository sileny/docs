# element

- [SideMenu](#sidemenu)

## SideMenu

```vue
<template>
  <section class="sidebar-wrapper">
    <el-menu class="sidebar-menu" router unique-opened :default-active="$route.path" :collapse="!sidebar.collapsed">
      <!-- 1 v-for -->
      <template v-for="route in routes">
        <!--1 v-if -->
        <template v-if="route.children && route.children.length > 0">
          <el-submenu :index="route.path" :key="route.path">
            <template slot="title"> {{ route.meta.name }}</template>
            <template v-for="item in route.children">
              <template v-if="item.children && item.children.length > 0">
                <!--2 v-if -->
                <el-submenu :index="item.path" :key="item.path">
                  <template slot="title"> {{ item.meta.name }}</template>
                  <template v-for="ctor in item.children">
                    <!--3 v-for -->
                    <el-menu-item :index="ctor.path" :key="ctor.path">{{ ctor.meta.name }}</el-menu-item>
                  </template>
                  <!--over 3 v-for -->
                </el-submenu>
              </template>
              <!--over 2 v-if -->
              <template v-else>
                <el-menu-item :index="item.path" :key="item.path">{{ item.meta.name }}</el-menu-item>
              </template>
            </template>
            <!--over 2 v-for -->
          </el-submenu>
        </template>
        <!--over 1 v-if -->
        <template v-else>
          <el-menu-item :index="route.path" :key="route.path">{{ route.meta.name }}</el-menu-item>
        </template>
      </template>
      <!--over 1 v-for -->
    </el-menu>
  </section>
</template>
<script>
import { mapGetters } from 'vuex';

export default {
  props: {
    routes: Array
  },
  computed: {
    ...mapGetters(['sidebar'])
  }
};
</script>
```

`sidebar.js` 状态管理代码
```js
// sidebar.js
import Storage from 'storage-ts';

const sidebar = {
  state: {
    sidebar: {
      collapsed: !+Storage.get('sidebarCollapsed')
    }
  },
  mutations: {
    TOGGLE_SIDEBAR: state => {
      if (state.sidebar.collapsed) {
        Storage.set('sidebarCollapsed', 1);
      } else {
        Storage.set('sidebarCollapsed', 0);
      }
      state.sidebar.collapsed = !state.sidebar.collapsed;
    }
  },
  actions: {
    ToggleSideBar: ({ commit }) => {
      commit('TOGGLE_SIDEBAR');
    }
  }
};

export default sidebar;
```

`getters.js` 代码
```js
export default {
  sidebar: state => state.sidebar.sidebar
};
```

`store.js` 代码
```js
import Vue from 'vue';
import Vuex from 'vuex';

import sidebar from './modules/sidebar';

import getters from './getters';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {},
  mutations: {},
  actions: {},
  modules: {
    menu,
    user,
    sidebar
  },
  getters
});
```

`SideMenu` 组件在 `Layout.vue` 里引用到

```vue
<template>
  <div class="page-layout">
    <div
      class="page-sidebar scrollbar scrollbar-light"
      :style="{ width: sidebar.collapsed ? sidebarWidth : swm }"
    >
      <SideMenu :routes="routerConfig"></SideMenu>
    </div>
    <div
      class="page-content scrollbar scrollbar-light"
      :style="{ width: `calc(100% - ${sidebar.collapsed ? sidebarWidth : swm})` }"
    >
      page-content
    </div>
  </div>
</template>

<script>
import SideMenu from '@/components/layout/SideMenu.vue';
import { mapGetters } from 'vuex';

export default {
  name: 'layout',
  components: {
    SideMenu
  },
  data() {
    return {
      sidebarWidth: '200px',
      swm: '50px',
      routerConfig: []
    };
  },
  computed: {
    ...mapGetters(['sidebar'])
  },
  created() {
    this.getRoutes();
  },
  methods: {
    getList() {
      const routes = this.$router.options.routes;
      const { path } = this.$route;
      const str = path
        .split('/')
        .slice(0, 2)
        .join('/');
      const arr = routes.filter(value => new RegExp('^' + str).test(value.path))[0].children;
      this.routerConfig = arr;
    },
    getRoutes() {
      this.getList();
      this.$router.afterEach((to, from) => {
        this.getList();
      });
    }
  }
};
</script>

<style lang="scss">
@import '../styles/ui-theme';
.page-layout {
  //background: #2c3e50;
  height: 100vh;
  display: flex;
  justify-content: flex-start;
}
.page-header {
  background-color: rgba(255, 255, 255, 1);
}
.page-sidebar {
  .logo {
    //width: 200px;
    background: url(../assets/logo.png) center no-repeat;
  }
  .el-menu {
    border-right: none;
    background-color: transparent;
  }
}
.page-sidebar,
.el-menu {
  background: linear-gradient(360deg, $--sidebar-linear-color-from 0%, $--sidebar-linear-color-to 100%);
  .el-menu-item,
  .el-submenu__title {
    color: $--font-color-light;
    &:hover,
    &:focus {
      background: linear-gradient(270deg, $--sidebar-item-linear-color-from 0%, $--sidebar-item-linear-color-to 100%);
    }
  }
  .el-menu-item,
  .el-submenu__title,
  .el-submenu .el-menu-item {
    height: 32px;
    line-height: 32px;
  }
  .el-menu-item.is-active {
    background: linear-gradient(270deg, $--sidebar-item-linear-color-from 0%, $--sidebar-item-linear-color-to 100%);
    color: $--font-color-light;
    &:before {
      position: absolute;
      top: 0;
      left: 0;
      bottom: 0;
      content: '';
      width: 4px;
      height: 32px;
      display: block;
      background-color: $--sidebar-item-left-color;
    }
  }
  .el-menu-item-group__title {
    padding-top: 0;
    padding-bottom: 0;
  }
}
.page-content {
  //$sidebar-width: 2.6rem;
  height: 100vh;
  //overflow: hidden;
  background: rgba(240, 242, 245, 1);
  .page-sidebar {
    //width: $sidebar-width;
  }
  .page-views {
    //width: calc(100% - #{$sidebar-width});
  }
  .page-sidebar,
  .page-views {
    //height: 100%;
  }
}
</style>
```
