## 33.
macos检索磁盘目录大小占用
```
sudo du -d 1 -h | sort -h
```

## 34.
每个依赖同时发生变化时才会执行useEffectAllDepsChange，与其中一个依赖发生变化就会执行的useEffect不同。
```
import { useEffect, useRef } from "react";

function usePrevious(value) {
  const ref = useRef<any[]>([]);
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}

export default function useEffectAllDepsChange(fn, deps) {
  const prevDeps = usePrevious(deps);
  const changeTarget = useRef<any[]>([]);
  useEffect(() => {
    // nothing to compare to yet
    if (changeTarget.current === undefined) {
      changeTarget.current = prevDeps;
    }
    // we're mounting, so call the callback
    if (changeTarget.current === undefined) {
      return fn();
    }
    // make sure every dependency has changed
    if (changeTarget.current.every((dep, i) => dep !== deps[i])) {
      changeTarget.current = deps;
      return fn();
    }
  }, [fn, prevDeps, deps]);
}
```
## 36.
微信小程序封装request方法
```
```
## 37.
android studio和idea项目工程mac快捷操作：  
1.在当前文件内搜索关键词
```
command + f
```
2.在整个项目工程内搜索关键词
```
command + shift + f
```
3.在整个项目工程内查找文件
```
command + o
```
4.在当前文件内搜索和替换关键词
```
command + r
```
5.在整个项目工程内搜索和替换关键词
```
command + shift + r
```

