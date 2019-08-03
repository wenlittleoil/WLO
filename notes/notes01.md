## 1
iphone微信使用base64显示图片，不支持background-image，应使用img的src显示。

## 2
点击底部按钮时,滚动至顶  
this.pageTopEle为页顶元素
```
if (this.pageTopEle) {
  // 不兼容时降级
  if (this.pageTopEle.scrollIntoView) {
    /** 平滑滚动置顶 */
    this.pageTopEle.scrollIntoView({ block: 'start', behavior: 'smooth' });
  } else {
    /** 生硬滚动置顶 */
    const pageTop = this.pageTopEle.offsetTop;
    window.scrollTo(0, pageTop);
  }
}
```

## 3
对于移动端的touch系列事件(touchstart/touchend/touchcancel等)，不支持冒泡，其事件处理程序必须直接注册在dom树最底层的img元素上。并且对于长按事件的日志统计，为了解决android和ios下的兼容，touchend和touchcancel必须同时注册，因为手指离开屏幕后只会触发其中一个。demo如下：  
```
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import collectLog from '@/collectLog';

class ImgWithPressLog extends Component {
  constructor(props) {
    super(props);
    this.state = {};
    this.longTouch = false;
  }
  imgTouchStart = event => {
    this.longTouch = false;
    setTimeout(() => (this.longTouch = true), 500);
    const { onTouchStart } = this.props;
    this.extraEventHandler(onTouchStart, event);
  };
  imgTouchEnd = event => {
    this.handleTouchCollect();
    const { onTouchEnd } = this.props;
    this.extraEventHandler(onTouchEnd, event);
  };
  imgTouchCancel = event => {
    this.handleTouchCollect();
    const { onTouchCancel } = this.props;
    this.extraEventHandler(onTouchCancel, event);
  };
  extraEventHandler = (...args) => {
    const func = args[0];
    const params = args.slice(1);
    if (typeof func === 'function') {
      func.apply(this, params);
    }
  };
  handleTouchCollect = () => {
    if (this.longTouch) {
      const { logParams, } = this.props;
      if (logParams && typeof logParams === 'object') {
        collectLog(logParams);
        this.longTouch = false;
        return;
      }
    }
    this.longTouch = false;
  };
  render() {
    const {
      onTouchStart,
      onTouchEnd,
      onTouchCancel,
      alt,
      ...otherProps
    } = this.props;
    return (
      <img
        onTouchStart={event => this.imgTouchStart(event)}
        onTouchEnd={event => this.imgTouchEnd(event)}
        onTouchCancel={event => this.imgTouchCancel(event)}
        alt={alt}
        {...otherProps}
      />
    );
  }
}

ImgWithPressLog.propTypes = {
  logParams: PropTypes.object,
};

export default ImgWithPressLog;

```

