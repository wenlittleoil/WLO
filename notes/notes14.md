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
