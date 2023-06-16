# Vue.nextTick

## 问题复现

今天在开发中遇到一个问题，通过 v-if 渲染一个表格后，无法获取这个表格以及对其进行下一步操作，用一个demo演示就是：

```vue
<template>
    <div>
        <el-table v-if="tableVisible" ref="table">
            <el-table-column prop="xxx" label="xxx" width="180"></el-table-column>
        </el-table>
    </div>
</template>

<script>
export default {
    name: "Test",
    data() {
        return {
            tableVisible: false,
        }
    },
    methods: {
        showTable() {
            this.tableVisible = !this.tableVisible;
            const table = this.$refs['table'];
            console.log(table);
        }
    }
}
</script>
```

当我触发【showTable】方法时，获取到的变量 table 为 undefined，最终解决方法为：**将获取dom的操作放入 Vue.nextTick(callback) 中**，即

```javascript
showTable() {
    this.tableVisible = !this.tableVisible;
    this.$nextTick(() => {
        const table = this.$refs['table'];
        console.log(table)
    })
}
```

## 原理探究

在Vue官网中有这样一段话：

> 可能你还没有注意到，Vue 在更新 DOM 时是**异步**执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 `Promise.then`、`MutationObserver` 和 `setImmediate`，如果执行环境不支持，则会采用 `setTimeout(fn, 0)` 代替。
>
> 例如，当你设置 `vm.someData = 'new value'`，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。多数情况我们不需要关心这个过程，但是如果你想基于更新后的 DOM 状态来做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员使用“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们必须要这么做。为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 `Vue.nextTick(callback)`。这样回调函数将在 DOM 更新完成后被调用

**理解：数据发生变化后，DOM 不会立即发生变化，如果想立即获取变化后的 DOM，则需要将操作放入 Vue.nextTick 中，因为 DOM 完成渲染后会执行 Vue.nextTick(callback) 中的回调函数。**

![](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220925215142085.png)

## 用法

1. **Vue生命周期前3个钩子函数**（beforeCreate、created、beforeCreate）如果要**操作 DOM**，一定要放在 nextTick 中，否则直接报错，因为**此时 DOM 还没进行渲染**

   ```javascript
   beforeCreate() {
       this.$refs.a.innerText = 'Hello World!';
   }
   ```

   ![](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220925215147310.png)

   正确用法：

   ```javascript
   beforeCreate() {
       this.$nextTick(() => {
           this.$refs.a.innerText = 'Hello World!1';
       })
   }
   ```

2. **改变数据后，想获取到最新的 DOM**，如：

   ```vue
   <template>
       <div>
           <div ref="a">{{a}}</div>
       </div>
   </template>
   
   <script>
   
   export default {
       name: "Test",
       data() {
           return {
               a: 'hello'
           }
       },
       methods: {
           show() {
               this.a = 'world';
               const a = this.$refs['a'].innerText;
               console.log(a)
           }
       },
   }
   </script>
   ```

   触发show方法，第一次打印出的结果为hello，不是最新的DOM，需获取最新则需要将获取方法作为nextTick的回调函数，即

   ```javascript
   show() {
       this.a = 'world';
       this.$nextTick(() => {
           const a = this.$refs['a'].innerText;
           console.log(a)
       })
   }
   ```

## 拓展

**Vue.$nextTick返回的是一个Promise对象，因此可以使用ES2017 async/await完成相同的事情**，如

```javascript
async show() {
    this.a = 'world';
    await this.$nextTick()
    const a = this.$refs['a'].innerText;
    console.log(a)
}
```

此时输出的a也是最新的值 world，而不会出现 hello。

