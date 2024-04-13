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
封装request方法(微信小程序/h5平台):  
一、Taro实现
```
import Taro, {
  hideLoading,
  request as TaroRequest,
  showLoading,
} from "@tarojs/taro";
import { toast } from '@/utils/tools'
import { loginBde } from '@/apis/auth-bde'

export type TReqOption = {
  autoLoading?: boolean;
  autoErrTip?: boolean;
  baseUrl?: string;
  needToken?: boolean;
  needLog?: boolean;
} & any;

export const TYPE = {
  BDE: 'BDE',
  SSC: 'SSC',
}

// 仅用于微信小程序静默登录
export type TokenInfo = { userId: number, accessToken: string };
export const getGlobalTokenInfo = (() => {
  let tokenInfo: Promise<TokenInfo>;
  return async () => {
    if (!tokenInfo) {
      tokenInfo = new Promise(async (resolve, reject) => {
        try {
          // wx.login
          const wxLoginRes = await Taro.login();
          // business login
          const res = await requestBDE({
            url: `/wx/login`,
            method: "POST",
            data: {
              wxCode: wxLoginRes.code
            },
            needToken: false, // 必传，登录无需token，若不传会导致死循环
          });
          resolve(res?.data as TokenInfo);
        } catch (error) {
          reject(error);
        }
      });
    }
    return tokenInfo;
  }
})();

export async function request<T>(
  options: TReqOption,
): Promise<T> {
  const {
    autoLoading = true,
    autoErrTip = true, 
    baseUrl,
    type,
    needToken = true,
    needLog = true,
  } = options;
  const newOptions = {
    ...options,
    url: `${baseUrl}${options?.url}`,
    header: {},
    timeout: 25000, // 25s超时
  }
  if (type === TYPE.BDE) {
    let token = '';
    if (needToken) {
      // token = Taro.getStorageSync('access_token'); // 常规登录
      token = (await getGlobalTokenInfo()).accessToken; // 微信小程序静默登录
    }
    newOptions.header = {
      ...newOptions.header,
      Authorization: `${token}`,
    }
  } else if (type === TYPE.SSC) {
    newOptions.header = {
      ...newOptions.header,
    }
  }

  const errTip = (message) => {
    toast(message || '请求失败');
  }

  try {
    if (autoLoading) {
      showLoading();
    }
    const response = await TaroRequest(newOptions);
    needLog && console.log('[response]', response);
    if (autoLoading) {
      hideLoading();
    }
    const { statusCode, data, } = response;
    const success = +data?.code === 0;
    if (!success) {
      if (autoErrTip) {
        const tip = data?.errMessage || data?.msg || data?.message;
        errTip(tip);
      }
      // if (
      //   type === TYPE.BDE && 
      //   data?.errCode === 40000
      // ) {
      //   // 登录态失效，重新获取code进行登录
      // }
      return Promise.reject(data);
    }
    return data;
  } catch (err) {
    console.log(`[response error]:`, err);
    if (autoLoading) {
      hideLoading();
    }
    if (autoErrTip) {
      const tip = err?.errMessage || err?.msg || err?.message;
      errTip(tip);
    }
    return Promise.reject(err);
  }
}

export const createRequest = (baseOptions: TReqOption) => async <T>(options: TReqOption) => {
  const _options = {
    ...baseOptions,
    ...options,
  }
  const res = await request<T>(_options);
  return res;
}

// 创建公共request方法
export const requestBDE = createRequest({
  baseUrl: BDE_API_URL,
  type: TYPE.BDE,
})

// 使用案例（业务请求）
export async function fetchProductList(reqArgs) {
  return requestBDE<Res>(
    {
      url: `/product/list`,
      method: "POST",
      data: reqArgs,
      autoLoading: false,
      needLog: false,
      autoErrTip: false,
    }
  );
}

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

