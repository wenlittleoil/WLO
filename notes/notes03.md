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
 * build2Tree 方法
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
build2Tree(sourcePlatList, { id: 'id', pid: 'parentId', mapping: { key: 'id', title: 'name'}});
```






