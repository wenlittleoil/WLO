## 15 
i.在html文档内，引入外部样式表时，在外部样式表内，引用相对地址的图片时，其相对位置是基于外部样式表的域名来定位的。例如以下文档：
> index.html in 'a.example.com'
```
<html>
  <head>
     <title>a.example.com</title>
     <link type="text/css" href="b.example.com/outside.css"></link>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```  
> outside.css in 'b.example.com'  
```
  #id {
    /* !这里的请求获取的图片将是'b.example.com/path/to/bg.jpg' */
    background-image: url('/path/to/bg.jpg');
  }
```  
ii.外部样式表的这种行为和外部脚本完全不同。引入的外部脚本执行时，仍然以宿主html文档环境为准，例如以下文档：
> index.html in 'a.example.com'
```
<html>
  <head>
     <title>a.example.com</title>
     <link type="text/javascript" href="b.example.com/outside.js"></link>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```  
> outside.js in 'b.example.com'
```
  console.log(window.location.origin, window.location.hostname); /* 仍然是'a.example.com' */
  location.href = '/path/to/your_page'; /* 这里的请求访问跳转的将是'a.example.com/path/to/your_page' */
```  
  

## 16
对于接口有调用频率限制的第三方服务，进行频率限流如下  
```
    for (let i = 0; i < list.length; i++) {
      await ctx.service.thirdPartyServiceApiCall({
         data: list[i]
      });
      // 限流，每80ms调用一次服务接口
      await new Promise((resolve) => {
        setTimeout(() => {
          resolve();
        }, 80);
      });
    }
```  

## 17
Javascript无法捕获异步错误
```
async function fn() {
  setTimeout(() => {
     throw new Error('发生了异步错误啦');
  }, 5000);
  console.log(111);
}
async function init() {
   try {
      await fn();
   } catch (error) {
      console.log('正常捕获了错误');
   }
}
init(); // Error无法被捕获，直接被抛出到顶层
```    
```
async function fn() {
    try {
       await new Promise((res, rej) => {
           setTimeout(() => {
               throw new Error('我的自定义错误');
           }, 5000);
       }); 
    } catch(error) {
        console.log('正常捕获了错误', error); // 无法捕获异步Error
    }
}
async function init() {
    await fn();
    console.log('因为前面发生了错误，永远不会被执行');
}
init();
``` 
相反，而以下错误能正常捕获到
```
async function foo() {
  await new Promise((res, rej) => {
         setTimeout(() => {
             console.log(111);
             res();
          }, 5000);
  });
  throw new Error('我是错误异常');
}
async function init() {
   try {
      await foo();
   } catch (error) {
      console.log('正常捕获了错误', error.message);
   }
}
init(); // 这次能正常地捕获到错误异常
``` 

