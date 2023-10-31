## 基础绑定vue

```vue
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue之系列课程</title>
</head>
<body>
<div id="app">
    <h1>{{msg}}</h1>
    <!--3.使用局部组件-->
    <!--使用 v-bind 形式将数据绑定到Vue实例中data属性中 日后data属性发生变化 组件内部数据跟着变化-->
    <login :name="username" :age="age"></login>
</div>
</body>
</html>
<script src="js/vue.js"></script>
<script>
    //1.声明组件模板对象
    const login = {
        template: `<div><h2>欢迎：{{name}} 年龄:{{age}}</h2></div>`,
        props: ["name", "age"],//用来接收父组件给当前组件传递数据  注意:props机制接收数据就相当于自己组件data中声明一个这样数据
    }

    const app = new Vue({
        el: "#app",
        data: {
            msg: "组件之间数据传递：通过在组件上声明动态数据传递给组件内部",
            username: "hello china",
            age: 21,
        },
        methods: {},
        computed: {},
        components: {//注册组件
            login,//2.注册局部组件
        }
    })
</script>

```

### 全局注册组件

```js
 Vue.component(`login`, {
    template: `<div><h2>用户登录</h2> <form action=""></form></div>`
     components: {
         // 依赖组件添加
    },
  	props: ["username", "age"], //props作用 用来接收使用组件时通过组件标签传递的数据
    data() { //用来给当前组件定义属于组件自己数据  组件中定义数据  data必须是一个函数
        return {}
    },
    methods: { //用来给组件自己定义一系列方法
        testMethod(count) {}
    },
    computed: { //用来给组件自己定义一些列计算方法
        counterCpt(){}
    },
    watch:{
      msg (newValue,oldValue){
        console.log("new:",newValue,"old:",oldValue)
      },
      // msgs:{
      //         handler:'logMsg',        //方法名
      //         deep:true,               //深度观察：对任何数据发生变化，watch方法都会被触发
      //         immediate:true         //立即调用：在侦听开始时立即调用一次watch方法
      //     },
      // 'msg.sender':['logMsg','logLine']   //数组方式，可调用多个方法
    },
    //初始化阶段
    beforeCreate() {
        console.log("beforeCreate:", this.msg);
    },
    created() {//在这个函数执行时Vue实例对象已经存在自身内部事件和生命周期函数以及自定义data methods computed
        console.log("created:", this.msg);
    },
    beforeMount() { //此时组件中template还是模板还没有渲染
        console.log(this);
        //console.log("beforeMount:",this.$el.innerHTML);
    },
    mounted() {  // 此时组件中页面的数据已经和data中数据一致
        console.log("mounted:", document.getElementById("aa").innerHTML);
    },
    //运行阶段
    beforeUpdate() {// 此时data中数据变化了,页面数据还是原始数据
        console.log("beforeUpdate:", this.counter);
        console.log("beforeUpdate:", document.getElementById("aa").innerHTML);
    },
    updated() {  //此时data 页面数据一致
        console.log("updated:", this.counter);
        console.log("updated:", document.getElementById("aa").innerHTML);
    },
    //销毁阶段
    beforeDestroy() {
    },
    destroyed() {
    }
});
```

### 局部注册组件

```js
// 局部组件登录模板声明
let login = { // 具体局部组件名称
	template: `<div><h2>用户登录</h2></div>`
}

const app = new Vue({
  el: "#app",
  data: {
  	msg: "Vue中组件之局部组件的使用"
  },
  methods: {},
  computed: {},
  components: { // 用来注册局部组件
    // login: login,
    // 简写
    login, // 登录局部组件
    add: { // 添加局部组件
    	template: `<div><h2>用户添加</h2> <form action=""></form></div>`
    }
  }
  })
```

