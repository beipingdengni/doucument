## 安装使用

```sh
<script src="https://cdn.jsdelivr.net/npm/vue@2.7.14/dist/vue.js"></script>

# 开发时候使用
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

npm install vue@^2
```

### webpack引用

```js
var webpack = require('webpack')
module.exports = {
  // ...
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js' // 用 webpack 1 时需用 'vue/dist/vue.common.js'
    }
  },
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('production')
      }
    })
  ]
}
```



## method

> 传递参数中，event

```js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // 在 `methods` 对象中定义方法
  methods: {
    greet: function (message, event) {
      // `this` 在方法里指向当前 Vue 实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
        event.preventDefault()
      }
    }
  }
})
```

