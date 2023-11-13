## 38.
简单封装pc和移动浏览器h5手势操作库gesture
```
type TGestureType = 'left' | 'right' | 'up' | 'down';

type TGestureListener = () => void;

type TFunc = (type: TGestureType, listener: TGestureListener) => void;

interface IGesture {
  init: () => void;
  handle: () => void;
  listeners: Array<{ type: TGestureType, listener: TGestureListener }>,
  on: TFunc,
  off: TFunc,
}

let touchstartX = 0;
let touchstartY = 0;
let touchendX = 0;
let touchendY = 0;

const gesture: IGesture = {
  init() {
    if ("ontouchstart" in window && "ontouchend" in window) {
      // 大部分pc和移动浏览器
      window.addEventListener('touchstart', function (event) {
        const touch = event.changedTouches[0];
        touchstartX = touch.clientX;
        touchstartY = touch.clientY;
      }, true);
      window.addEventListener('touchend', function (event) {
        const touch = event.changedTouches[0];
        touchendX = touch.clientX;
        touchendY = touch.clientY;
  
        if (!gesture.listeners.length) return;
        gesture.handle();
      }, true);
    } else if ("onmousedown" in window && "onmouseup" in window) {
      // mac Safari不支持touchstart/touchend事件，需要使用mousedown/mouseup兼容
      window.addEventListener("mousedown", (event) => {
        touchstartX = event.clientX;
        touchstartY = event.clientY;
      }, true);
      window.addEventListener("mouseup", (event) => {
        touchendX = event.clientX;
        touchendY = event.clientY;

        if (!gesture.listeners.length) return;
        gesture.handle();
      }, true);
    }
  },
  handle() {
    // 缓冲值，避免用户不小心轻微滑动导致事件触发，单位为设备独立像素
    const X_BUF = 10;
    const Y_BUF = 15;
    // 滑动起点和终点在x和y轴方向的差值
    const xDiff = Math.abs(touchendX - touchstartX);
    const yDiff = Math.abs(touchendY - touchstartY);

    const call = (type: TGestureType) => gesture.listeners.forEach(item => {
      item.type === type && item.listener();
    });
    
    if (touchendX < touchstartX) {
      console.log('手指向左滑-left');
      xDiff > X_BUF && call('left');
    }
    if (touchendX > touchstartX) {
      console.log('手指向右滑-right');
      xDiff > X_BUF && call('right');
    }
    if (touchendY < touchstartY) {
      console.log('手指向上滑-up');
      yDiff > Y_BUF && call('up');
    }
    if (touchendY > touchstartY) {
      console.log('手指向下滑-down');
      yDiff > Y_BUF && call('down');
    }
  },
  listeners: [],
  on: (type, listener) => {
    const index = gesture.listeners.findIndex(item => 
      item.type === type && item.listener === listener
    );
    if (index === -1) {
      gesture.listeners.push({
        type,
        listener
      });
    }
  },
  off: (type, listener) => {
    const index = gesture.listeners.findIndex(item => 
      item.type === type && item.listener === listener
    );
    if (index > -1) {
      gesture.listeners.splice(index, 1);
    }
  },
}

gesture.init();

export default Object.freeze(gesture);
```
