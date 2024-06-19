## 40.
记住页面历史状态（用于导航前进返回/刷新等场景下回显原状态）
1. 自定义react hook
```
// useHistoryState.ts
import { useState } from 'react'
import { useHistory } from 'react-router'
// import { useHistory } from 'umi'

// key must be uniq in global history state object
export default function useHistoryState<T>(key: string, initialValue: T): [T, (t: T) => void] {
  const history = useHistory()
  const [rawState, rawSetState] = useState<T>(() => {
    const value = (history.location.state as any)?.[key]
    console.log('history-init', value)
    return value ?? initialValue
  })
  function setState(value: T) {
    const oldState = history.location.state as any
    const newState = {
      ...oldState,
      [key]: value
    }
    console.log('history-setState', oldState, newState)
    history.replace({
      ...history.location,
      state: newState
    })
    rawSetState(value)
  }
  return [rawState, setState]
}
```
2. 使用方法类似useState，如下：
```
import useHistoryState from './useHistoryState'
const [stateValue, setStateValue] = useHistoryState('pageName_componentName_stateName', 'initialStateValue')
```

## 41.
使用browser fetch api异步请求进行上传和下载文件
1. 上传文件
```
    if (uploading) return
    setUploading(true)

    const apiUrl = `/api/file/upload`
    const formData = new FormData()
    formData.append('myFileFieldName', file as File)

    fetch(apiUrl, {
      method: 'POST',
      body: formData,
      // 浏览器会自动根据file类型追加合适的请求标头Content-Type,例如: multipart/form-data; boundary=—-WebKitFormBoundaryfgtsKTYLsT7PNUVD
      headers: {},
    })
      .then(async (res) => {
        const resObj = await res.json()
        console.log('上传返回结果', res, resObj)
        if (+resObj?.code == 0) {
          console.log('文件上传成功')
        } else {
          throw resObj
        }
      })
      .catch((err) => {
        console.error('文件上传失败', err)
      })
      .finally(() => {
        setUploading(false)
      })
```
2. 下载文件
```
    if (downloading) return
    setDownloading(true)

    const apiUrl = `/api/file/download`
    let filename: string
    fetch(apiUrl, {
      method: 'POST',
      body: JSON.stringify({ fileId }),
      headers: {
        'Content-Type': 'application/json; charset=utf-8',
      },
    })
      .then((res) => {
        // 前端和api必须同源才能访问受限制标头
        const disposition = res.headers.get('Content-Disposition')
        if (disposition) {
          filename = disposition.split(/;(.+)/)[1].split(/=(.+)/)[1]
          if (filename.toLowerCase().startsWith("utf-8''"))
            filename = decodeURIComponent(filename.replace(/utf-8''/i, ''))
          else filename = filename.replace(/['"]/g, '')
        }
        return res.blob()
      })
      .then((blob) => {
        const url = window.URL.createObjectURL(blob)
        const link = document.createElement('a')
        link.href = url
        // 从响应头部Content-Disposition中获取filename，若不存在，则由浏览器用户代理自行处理下载文件名兜底(用''表示)
        link.download = filename || ''
        link.click()
        window.URL.revokeObjectURL(url)
        console.log('[下载文件成功]')
      })
      .catch((err) => {
        console.log('[下载文件失败]:', err)
      }).finally(() => {
        setDownloading(false)
      })
```

## 42.
节流和防抖方法的封装
> 典型应用场景：移动端页面的滚动，导致scroll事件函数高频触发，而浏览器的渲染帧率仅为16ms/每次，频繁重绘导致页面卡顿，使用节流工具函数可减少页面的卡顿现象。
```
/**
 * 1.节流工具函数
 * @param {Function} fn 高频触发函数
 * @param {Number} interval 间隔执行时间 单位ms
 * @param {Boolean} useLastArg 是否使用最新参数调用 默认为false
 */
export const throttle = (fn, interval, useLastArg = false) => {
  let flag = false;
  let lastArgs = [];
  return function(...args) {
    const context = this;
    lastArgs = args;
    if (!flag) {
      flag = true;
      setTimeout(() => {
        const callArgs = useLastArg ? lastArgs : args;
        fn.apply(context, callArgs);
        flag = false;
      }, interval);
    }
  };
};
```  

> 典型应用场景：用户不断输入，高频触发onchange事件函数，此时要进行模糊查询时。  
```
/**
 * 2.防抖工具函数
 * @param {Function} fn 高频触发函数
 * @param {Number} delay 延迟执行时间 单位ms
 */
export const debounce = (fn, delay) => {
  let timer;
  return function(...args) {
    const context = this;
    if (timer) {
      window.clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
};
```


