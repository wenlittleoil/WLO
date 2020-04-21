## 8
问题：移动端h5页面在高清屏中1px显示过粗。  
解决方式：利用css的媒体查询或者js的window.devicePixelRatio获得设备像素比，根据该值，对1px元素进行y轴方向的缩放。  
css示例：
```
// 对.target-element元素设置底部1px边框
.target-element {
  &:after {
    content: '';
    display: block;
    width: 100%;
    height: 1px;
    background-color: #e5e5e5;
    position: absolute;
    bottom: 0;
    left: 0;
    transform: scale(1, 1);
    @media (-webkit-min-device-pixel-ratio: 2) {
      transform: scale(1, 0.5);
    }
    @media (-webkit-min-device-pixel-ratio: 3) {
      transform: scale(1, 0.33);
    }
  }
}
```  
  
  
