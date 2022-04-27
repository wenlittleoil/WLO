  
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
