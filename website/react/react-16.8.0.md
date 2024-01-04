## hooks

[参考文章链接](https://zhuanlan.zhihu.com/p/486496578)

### useState、useEffect、useContext、useReducer

1. useState——让函数拥有内部状态

   1. ```jsx
      const [state,setState] = useState(initialState);
      ```

2. useEffect——可以看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合

   1. ```jsx
      useEffect(didUpdate,dependencies);
      //该hook接收一个包含命令式、且有可能有副作用代码的函数(didUpdate)
      //dependencies为[]时，仅函数第一次执行时执行。
      //不传，函数每次执行时都会重新执行。
      //传入依赖[a,b,...]函数每次执行时，如果依赖有变化，则重新执行第一个回调方法。
      
      //在依赖变化和销毁前可以执行清理函数。
      useEffect(
        //do update works
        return function(){
          //do clear works
        }
      );
      ```

3. useContext——让函数可以接收content,组件之间共享状态

   1. ```jsx
      const myContext = React.createContext();
      function TestUseState(){
          return (
              <myContext.Provider value={{name:"xxx",age:21}}>
                  <SubComponent />
              </myContext.Provider>
          );
      }
      function SubComponent(){
          const value = useContext(myContext);
          return (
              <div>
                  <myContext.Consumer>
                      {value=> <div>myName:{value.name}</div>}
                  </myContext.Consumer>
                  <div>myAge:{value.age}</div>
              </div>
          );
      }
      
      /*
       *React.createContext创建一个context类
       *myContext.Provider提供value（生产者）
       *myContext.Consumer和useContext(myContext)使用value（消费者）
       */
      ```

4. useReducer —— 独立状态管理

   1. 使我们的代码具有更好的可读性、可维护性、可预测性

   2. reducer是一个利用action提供的信息，将state从A转换到B的一个纯函数，具有一下几个特点：

      - 语法：(state, action) => newState

      - Immutable：每次都返回一个newState， 永远不要直接修改state对象

      - Action：一个常规的Action对象通常有type和payload（可选）组成

      - - type： 本次操作的类型，也是 reducer 条件判断的依据
        - payload： 提供操作附带的数据信息

   3. ```jsx
      const [state,dispatch] = useReducer(reducer,initialArg,init);
      function reducer(state,action){
        switch(action.rype){
          case actionOne:
            funcOne();
            break;
          ...
        }
      }
      dispatch({type:action});
      //state 状态，dispatch 触发状态更新，reducer 状态修改器
      ```

   ### useCallback、useMemo、React.memo

   1. useCallback —— 缓存函数

      1. ```jsx
         const memoizedCallback = useCallback(()=>{
           doSomething(a,b);
         },[a,b]);
         ```

   2. useMemo —— 缓存计算结果

      1. 传递一个创建函数和依赖项，创建函数会需要返回一个值，只有在依赖项发生改变的时候，才会重新调用此函数，返回一个新的值

      2. ```jsx
         const memoizedValue = useMemo(
           ()=>{
             computeExpensiveValue(a,b)
           }
         ,[a,b]);
         /*
          *第一个函数参数会在渲染期间执行
          *第二个参数依赖列表是执行条件，和useEffect一致
          */
         ```

   3. React.memo

      1. React.memo()是一个高阶函数，它与[React.PureComponent](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttps%3A%2F%2Freactjs.org%2Fdocs%2Freact-api.html%23reactpurecomponent)类似，但是一个函数组件而非一个类。React.memo()可接受2个参数，第一个参数为纯函数的组件，第二个参数用于对比props控制是否刷新，与[shouldComponentUpdate()](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttps%3A%2F%2Freactjs.org%2Fdocs%2Freact-component.html%23shouldcomponentupdate)功能类似。当我们对父组件进行重新渲染时有时我们并不需要子组件进行重新渲染，这是就可以使用memo来判断子组件是否需要渲染

      2. 

      3. ```jsx
         const Child = memo(({data}) =>{
             return (
                 <div>
                     <div>child</div>
                     <div>{data.name}</div>
                 </div>
             );
         })
         ```

   4. 缓存比较（memo>useCallback>useMemo）

      1. 优先做纯函数组件优化
      2. 再缓存传入的方法
      3. 最后做精细化的渲染缓存控制

   5. useRef —— 函数存储器

      1. 引用某个dom节点或者某个类组件

      2. ```jsx
         React.createRef()//创建一个ref
         <input ref="inputRef" /> //this.refs['inputRef']来访问
         <CustomInput ref="comRef" /> //this.refs['comRef']来访问
         ```

      3. 回调函数

         1. 回调函数就是在dom节点或组件上挂载函数，函数的入参是dom节点或组件实例，达到的效果与字符串形式是一样的，都是获取其引用。回调函数的触发时机：

            1. 组件渲染后，即componentDidMount后
            2.  组件卸载后，即componentWillMount后，此时，入参为null
            3.  ref改变后

         2. ```jsx
            <input ref={(input) => {this.textInput = input;}} type="text" />//1.dom节点上使用回调函数
            <CustomInput ref={(input) => {this.textInput = input;}} /> //2.类组件上使用
            ```

         3. 