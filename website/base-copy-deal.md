

https://blog.csdn.net/Umbrella_Um/article/details/111067720



### 复制文本操作

> 1、navigator.clipboard 需要浏览器支持，且未https才能使用
>
> 2、document.execCommand('copy') 只能操作当前文档内的数据

```js
/**
 * 复制文本
 * @param {String} text
 */
export function copyText(text = '') {
  try {
    return navigator.clipboard
      .writeText(text)
      .then(() => {
        return Promise.resolve()
      })
      .catch(err => {
        console.error('复制失败：', err)
        return Promise.reject(err)
      })
  } catch (e) {
    // console.log('navigator.clipboard', e)
    let input = document.createElement('input')
    input.style.position = 'fixed'
    input.style.top = '-10000px'
    input.style.zIndex = '-999'
    document.body.appendChild(input)
    console.log('input', input)
    input.value = text
    input.focus()
    input.select()
    try {
      let result = document.execCommand('copy')
      document.body.removeChild(input)
      if (!result || result === 'unsuccessful') {
        return Promise.reject('复制失败')
      } else {
        return Promise.resolve()
      }
    } catch (e) {
      document.body.removeChild(input)
      return Promise.reject(
        '当前浏览器不支持复制功能，请检查更新或更换其他浏览器操作'
      )
    }
  }
}
```

### 可以使用clipboard.js

官网：https://clipboardjs.com/

中文简介：https://zhuanlan.zhihu.com/p/337309625

参考博客：https://blog.csdn.net/lalala_dxf/article/details/128374242

参考博客：https://zhuanlan.zhihu.com/p/617588309?utm_id=0



### vue 操作复制

```jsx
npm install vue-clipboards --save

import VueClipboards from 'vue-clipboards';
Vue.use(VueClipboards);

<button v-clipboard="copyText" @success="handleSuccess" @error="handleError">Copy</button>

data() {
 return {
      copyText: '复制的文本
     }
 },
 methods: {
     handleSuccess(e) {
         console.log(e);
     },
     handleError(e) {
         console.log(e);
     },
 }

```

