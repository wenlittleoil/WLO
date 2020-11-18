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




