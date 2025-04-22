## 51.
使用命令行工具ffmpeg处理源视频文件video.mp4，从而生成hls协议的m3u8索引和ts片段文件
```
ffmpeg -i ./video.mp4 -codec: copy -start_number 0 -hls_time 10 -hls_list_size 0 -f hls output.m3u8
```

## 52.
如何在android 14真机上调试react-native的debug包  
开发平台：macOS Monterey版本12.6.5  
react-native版本：0.73.6  
第一步：在rn项目上运行`yarn start && yarn android`，即`react-native start && react-native run-android`。  
第二步：在rn项目上找到包`android/app/build/outputs/apk/debug/app-debug.apk`并手动传输、安装到您的android真机设备上。  
第三步：使用扩展坞数据线将您的android真机连接到macOS笔记本电脑，并打开android真机的开发者选项和允许USB调试。  
第四步：在macOS另一个终端命令行执行`adb devices`查看当前电脑总共连接了几台adb设备，输出结果如下
```
List of devices attached
10AE1K1SYA00148	device
emulator-5554	device
```
例如上述输出结果中的`10AE1K1SYA00148`即为您的android真机，然后命令行执行`adb -s 10AE1K1SYA00148 reverse tcp:8081 tcp:8081`将您的android真机端口代理到macOS电脑，从而可以访问本地dev server进行开发调试。  
第五步：重新启动您android真机上已安装的apk应用。

## 53.
微信小程序Taro中，滑动列表时，动态侦测当前可视的列表元素
```
  // 当前可视的列表元素索引
  const [activeIndex, setActiveIndex] = useState<number>();
  useEffect(() => {
    // 滑动时切换可视元素索引
    const observer = Taro.createIntersectionObserver(
      Taro.getCurrentInstance().page as PageInstance,
      {
        thresholds: [1],
        observeAll: true,
        initialRatio: 1, // 初始时完全呈现也不触发观测回调
      }
    );
    const observeTarget = '.item'; // 观测元素
    const observeCallback = (res) => {
      if (res.intersectionRatio === 1) {
        console.log(res.id + "列表元素完全进入视口");
        const indexStr = res.id?.replace("item-", "");
        if (indexStr) {
          const index = Number(indexStr);
          setActiveIndex(index);
        }
      }
    }
    observer.relativeToViewport().observe(observeTarget, observeCallback);
    return () => {
      observer?.disconnect();
    }
    // list为源数据列表
  }, [list]);

  // 渲染源数据列表list
  <ScrollView
    style={{
      width: "100vw",
      height: "100vh",
    }}
    scrollY
    scrollWithAnimation
  >
    {list.map((item, index) => {
      return (
        <View
          key={item.id}
          className="item"
          // data-index={index} // 不支持自定义的data-*属性
          id={`item-${index}`}
          style={{
            width: "100%",
            height: "100px",
          }}
        >
          列表元素项{index}
        </View>
      )
    })}
  </ScrollView>
```

## 54.
微信公众号h5网页的js-sdk授权(适用于手机微信打开h5网页的场景)
- [微信js-sdk文档](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)
```
import sha1 from 'sha1';

// 生成特定长度的随机字符串
const generateRandomString = (length: number) => {
  let result = '';
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  const charactersLength = characters.length;
  let counter = 0;
  while (counter < length) {
    result += characters.charAt(Math.floor(Math.random() * charactersLength));
    counter += 1;
  }
  return result;
}

const getSignature = (tempObj) => {
  const {
    jsapi_ticket,
    noncestr,
    timestamp,
    url,
  } = tempObj;
  const str = `jsapi_ticket=${jsapi_ticket}&noncestr=${noncestr}&timestamp=${timestamp}&url=${url}`;
  return sha1(str);
}

export const wxConfig = async () => new Promise(async (resolve, reject) => {
  // 参考微信js-sdk文档，wx.config的noncestr及timestamp字段与生成signature的参数应一致
  const noncestr = generateRandomString(8);
  const timestamp = Math.floor(Date.now() / 1000); // 秒级时间戳
  const url = window.location.href.split('#')[0]; // 当前h5网页的url路径，不支持路由History模式

  try {
    /*
    *  调用业务服务端接口getJsApiTicket获取jsapi_ticket，
    *  参考https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html#62
    */
    const jsapi_ticket = await getJsApiTicket();
    const signatureOptions = {
      jsapi_ticket,
      noncestr,
      timestamp,
      url,
    }
    const signature = getSignature(signatureOptions);
    const configOptions = {
      debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
      appId: "your_wx_appid", // 必填，微信公众号的唯一标识
      timestamp, // 必填，生成签名的时间戳
      nonceStr: noncestr, // 必填，生成签名的随机串
      signature,// 必填，签名，见 附录-JS-SDK使用权限签名算法
      jsApiList: [ // 必填，需要使用的JS接口列表，凡是要调用的接口都需要传进来
        'scanQRCode',
      ],
    }
    console.log('[wx.config - 配置参数]:', {
      ...configOptions,
      signatureOptions,
    });
    wx.config(configOptions);
    wx.ready(function(){
      /**
       * config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，
       * config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，
       * 则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，
       * 不需要放在ready函数中。
       */
      console.log('[wx.config - ready]');
      resolve(undefined);
    });
    wx.error(function(res){
      /**
       * config信息验证失败会执行error函数，如签名过期导致验证失败，
       * 具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，
       * 对于SPA可以在这里更新签名。
       */
      console.log('[wx.config - error]', res);
      reject(res);
    });
  } catch (error) {
    reject(error);
  }
});
```
使用方法如下
```
// 在要使用微信js-sdk方法的页面
useEffect(() => {
  const initWx = async () => {
    // 进入要使用微信开放API的页面后，在调用js-sdk的api前，先完成wx.config的初始化
    try {
      await wxConfig();
    } catch(err) {
      console.log('微信js-sdk初始化报错', err);
      return;
    }
  }
  initWx();
}, []);
// 调用微信js-sdk的扫一扫功能
<button
  onClick={() => {
    wx.scanQRCode({
      needResult: 1, // 默认为0，扫描结果由微信处理，1则直接返回扫描结果，
      scanType: ["qrCode","barCode"], // 可以指定扫二维码还是一维码，默认二者都有
      success: function (res) {
        const result = res.resultStr; // 当needResult 为 1 时，扫码返回的结果
        console.log("微信扫描结果: ", result);
      },
      fail: function (res) {
        console.log("微信扫描失败: ", res);
      },
    });
  }}
>
  调用微信扫一扫
</button>
```



