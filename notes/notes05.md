  
## 27. 
手动实现Promise
```
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function MyPromise(executor) {
  this.state = PENDING;
  this.value = null;
  this.reason = null;
  this.onFulfilledCallbacks = [];
  this.onRejectedCallbacks = [];
  const resolve = (value) => {
    setTimeout(() => {
      this.state = FULFILLED;
      this.value = value;
      this.onFulfilledCallbacks.forEach(callback => callback(this.value));
    });
  }
  const reject = (reason) => {
    setTimeout(() => {
      this.state = REJECTED;
      this.reason = reason;
      this.onRejectedCallbacks.forEach(callback => callback(this.reason));
    });
  }
  try {
    executor(resolve, reject);
  } catch(error) {
    reject(error);
  }
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
  // 处理特殊参数情况，增强健壮性
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };

  if (this.state === FULFILLED) {
    return new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          const ret = onFulfilled(this.value);
          resolve(ret);
        } catch(error) {
          reject(error);
        }
      });
    });
  }
  if (this.state === REJECTED) {
    return new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          const ret = onRejected(this.reason);
          resolve(ret);
        } catch(error) {
          reject(error);
        }
      });
    });
  }
  if (this.state === PENDING) {
    return new MyPromise((resolve, reject) => {
      this.onFulfilledCallbacks.push(value => {
        try {
          const ret = onFulfilled(value);
          resolve(ret);
        } catch(error) {
          reject(error);
        }
      });
      this.onRejectedCallbacks.push(reason => {
        try {
          const ret = onRejected(reason);
          resolve(ret);
        } catch(error) {
          reject(error);
        }
      });
    });
  }
}

MyPromise.prototype.catch = function(onRejected) {
  if (this.state === REJECTED) {
    return new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          const ret = onRejected(this.reason);
          resolve(ret);
        } catch(error) {
          reject(error);
        }
      });
    });
  }
  if (this.state === PENDING) {
    return new MyPromise((resolve, reject) => {
      this.onRejectedCallbacks.push(reason => {
        try {
          const ret = onRejected(reason);
          resolve(ret);
        } catch(error) {
          reject(error);
        }
      });
    });
  }
}
```
  
   
## 28. 
异步请求下载资源文件
```
import { message } from "antd";

// 从响应头content-disposition中解析出后端返回的资源文件名(带扩展的，下载到本地，方便电脑识别)
const getFilename = (req: XMLHttpRequest) => {
  const disposition =
    req.getResponseHeader("content-disposition") ||
    req.getResponseHeader("Content-Disposition");
  let filename = "资源文件";
  const filenameRegex = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/;
  const matches = filenameRegex.exec(disposition || "");
  if (matches != null && matches[1]) {
    filename = matches[1].replace(/['"]/g, "");
    filename = decodeURI(filename);
  }
  return filename;
};

// 将一个json文件(Blob大对象)读取为一个普通的json对象
const readBlobAsJsonObj = async (blob: Blob) => {
  return new Promise((resolve, reject) => {
    const fr = new FileReader();
    fr.onload = event => {
      const str = event?.target?.result;
      try {
        resolve(str ? JSON.parse(str as string) : null);
      } catch (err) {
        reject(err);
      }
    };
    fr.readAsText(blob);
  });
};

const downloadFile = async (url: string) => {
  return new Promise((resolve, reject) => {
    const req = new XMLHttpRequest();
    req.open("GET", url, true);
    req.responseType = "blob";
    // 关键：请求头中追加登录鉴权令牌(这点是通过href链接下载无法做到的，href链接下载只能支持cookie校验)
    req.setRequestHeader("Authorization", localStorage.getItem("token"));
    req.onload = async function(event) {
      const blob = req.response;
      if (blob.type !== "application/octet-stream") {
        // 返回的响应体并非想要的二进制文件，例如application/json等后端返回信息，则reject出去供业务调用侧处理具体错误
        const resObj = await readBlobAsJsonObj(blob);
        reject(resObj);
        return;
      }
      // 下载资源文件流程
      const filename = getFilename(req);
      const link = document.createElement("a");
      link.href = window.URL.createObjectURL(blob);
      link.download = filename;
      link.click();
      resolve({ filename, event });
    };
    // 请求过程出错
    req.onerror = function(event) {
      message.error("文件下载失败");
      reject(event);
    };
    req.send();
  });
};

export default downloadFile;
```
   
    
