---
title: Vue+TypeScript 折腾小记
date: 2019-07-20 15:48:07
categories:
- 小记
tags:
- Vue
- vue
- vuetify
- TypeScript
- 前端
---
## 在所有东西之前
为什么要搞这个？
1. 工作需求
2. TypeScript比较吸引人
3. Vue3也要加入对TypeScript的支持
4. 最近确实比较没事做

## 快速开始一个TypeScript下的Vue项目
```bash
> vue create hello-world
```
1. 选择Manually
![image](http://huangyafei.org/image/manually.png)
2. 选择TypeScript等配置，在这里我顺便把vue-router和vuex选上了
![image](http://huangyafei.org/image/select.png)
3. 最终配置, 在linter/formatter那里我选择的是prettier，因为组内都是用这个
![image](http://huangyafei.org/image/finally.png)

愉快的敲回车，等着完毕后
```bash
> cd hello-world
> npm run serve
```
此时一个基于TypeScript的vue项目已经OK了

> 注: 为了支持基于类的API，Vue官方维护了一个[vue-class-component](https://github.com/vuejs/vue-class-component)项目。而使用以上方法新建Vue项目，实际上使用了[vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)，该项目扩展了官方的vue-class-component

## 加入Vuetify这个UI框架
```bash
> vue add vuetify
```
一路回车，选择默认配置。

一切完毕后，会看到`components/HelloWorld.Vue`文件已经被修改为vuetify的预设了，下面修改这个文件，采用基于class的API
```js
// components/HelloWorld.Vue
<script lang="ts">
import { Vue, Prop, Component } from "vue-property-decorator";

@Component
export default class HelloWorld extends Vue {
  ecosystem: any = [
  ];
  importantLinks: any = [
  ];
  whatsNext: any = [
  ];
}
</script>
```

```js
// App.vue
<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Vuetify</span>
        <span class="font-weight-light">MATERIAL DESIGN</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
    </v-toolbar>

    <v-content>
      <router-view />
    </v-content>
  </v-app>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";

@Component
export default class app extends Vue { }
</script>
```

> 下面开始凑字数
---

## vue-property-decorator用过的装饰器的简单用法(其实github首页都有)

### props
> `@Prop(options: (PropOptions | Constructor[] | Constructor) = {}) `decorator

```js
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) readonly propA: number | undefined
  @Prop({ default: 'default value' }) readonly propB!: string
  @Prop([String, Boolean]) readonly propC: string | boolean | undefined
}
```
以上用法等于
```js
export default {
  props: {
    propA: {
      type: Number
    },
    propB: {
      default: 'default value'
    },
    propC: {
      type: [String, Boolean]
    }
  }
```

### watch
> `@Watch(path: string, options: WatchOptions = {})` decorator

```js
import { Vue, Component, Watch } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Watch('child')
  onChildChanged(val: string, oldVal: string) {}

  @Watch('person', { immediate: true, deep: true })
  onPersonChanged1(val: Person, oldVal: Person) {}

  @Watch('person')
  onPersonChanged2(val: Person, oldVal: Person) {}
}
```
以上用法等于
```js
export default {
  watch: {
    child: [
      {
        handler: 'onChildChanged',
        immediate: false,
        deep: false
      }
    ],
    person: [
      {
        handler: 'onPersonChanged1',
        immediate: true,
        deep: true
      },
      {
        handler: 'onPersonChanged2',
        immediate: false,
        deep: false
      }
    ]
  },
  methods: {
    onChildChanged(val, oldVal) {},
    onPersonChanged1(val, oldVal) {},
    onPersonChanged2(val, oldVal) {}
  }
}
```

### emit
> `@Emit(event?: string)` decorator
```js
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  count = 0
  // 在方法里调用addToCount后会自动emit一个'add-to-count'事件，并且emit的值是this.count
  @Emit()
  addToCount(n: number) {
    this.count += n
  }

  @Emit('reset')
  resetCount() {
    this.count = 0
  }

  @Emit()
  returnValue() {
    return 10
  }

  @Emit()
  onInputChange(e) {
    return e.target.value
  }

  @Emit()
  promise() {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```
以上用法等于
```js
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    addToCount(n) {
      this.count += n
      this.$emit('add-to-count', n)
    },
    resetCount() {
      this.count = 0
      this.$emit('reset')
    },
    returnValue() {
      this.$emit('return-value', 10)
    },
    onInputChange(e) {
      this.$emit('on-input-change', e.target.value, e)
    },
    promise() {
      const promise = new Promise(resolve => {
        setTimeout(() => {
          resolve(20)
        }, 0)
      })

      promise.then(value => {
        this.$emit('promise', value)
      })
    }
  }
}
```

### mixins
> `Mixins`，似乎直接用的就是vue-class-component的mixins类

```js
// mixin.js
import { Vue, Component } from 'vue-property-decorator'

// You can declare a mixin as the same style as components.
@Component
export default class MyMixin extends Vue {
  mixinValue = 'Hello'
}
```

```js
import { Component, Mixins } from "vue-property-decorator";
import MyMixin from './mixin.js'

// Use `mixins` helper function instead of `Vue`.
// `mixins` can receive any number of arguments.
@Component
export class MyComp extends Mixins(MyMixin) {
  created () {
    console.log(this.mixinValue) // -> Hello
  }
}
```

### property
> `get property_name() {}`

```js
import { Vue, Component } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  value: number = 10;

  get fake_value(): number {
      return this.value * 10;
  }
}
```

### 基于vuex实现loading dialog
思路:
1. 写一个dialog的component，根据props: show(默认为false)，来决定是否显示
2. 将这个dialog放在App.vue
3. 在vuex内添加一个`loading: boolean`的state，并且实现mutation改变`loading(true or false)`
4. 在App.vue，loading dialog的props show与vuex的loading state绑定起来
5. 其他组件在需要显示loading的时候，通过vuex的mutation将loading设为true，完毕后再将loading设为false

实现:
1. Loading.vue(使用了vuetify)
```js
<template>
  <v-dialog :value="show" hide-overlay persistent width="300">
    <v-card color="info" dark>
      <v-card-text>
        {{ text }}
        <v-progress-linear
          indeterminate
          color="white"
          class="mb-0"
        ></v-progress-linear>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import { Vue, Component, Prop } from "vue-property-decorator";

@Component
export default class Loading extends Vue {
  @Prop({ type: Boolean, default: false })
  show: boolean;

  @Prop({ type: String, default: "请稍后" })
  text: string;
}
</script>
```
2. 使用[vuex-class-component](https://github.com/michaelolof/vuex-class-component)实现vuex
```js
//store/app.ts
import { VuexModule, mutation, Module } from 'vuex-class-component'

@Module({ namespacedPath: 'app/' })
export default class AppModule extends VuexModule {
    private loading = false

    get getLoading(){
        return this.loading
    }

    @mutation
    showLoading() {
        this.loading = true
    }

    @mutation
    hideLoading() {
        this.loading = false
    }
}
```

```js
import Vuex from 'vuex'
import Vue from 'vue'

import AppModule from './app'

const moduleMap: { [key: string]: any } = {
    app: AppModule,
}

const modules: { [key: string]: any } = {}

Object.keys(moduleMap).forEach(key => {
    const module = moduleMap[key]
    modules[key] = module.ExtractVuexModule(module)  // 使用ExtracVuexModule返回真正的vuex module
})

Vue.use(Vuex)
const store = new Vuex.Store({
    modules
})

// A vuex manager is simply an exportable object that houses all your proxies. We can easily create a vuex manager in our original vuex store file. (See Example)
export const vxm = {
    app: AppModule.CreateProxy(store, AppModule), // CreateProxy，一个vuex的proxy
}

export default store
```
3. vuex-class-component的简要介绍
   - module的用法
    ```js
    import { VuexModule, mutation, Module } from 'vuex-class-component'

    @Module({ namespacedPath: 'module_name/' }) // 这个module_name一定要和new Vuex.Store({ modules: { module_name:xxx } })一定要和modules内的key一模一样
    export default class AppModule extends VuexModule {
        ...
    }
    ```