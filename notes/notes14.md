
```
const getSkuList = (attrList) => {
  return attrList.reduce(
    (total, current) => total.flatMap((t) => current.map((c) => [...t, c])),
    [[]]
  );
};
const list = getSkuList([[2, 4], ["a", "b"], ["A", "B", "C"]]);
console.log(list);
```