## 18. 
```
/**
 * build2Tree 方法一
 * 将扁平化的数组nodes根据节点项id, pid 和 children 将一个个节点构建成一棵或者多棵树
 * 另外还支持对节点项对象添加额外的属性
 * @param nodes 节点对象数组
 * @param config 配置对象
 * @returns jsonTree 结果树的数组
 */
function build2Tree(nodes = [], config = {}) {
  const id = config?.id || 'id';
  const pid = config?.pid || 'pid';
  const children = config?.children || 'children';
  const mapping = config?.mapping || {};
  const attribute = config?.attribute || {};

  const idMap = {};
  const jsonTree = [];

  nodes.forEach((node) => { node && (idMap[node[id]] = node); });
  nodes.forEach((node) => {
    if (node) {
      Object.keys(mapping).map((item) => {
        node[item] = node[mapping[item]];
        return node;
      });
      Object.keys(attribute).map((item) => {
        node[item] = attribute[item];
        return node;
      });
      const parent = idMap[node[pid]];
      if (parent) {
        !parent[children] && (parent[children] = []);
        parent[children].push(node);
      } else {
        jsonTree.push(node);
      }
    }
  });
  return jsonTree;
}
```
使用方式如下:
```
const sourcePlatList = [{"id":"1","name":"深圳**科技有限公司","departmentId":"1","parentId":0,"count":27,"sortOrder":100000000},{"id":"5","name":"人力","departmentId":"5","parentId":1,"count":1,"sortOrder":99997000},{"id":"6","name":"产品","departmentId":"6","parentId":1,"count":4,"sortOrder":99996000},{"id":"7","name":"UI","departmentId":"7","parentId":6,"count":1,"sortOrder":100000000},{"id":"8","name":"PM","departmentId":"8","parentId":6,"count":3,"sortOrder":99999000},{"id":"9","name":"平面","departmentId":"9","parentId":7,"count":0},{"id":"10","name":"视觉","departmentId":"10","parentId":9,"count":0}];
const resultTreeList = build2Tree(sourcePlatList, { id: 'id', pid: 'parentId', mapping: { key: 'id', title: 'name'}});
console.log(resultTreeList);
```
```
/**
 * build2Tree 方法二
 */
function build2Tree(flatNodes = []) {
  // 先深拷贝原节点列表，避免影响到原节点列表
  const list = JSON.parse(JSON.stringify(flatNodes));

  // 对每个节点追加其子节点列表（节点间会通过引用关联在一起）
  list.forEach(item => {
    const childNodes = list.filter(node => node.parentId === item.id);
    if (!item.children) item.children = [];
    if (childNodes.length > 0) {
        childNodes.forEach(childNode => {
            item.children.push(childNode);
        });
    } 
  });

  // 只返回一级节点列表（即不存在父节点的那些节点为一级节点）
  return list.filter(item => list.every(i => i.id !== item.parentId));
}

const originalFlatNodes = [{"id":1,"name":"深圳**科技有限公司","departmentId":"1","parentId":0,"count":27,"sortOrder":100000000},{"id":5,"name":"人力","departmentId":"5","parentId":1,"count":1,"sortOrder":99997000},{"id":6,"name":"产品","departmentId":"6","parentId":1,"count":4,"sortOrder":99996000},{"id":7,"name":"UI","departmentId":"7","parentId":6,"count":1,"sortOrder":100000000},{"id":8,"name":"PM","departmentId":"8","parentId":6,"count":3,"sortOrder":99999000},{"id":9,"name":"平面","departmentId":"9","parentId":7,"count":0},{"id":10,"name":"视觉","departmentId":"10","parentId":9,"count":0}];

build2Tree(originalFlatNodes);

```

## 19. 
通过给margin负值，能使子元素向外扩张，例子如下：
```
<!DOCTYPE html>
<html lang="en">
<head>
  <title>测试css</title>
  <style>
    .father {
      width: 400px;
      height: 300px;
      background-color: lightgray;
      padding: 10px;
    }
    .son {
      height: 100px;
      background-color: cyan;

      /* 当给子元素margin负值时，能使内容向外扩张 */
      margin: -10px -10px 10px -10px;
    }
  </style>
</head>
<body>
  <div class="father">
    <div class="son"></div>
  </div>
</body>
</html>
```

## 20. 
全局设置浏览器自带的滚动条样式，例子如下：
```
/*滚动条凹槽轨道的颜色，还可设置边框属性 */
*::-webkit-scrollbar-track-piece {
  background-color: #f8f8f8;
  -webkit-border-radius: 2px;
  -moz-border-radius: 2px;
  border-radius: 2px;
}
/*滚动条的宽度*/
*::-webkit-scrollbar {
  width: 8px; // 垂直滚动条生效宽度
  height: 8px; // 水平滚动条生效高度
}
/*滚动条滚轴的设置*/
*::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, .15);
  background-clip: padding-box;
  -webkit-border-radius: 4px;
  -moz-border-radius: 4px;
  border-radius: 4px;
}
/*滚动条鼠标悬浮移上去*/
*::-webkit-scrollbar-thumb:hover {
  background-color: rgba(0, 0, 0, .15);
}
```


