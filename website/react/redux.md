

```jsx
import React from 'react'
import { useDispatch, useSelector } from 'react-redux'
// useSelector : 读取redux的state的数据
// useDispatch : 修改redux的state的数据

const Login = () => {
  const dispatch = useDispatch()
  let num = useSelector(state => state.count.num)

  return (
    <div>
      <h3>{num}</h3>
      <button onClick={() => dispatch({ type: 'asyncAdd', payload: 10 })}>进入系统</button>
    </div>
  )
}

export default Login

```

