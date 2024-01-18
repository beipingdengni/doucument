

```jsx
import {combineReducers,applyMiddleware, createStore} from 'redux'
import {composeWithDevTools} from 'redux-devtools-extension'
import {connect} from 'react-redux'
import thunk from 'redux-thunk'
```



> redux-thunk作用:    默认情况下dispatch只能接收一个对象,使用redux-thunk可以让dispatch除了可以接收一个对象以外, 还可以接收一个函数，是通过dispatch派发一个函数的时候能够去执行这个函数, 而不是执行reducer函数



### 使用案例

```jsx
// combineReducers，用于汇总多个reducer
// 引入createStore,专门用于创建Redux最为核心的store对象
import {combineReducers,applyMiddleware,compose,createStore} from 'redux'
import thunk from 'redux-thunk'

// 引入redux开发工具
import {composeWithDevTools} from 'redux-devtools-extension'
// 或者使用
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) : compose


//汇总所有的reducer变为一个总的reducer 
//引入为form组件服务的reducer
import form_reducer from './form_reducer'
//引入为list组件服务的reducer
import list_reducer from './list_reducer'
const reducer= combineReducers({form_reducer,list_reducer})


// 暴露store，并将对开发者工具和redux-thunk的支持添加进去
export default createStore(reducer,composeWithDevTools(applyMiddleware(thunk)))

// 替换上面使用
const enhancer = composeEnhancers(applyMiddleware(thunk))// 使用thunk中间件
const store = createStore(reducer, enhancer)
export default store


```

### form_reducer.jsx

```jsx
import {OPEN_FORM,CLOSE_FORM} from '../constant'
/**
 * form组件的reducer
 */
// 初始状态
let initSate = {
    isOpen:false //是否打开表单
}
export default function formReducer(state=initSate,action){
    console.log(action)
    switch(action.type){
        case OPEN_FORM: // 显示表单
            return {...state,isOpen:true} // 注意更新状态的写法
        case CLOSE_FORM: // 隐藏表单
            return {...state,isOpen:false}
        default:
            return state
    }
}
```



### 组件引入redux

```jsx
import React, { Component } from 'react'
// 引入connect高阶函数
import {connect} from 'react-redux'
// 引入form的action对象，进而操作form的显示与隐藏
import {openForm,openFormAsync} from '../../store/actions/form'

class Header extends Component {
		// 同步打开表单
    openForm = () => {
        // 在props中调用映射后的函数，即可更改状态
        this.props.openForm(null)
    }
		// 异步打开表单
    openFormAsync = () => {
        this.props.openFormAsync(null)
    }
    render() {
        return (
            <div>
                <h3>Header</h3>
                <button onClick={this.openForm}>同步打开表单</button>
                <button onClick={this.openFormAsync}>异步打开表单</button>
            </div>
        )
    }
}
const mapStateToProps = (state) => {
    return {
        name:state.form_reducer.isOpen
    }
}
const mapDispatchToProps = {
    openForm,
    openFormAsync
}
//或
//const mapDispatchToProps = dispatch => {
//  return {
//    addTodo: text => dispatch(addTodo(text)),
//    toggleTodo: id => dispatch(toggleTodo(id)),
//    deleteTodo: id => dispatch(deleteTodo(id))
//  };
//};
// 通过connect函数进行包装  
export default connect(mapStateToProps,mapDispatchToProps)(Header)
```



### redux-thunk

```jsx
// 参数是 dispatch， state
export const getUserInfo = (dispatch, getState) =>{
    fetch('http://127.0.0.1:7001/info')
        .then((response)=>{
            return response.json();
        })
        .then((data)=>{
            // console.log('在action中获取到的网络数据', data);
            dispatch(changeAction(data));
        })
        .catch((error)=>{
            console.log(error);
        })
}

const mapDispatchToProps = (dispatch) =>{
    return {
        decrement(){
            dispatch(subAction(1)); // 发送对象
        },
        changeInfo(info){
            dispatch(getUserInfo);  // 发送方法
        }
    }
};
```

