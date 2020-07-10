## 8
问题：移动端h5页面在高清屏中1px显示过粗。  
解决方式：利用css的媒体查询或者js的window.devicePixelRatio获得设备像素比，根据该值，对1px元素进行y轴方向相应倍率的缩小。  
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
  
## 9
根据一个数组，排队逐个请求后端接口，每一次都等待上一个接口返回响应后才开始发起下一个接口请求。  
```  
  // 目标数组
  const idList = ids.split(/\s+/gi);

  // 请求开始，设置加载中
  this.setState({
    loading: true,
  });

  const results = await idList.reduce(async (prevReqPromise, id, idx) => {
    // 等待上一个接口请求的resolve，且resolve后的结果数组保存了之前所有的请求结果
    const middleResults = await prevReqPromise;

    // 组装当前这个请求的参数
    const bodyParams = {
      id,
    }

    // 发起当前请求
    try {
      await request.json(
        api.newUpdateClazzPlan,
        bodyParams,
      );

      // 当前请求的结果1(成功)
      const result = {
        status: 'success',
        id,
      };
      middleResults.push(result);
    } catch (error) {
      // 当前请求的结果2(失败)
      const result = {
        status: 'failture',
        id,
      };
      middleResults.push(result);
    }

    // 关键一步，将当前请求的结果放入中间数组中并返回
    return middleResults;
  }, Promise.resolve([]));

  // 所有请求都已结束，且所有的请求结果都放在results数组中，开始进行请求后的ui响应(例如弹窗通知用户多少个请求成功，多少个失败了)
  const failtureResults = results.filter(item => item.status === 'failture');
  if (failtureResults.length) {
    Modal.info({
      id: 
        `${failtureResults.length}个课时更新失败`,
      content: (
        <div
          style={{
            maxHeight: '600px',
            overflowY: 'scroll',
          }}
        >
          <h4>更新失败的课时列表：</h4>
          {failtureResults.map((item) => (
            <p 
              key={item.id}
            >
              课时:
              {item.id}
            </p>
          ))}
        </div>
      ),
    });
  } else {
    message.info('批量修改课时成功');
  }

  // 关闭loading
  this.setState({
    loading: false,
  });
```  

## 10  
nextjs生命周期
> 1、getInitialProps在客户端渲染时 ( h5的history事件页面跳转 ) 会在客户端执行一次，在服务端渲染时( 用户浏览器地址栏输入，刷新或者通过location跳转 ) 会在服务端执行一次。只会在其中一个端执行一次，不会同时在两端都执行一遍。  
> 2、componentwillmount和render在服务端渲染时，在服务端和客户端会先后都执行一次。但在客户端渲染时只会在客户端执行一次。  
> 3、componentdidmount无论客户端渲染还是服务端渲染，只会在客户端执行一次。  

## 11
Nginx反向代理配置
> sudo vim /etc/nginx/nginx.conf
```
http {

        server {
            listen 8000;
            server_name localhost;

            # For the API service
            location /api {
              proxy_pass http://localhost:8008;
              proxy_set_header Host $host;

              # proxy_hide_header Cache-Control;
              
              # Through Nginx Proxy, for browser,
              # both API and Resource request are on the same domain,
              # it doesn't need to set CORS headers,
              # so on the below, 
              # some headers from upstream server were set hidden to client.
              proxy_hide_header Access-Control-Allow-Credentials;
              proxy_hide_header Access-Control-Allow-Headers;
              proxy_hide_header Access-Control-Allow-Origin;
              
              # By default, some headers from upstream server, such as Server
              # etc was blocked, then Nginx use its own headers for client,
              # in this case, it's 'Server: nginx/1.14 (Ubuntu)'.
              # but now we ban such behavior, Server header will be totally
              # decided by the upstream server.
              proxy_pass_header Server;

            }

            # For the Frontend Resource service
            location / {
              proxy_pass http://localhost:8002;
            }
        }

}
```   
> sudo nginx -t  
> sudo nginx -s reload
    
  
## 12  
NodeJs流式压缩加密文件  
```  
const path = require('path');
const fs = require('fs');
const zlib = require('zlib');
const crypto = require('crypto');

const fileSrc = path.resolve(__dirname, './test2-bigfile.log');
const basename = path.basename(fileSrc, '.log');
const fileTar = `./${basename}.gz`;

const src = fs.createReadStream(fileSrc);
const tar = fs.createWriteStream(fileTar);
src.pipe(zlib.createGzip()).pipe(crypto.createCipher('aes192', '111222')).pipe(tar);

// src.on('end', () => {
//   fs.stat(fileTar, (err, stats) => {
//     console.log(err, stats)
//   })
// });
tar.on('finish', () => {
  fs.stat(fileTar, (err, stats) => {
    console.log(err, stats)
  })
});
```  
  
## 13  
模拟使用背景图撑开容器元素，css代码如下：  
```
.container{  
  /* target.jpg 1920*770 unit px */
  background: url('/target.jpg');
  
  background-repeat:no-repeat; 
  background-size:100% 100%;
  background-position-x: center;
  margin: 0 auto;
}
.container::after {
  content: "";
  display: block;
  
  /* `height/width` radio from target.jpg */
  padding-bottom: 40.1%;
}
```
  

  
