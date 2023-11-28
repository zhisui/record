
# 取消Axios接口请求
## 场景

取消请求主要运用于在 tab 切换和路由切换时候两种情景下：

-   tab 下拉框切换请求，如果在某个 tab 中接口请求耗时比较长，接口数据未返回的情况下切换到另外的 tab,假设这个 tab 的
    数据请求很快，而过一段时间后请求耗时长的数据回来了，则会更新当前的数据，则显示的数据便错乱，
    所以需要在切换 tab 的时候取消当前 tab 的请求。
-   路由快速跳转时取消上个页面的接口请求的，否则在一些情况下接口请求时间过长会导致页面卡顿。

## 利用 axios 的 cancelToken 取消请求

axios 官方提供了两种取消请求的[方法](https://github.com/axios/axios#cancellation)，通过 `AbortController` 和 `CancelToken`，AbortController 从 v0.22.0 版本开始支持，CancelToken 从 v0.22.0 就已经废弃了，由于项目是使用的 v0.22.0 以下的 版本，故使用 CancelToken 实现，后期开新项目可能会去尝试下 AbortController。

CancelToken 提供了两种使用方法：

-   通过 CancelToken.source();

```js
const CancelToken = axios.CancelToken
const source = CancelToken.source()

axios
    .get('/user/12345', {
        cancelToken: source.token,
    })
    .catch(function (thrown) {
        if (axios.isCancel(thrown)) {
            console.log('Request canceled', thrown.message)
        } else {
            // handle error
        }
    })

axios.post(
    '/user/12345',
    {
        name: 'new name',
    },
    {
        cancelToken: source.token,
    }
)

// cancel the request (the message parameter is optional)
source.cancel('Operation canceled by the user.')
```

这种方法比较适合取消单个接口的请求

-   通过实例化 CancelToken 构建函数

```js
const CancelToken = axios.CancelToken
let cancel

axios.get('/user/12345', {
    cancelToken: new CancelToken(function executor(c) {
        // An executor function receives a cancel function as a parameter
        cancel = c
    }),
})

// cancel the request
cancel()
```

执行上面的 cancel 函数就可以取消请求。

### 在项目中运用

由于项目中要处理多个接口，所以第二种方法比较适合。

## 思路

在 axios 请求拦截器中为需要处理的接口依次建立一个 CancelToken 实例，在回调函数里面将 cancel 函数存储在 vuex 中的一个数组里面，在页面跳转时候去依次执行这个 cancel，执行完之后将将这个数组清空，以便进行下次操作。

## 实现

store.js

```js
import Vue from 'vue'
import Vuex from 'vuex'

const state = {
    cancelTokenArr: [], //每一个cancel函数
}

const mutation = {
    AddToken(state, payLoad) {
        state.cancelTokenArr.push(payLoad.cancelToken)
    },
    ClearToken({ cancelTokenArr }) {
        cancelTokenArr.forEach((cancel) => {
            if (cancel) {
                cancel('路由跳转取消请求')
            }
        })
        cancelTokenArr = []
    },
    EmptyToken(state, payLoad) {
        state.cancelTokenArr = []
    },
}

export default new Vuex.Store({
    state,
    mutations,
})
```

在请求拦截器中

```js
import axios from 'axios'
import store from '@/stores'

axios.interceptors.request.use((config) => {
    if (!config.url.includes('findUser')) {
        config.cancelToken = new axios.CancelToken(function (cancel) {
            //将cancel存起来
            store.commit('AddToken', { cancelToken: cancel })
        })
    }
    return config
})
```

可搭配路由跳转使用或者在某些特定操作中使用，比如说切换请求接口的时候

```js
router.beforeEach((to, from, next) => {
    if (%%%%) {
            store.commit('ClearToken')
            store.commit('EmptyToken')
            next()
        } else {
        next()
    }
})
```
