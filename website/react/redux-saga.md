## 安装

```sh
npm install redux-saga
```

## 配置中间件

store/index.js

```jsx
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'
import { composeWithDevTools } from '@redux-devtools/extension'

//导入的redux-saga并不是像redux-thunk一样是一个对象，而是一个方法
const sagaMiddleware = createSagaMiddleware();
//使用该方法创建一个saga对象
const storeEnhancer =  applyMiddleware(thunkMiddleware);

//告诉系统使用中间件
// const store = createStore(reducer, storeEnhancer);
const store = createStore(reducer, composeWithDevTools(applyMiddleware(storeEnhancer)))

// 将saga和reducer联系起来  //将saga运行起来
sagaMiddleware.run();

export default store
```

## 配置使用（在生成器函数中获取网络数据）

```jsx
import {takeEvery, put} from 'redux-saga/effects'
import {GET_USER_INFO} from './constants';
import {changeAction} from './action';

// sagas.js
// saga中间件 主saga，用于区别是否需要saga来处理异步操作，如果没有异步，则放行
function* mainSaga() {
  // 这样的调用，它只能监听一个saga，不能进行模块化
  yield watchSaga()
}

// 监听saga，监听type类型为异步操作名称的，此saga会通过主saga分配过来
function* watchSaga() {
   // 指定需要拦截的action类型, 拦截到这个类型的action之后交给谁来处理
   yield takeEvery(GET_USER_INFO, workSaga)
}
// 工作saga，监听saga得到任务后，把任务分配给工作saga
function* workSaga({data}) {
    // 获取网络数据
    const data = yield fetch('http://127.0.0.1:7001/info',{
      method: 'POST',
      headers: new Headers({
      	'Content-Type': 'application/x-www-form-urlencoded' // 指定提交方式为表单提交
      }),
      body: new URLSearchParams([["foo", 1],["bar", 2]]).toString()
      })
    .then((response)=>{return response.json();}).catch((error)=>{console.log(error);});
  	// 以上方法可以使用如下（loginApi是封装了异步网络请求）
  	// let ret = yield call(loginApi, payload)
    // 保存获取到的数据 相当于 store.dispatch(changeAction(data));
    yield put(changeAction(data));
}
// 主saga要对外暴露出去
export default mainSaga
```

如果我们只需要拦截一个类型的action, 那么直接通过 **`yield takeEvery / yield takeLatest`**即可;但是如果我们想同时拦截多个类型的action, 那么我们就必须借助另外一个函数**`all()`**

如果我们只需要保存一个数据, 那么直接通过 **`yield put`** 即可但是如果我们想同时保存多个数据 , 那么我们就必须借助另外一个函数**`all()`**

```jsx
// yield put(changeAction(data));
yield all([
    yield put(changeUserAction(data1)),
    yield put(changeInfoAction(data2)),
    yield put({type:'CHANGE_USER_NAME', name: data1.name})
    yield put({type:'CHANGE_USER_Age', name: data1.age}),
])

// yield takeEvery(GET_USER_INFO, myHandler)
// yield takeLatest(GET_USER_INFO, myHandler)
yield all([
    yield takeEvery(GET_USER_INFO, myHandler),
    yield takeEvery(ADD_COUNT, myHandler),
    yield takeEvery(SUB_COUNT, myHandler),
]);
```

## takeEvery和takeLatest

- 区别: 是否能够完整的执行监听方法
  - 对于**`takeEvery`**而言, 每次拦截到对应类型的action, 都会完整的执行监听方法
  - 对于**`takeLatest`**而言, 每次拦截到对应类型的action, 都不能保证一定能够完整的执行监听方法

> 例如: 连续派发了3次**`GET_USER_INFO`**的action,那么对于**`takeEvery`**而言, myHandler就会被完整的执行3次;那么对于takeLatest而言, 如果派发下一次同类型action的时候,上一次派发的action还没有处理完, 也就是上一次的监听方法还没有处理完,那么**`takeLatest`**会放弃还没有处理完的代码, 直接开始处理下一次的action

#### 