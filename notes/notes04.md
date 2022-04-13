## 21. 
部署web站点脚本：
```
# 前端项目构建完成后，查看当前项目目录有如下输出，其中 dist/ 为产物目录
$ ls -al 
  drwxr-xr-x    5 wen  staff     160  3 25 00:32 dist
  -rw-r--r--    1 wen  staff     126  3 21 19:55 README.md
  drwxr-xr-x  769 wen  staff   24608  3 25 23:03 node_modules
  -rw-r--r--    1 wen  staff     876  3 25 00:29 package.json
  drwxr-xr-x    4 wen  staff     128  3 25 00:32 src
  ...
  
# 将产物 dist/ 覆盖同步到远程生产机的相应nginx资源服务目录，其中 --delete 表示会先删除远程目标目录原有文件
$ rsync --delete -avz dist/ root@192.168.0.1:~/app/blog/frontend/dist/    # command1

# 注意上述脚本命令`command1`需要提前先配置构建机和生产机之间的免密登录，或者使用如下方式：
$ sudo apt-get -y install sshpass
$ sshpass -p "$PROD_PASSWORD" rsync --delete -avz dist/ root@192.168.0.1:~/app/blog/frontend/dist/    # command2

# 注意若上述脚本命令`command2`出现错误`Host key verification failed. rsync error: unexplained error...`则用如下方式：
$ rsync --rsh="sshpass -p $PROD_PASSWORD ssh -o StrictHostKeyChecking=no -l root" --delete -avzr dist/ root@192.168.0.1:~/app/blog/frontend/dist/
```  
  
## 22. 
交换数组中的两项位置顺序
```
function getSwitchPositionArr(list: any[], sourceIndex: number, targetIndex: number) {
  const arr = [...list];
  arr[sourceIndex] = arr.splice(targetIndex, 1, arr[sourceIndex])[0];
  return arr;
}
```

## 23. 
对于一个内部可滚动的元素容器，但内部不显示滚动条的css设置
```css
.scrollable_container_element {
  scrollbar-width: none;
  -ms-overflow-style: none;
  &::-webkit-scrollbar {
    display: none;
  }
}
```

## 24. 
node和浏览器环境中js执行顺序机制：同步代码->异步微任务->异步宏任务->结束
```
if (typeof setImmediate !== 'undefined') {
  setImmediate(() => {
    // 异步宏任务
    console.log('immediate');
  });
}

async function async1() {
  // 同步执行
  console.log('async1 start');
  // 同步执行
  await async2();
  // 异步微任务
  console.log('async1 end');
}

async function async2() {
  // 同步执行
  console.log('async2');
}

// 同步执行
console.log('start');

setTimeout(() => {
  // 异步宏任务 优先级比setImmediate高
  console.log('timeout');
}, 0);

// 同步执行
async1();

new Promise((resolve) => {
  // 同步执行
  console.log('promise fn');
  resolve();
}).then(() => {
  // 异步微任务，相当于async函数内部await下一行后面代码，优先级一样，按注册顺序来
  console.log('promise then');
});

if (typeof process !== 'undefined') {
  process.nextTick(() => {
    // 异步微任务 优先级最高
    console.log('nextTick');
  });
}

// 同步执行
console.log('end');
```  
控制台如下输出：
``` 
  start
  async1 start
  async2
  promise fn
  end
  nextTick
  async1 end
  promise then
  timeout
  immediate
```
  
## 25. 
css布局技术：‘上背景图+中间可动态根据内容高度拉伸背景图+下背景图’。  
html代码如下：
```
    <div class="with-bg-box">
      <div class="box-body">
        {children}
      </div>
    </div>
```
相应的css代码如下：
```
.with-bg-box {
  width: 88px; // 大外盒宽度
  min-height: 47px; // 上图高度+下图高度
  background-image: url('./imgs/top.png'), url('./imgs/bottom.png');
  background-repeat: no-repeat, no-repeat;
  background-position: center top, center bottom;
  background-size: 100% auto;
  position: relative;
  &::before {
    content: '';
    position: absolute;
    left: 0;
    right: 0;
    margin: -1px 0;
    background: url('./imgs/middle.png') repeat-y;
    background-size: 100% auto;
    z-index: -1;
    top: 38px; // 上图高度
    bottom: 9px; // 下图高度
  }
  .box-body {
    margin: 0 4px;
  }
}
```
  
  
