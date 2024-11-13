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


