## 45.
计算不同规格值组合的笛卡尔乘积
```
const getSkuList = (attrList: any[][]) => {
  return attrList.reduce(
    (total, current) => total.flatMap((t) => current.map((c) => [...t, c])),
    [[]]
  );
};
const list = getSkuList([[2, 4], ["a", "b"], ["A", "B", "C"]]);
console.log(list);
```

## 46.
基于antd Upload封装的上传视频组件
```
```

## 47.
快速交换数组中的两项顺序(修改原数组)
```
const swapElements = (myArray: any[], index1: number, index2: number) => {
  [myArray[index1], myArray[index2]] = [myArray[index2], myArray[index1]];
};
const targetArr = [0, 1, 2, 3];
swapElements(targetArr, 0, 2);
console.log(targetArr);
```

## 48.
使用webpack第三方插件filemanager-webpack-plugin插件管理构建输出文件
```
  webpackChain(chain) {
    // 将项目工程下的your-target-dir文件拷贝输出到dist/your-target-dir
    const actions = [
      {
        source: 'your-target-dir', 
        destination: path.resolve(__dirname, `../dist/your-target-dir`)
      },
    ];
    chain.merge({
      plugin: {
        install: {
          plugin: require('filemanager-webpack-plugin'),
          args: [{
            events: {
              onEnd: {
                // move: actions, // 移动
                copy: actions, // 拷贝
              },
            },
          }],
        },
      },
    });
  }
```
## 49.
常用js正则表达式合集
```
const textInputRules = [
  { pattern: /(^\S)((.)*\S)?(\S*$)/, message: "前后不能出现空格" },
];

const onlyWordNumRules = [
  { pattern: /^[A-Za-z0-9]+$/, message: "只支持输入字母、数字" },
];

const onlyNumRules = [{ pattern: /^[0-9]+$/, message: "只支持输入数字" }];

const emailRegRules = [{
  pattern: /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
  message: "请输入正确的邮箱格式"
}];

const onlyChineseRules = [{ pattern: /^[u4e00-u9fa5]+$/, message: "只支持输入中文" }];

const excludeChineseRules = [{ pattern: /^[^\u4e00-\u9fa5]+$/, message: "不支持输入中文" }];
```
## 50.
微信小程序两种不同轮播图效果的实现（使用Taro框架）
```
// 1.左右滑动轮播

// 2.点击切换轮播
      <View className='interpic-container'>
        {interactiveImages?.map((picUrl, ind) => {
          return (
            <View 
              key={ind}
              onClick={() => {
                if (index === interactiveImages?.length - 1) {
                  // 当前处于最后一帧，则重新回到第一帧
                  setIndex(0)
    
                  const _animations = [...animations];
    
                  // 当前帧消失
                  animationInstance.opacity(0).step();
                  _animations[index] = animationInstance.export();
                  // 重新回到第一帧
                  animationInstance.opacity(1).step();
                  _animations[0] = animationInstance.export();
    
                  setAnimations(_animations);
                } else {
                  // 正常切换到下一帧
                  setIndex(index + 1);
  
                  const _animations = [...animations];
  
                  // 当前帧动画消失
                  animationInstance.opacity(0).step();
                  _animations[index] = animationInstance.export();
                  // 下一帧动画出现
                  animationInstance.opacity(1).step();
                  _animations[index + 1] = animationInstance.export();
  
                  setAnimations(_animations);
                }
              }}
              className={'interpic-item'}
              animation={animations[ind]} // 绑定动画效果
            >
              <Image 
                className="pic"
                src={picUrl}
                mode="aspectFill"
                lazyLoad
              />
            </View>
          )
        })}
      </View>
```

