# 1
iphone微信使用base64显示图片，不支持background-image，应使用img的src显示。

# 2
// 点击底部按钮时,滚动至顶  
// this.pageTopEle为页顶元素
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
