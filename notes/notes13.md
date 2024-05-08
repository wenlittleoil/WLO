## 40.
记住页面历史状态（用于导航前进返回/刷新等场景下回显原状态）
1. 自定义react hook
```
// useHistoryState.ts
import { useState } from 'react'
import { useHistory } from 'react-router'
// import { useHistory } from 'umi'

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

