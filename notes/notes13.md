## 40.
记住页面历史状态（用于导航前进返回/刷新等场景下回显原状态）
1. 自定义react hook
```
// useHistoryState.ts
import { useState } from 'react'
import { useHistory } from 'react-router'
// import { useHistory } from 'umi'

// key must be uniq in global history state object
export default function useHistoryState<T>(key: string, initialValue: T): [T, (t: T) => void] {
  const history = useHistory()
  const [rawState, rawSetState] = useState<T>(() => {
    const value = (history.location.state as any)?.[key]
    console.log('history-init', value)
    return value ?? initialValue
  })
  function setState(value: T) {
    const oldState = history.location.state as any
    const newState = {
      ...oldState,
      [key]: value
    }
    console.log('history-setState', oldState, newState)
    history.replace({
      ...history.location,
      state: newState
    })
    rawSetState(value)
  }
  return [rawState, setState]
}
```
2. 使用方法类似useState，如下：
```
import useHistoryState from './useHistoryState'
const [stateValue, setStateValue] = useHistoryState('pageName_componentName_stateName', 'initialStateValue')
```

## 41.
使用browser fetch api异步请求进行上传和下载文件
1. 上传文件
```
    if (uploading) return
    setUploading(true)

    const apiUrl = `/api/file/upload`
    const formData = new FormData()
    formData.append('myFileFieldName', file as File)

    fetch(apiUrl, {
      method: 'POST',
      body: formData,
      // 浏览器会自动根据file类型追加合适的请求标头Content-Type,例如: multipart/form-data; boundary=—-WebKitFormBoundaryfgtsKTYLsT7PNUVD
      headers: {},
    })
      .then(async (res) => {
        const resObj = await res.json()
        console.log('上传返回结果', res, resObj)
        if (+resObj?.code == 0) {
          console.log('文件上传成功')
        } else {
          throw resObj
        }
      })
      .catch((err) => {
        console.error('文件上传失败', err)
      })
      .finally(() => {
        setUploading(false)
      })
```
2. 下载文件
```
    if (downloading) return
    setDownloading(true)

    const apiUrl = `/api/file/download`
    let filename: string
    fetch(apiUrl, {
      method: 'POST',
      body: JSON.stringify({ fileId }),
      headers: {
        'Content-Type': 'application/json; charset=utf-8',
      },
    })
      .then((res) => {
        // 前端和api必须同源才能访问受限制标头
        const disposition = res.headers.get('Content-Disposition')
        if (disposition) {
          filename = disposition.split(/;(.+)/)[1].split(/=(.+)/)[1]
          if (filename.toLowerCase().startsWith("utf-8''"))
            filename = decodeURIComponent(filename.replace(/utf-8''/i, ''))
          else filename = filename.replace(/['"]/g, '')
        }
        return res.blob()
      })
      .then((blob) => {
        const url = window.URL.createObjectURL(blob)
        const link = document.createElement('a')
        link.href = url
        // 从响应头部Content-Disposition中获取filename，若不存在，则由浏览器用户代理自行处理下载文件名兜底(用''表示)
        link.download = filename || ''
        link.click()
        window.URL.revokeObjectURL(url)
        console.log('[下载文件成功]')
      })
      .catch((err) => {
        console.log('[下载文件失败]:', err)
      }).finally(() => {
        setDownloading(false)
      })
```

## 42.
节流和防抖方法的封装
> 典型应用场景：移动端页面的滚动，导致scroll事件函数高频触发，而浏览器的渲染帧率仅为16ms/每次，频繁重绘导致页面卡顿，使用节流工具函数可减少页面的卡顿现象。
```
/**
 * 1.节流工具函数
 * @param {Function} fn 高频触发函数
 * @param {Number} interval 间隔执行时间 单位ms
 * @param {Boolean} useLastArg 是否使用最新参数调用 默认为false
 */
export const throttle = (fn, interval, useLastArg = false) => {
  let flag = false;
  let lastArgs = [];
  return function(...args) {
    const context = this;
    lastArgs = args;
    if (!flag) {
      flag = true;
      setTimeout(() => {
        const callArgs = useLastArg ? lastArgs : args;
        fn.apply(context, callArgs);
        flag = false;
      }, interval);
    }
  };
};
```  

> 典型应用场景：用户不断输入，高频触发onchange事件函数，此时要进行模糊查询时。  
```
/**
 * 2.防抖工具函数
 * @param {Function} fn 高频触发函数
 * @param {Number} delay 延迟执行时间 单位ms
 */
export const debounce = (fn, delay) => {
  let timer;
  return function(...args) {
    const context = this;
    if (timer) {
      window.clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
};
```

## 43.
echarts世界地图绘制（不需要引入百度地图导航功能）
```
import * as echarts from 'echarts'
import worldGeoJson from './world.json' // 标准地理地图数据，参考echarts文档相关说明

echarts.registerMap('world', worldGeoJson as any)
const chart = echarts.init(dom) // 地图挂载的div节点
const options: echarts.EChartsOption = {
  series: [
    {
        // 散点图(气泡图)
        type: 'scatter',
        coordinateSystem: 'geo',
        zlevel: 1,
        symbol: 'circle',
        // 自定义气泡大小
        symbolSize: function (val) {
          return 4 + val[2] / 100
        },
        tooltip: {
          show: true,
          borderWidth: 0,
          formatter: (params) => {
            const { data } = params
            const { name, value } = data
            return `<div>${name}: ${value[2]}</div>`
          }
        },
        // 气泡样式
        itemStyle: {
          borderColor: '#1890FF',
          color: '#1890FF',
          opacity: 0.6
        },
        // 序列数据
        data: [
          {
            name: "China",
            value: [116.20, 39.56, 1272], // [longitude, latitude, total]
          },
          {
            name: "America",
            value: [-122.8950075, 47.0451022, 649]
          },
        ],
    }
  ],
  // visualMap: {
  //   //图例值控制
  //   min: 10,
  //   max: 50,
  //   show: true,
  //   calculable: true,
  //   color: ['#ff9500'],
  //   textStyle: {
  //     color: '#fff'
  //   }
  // },
  backgroundColor: '#fff',
  // 地图及样式基础配置
  geo: {
    map: 'world',
    label: {
      show: false
    },
    emphasis: {
      itemStyle: {
        areaColor: '#2a333d66'
      },
      label: {
        show: false,
        color: '#fff'
      }
    },
    roam: 'move', // 允许鼠标横移漫游，不允许鼠标缩放
    zoom: 1, // 当前地图缩放值
    layoutCenter: ['50%', '50%'], //地图位置
    layoutSize: '180%',
    itemStyle: {
      areaColor: '#edf3f3',
      borderColor: '#9d9f9f'
    },
    tooltip: {
      show: false
    }
  },
  tooltip: {
    trigger: 'item'
  }
}
chart.setOption(options, true)
```

## 44.
基于antd Upload封装的上传图片组件
```
import React, { useEffect, useState } from "react";
import { PlusOutlined } from "@ant-design/icons";
import { Image, Upload } from "antd";
import type { GetProp, UploadFile, UploadProps } from "antd";
import { UploadFileStatus } from "antd/lib/upload/interface";
import uniqueId from "lodash/uniqueId";

type FileType = Parameters<GetProp<UploadProps, "beforeUpload">>[0];

const getBase64 = (file: FileType): Promise<string> =>
  new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result as string);
    reader.onerror = (error) => reject(error);
  });

interface IProps extends Omit<UploadProps, 'value' | 'onChange'> {
  value?: string[]; // 图片url列表
  onChange?: (value?: string[]) => void;
  limitFileList?: number; // 限制最多上传几张图片，默认只能上传单张
}

const UploadPictures: React.FC<IProps> = (props) => {
  const { value, onChange = () => {}, limitFileList = 1, ...restUploadProps } = props;
  const [previewOpen, setPreviewOpen] = useState(false);
  const [previewImage, setPreviewImage] = useState("");
  const [fileList, setFileList] = useState<UploadFile[]>([]);

  useEffect(() => {
    const list = (value || []).map((item) => ({
      uid: uniqueId("reveal_"),
      name: "Image",
      status: "done" as UploadFileStatus,
      url: item,
    }));
    setFileList(list);
  }, [value]);

  const handlePreview = async (file: UploadFile) => {
    if (!file.url && !file.preview) {
      file.preview = await getBase64(file.originFileObj as FileType);
    }

    setPreviewImage(file.url || (file.preview as string));
    setPreviewOpen(true);
  };

  const handleChange: UploadProps["onChange"] = ({ fileList: newFileList }) => {
    console.log("handleChange", newFileList);
    // 上传成功后才需要更新fileList，对于失败的上传直接跳过
    const list = newFileList
      .filter((item) => item.status !== "error")
      .filter(
        (item) => !(item.status === "done" && !item.url && !item.response?.data)
      );
    // 对于成功的上传，取出服务端返回的图片链接url
    const _list = list.map((item) => {
      if (item.status === "done" && item.response?.data && !item.url) {
        return {
          ...item,
          url: item.response?.data,
        };
      }
      return item;
    });
    // 更新组件内部状态fileList
    setFileList(_list);
    // 所有上传完毕后，才更新外部状态value
    if (_list.every((item) => !!item.url)) {
      onChange(_list.map((item) => item.url));
    }
  };

  const uploadButton = (
    <button style={{ border: 0, background: "none" }} type="button">
      <PlusOutlined />
      <div style={{ marginTop: 8 }}>上传</div>
    </button>
  );
  return (
    <>
      <Upload
        accept="image/*"
        action={"/api/upload"}
        name="file"
        headers={{}}
        listType="picture-card"
        fileList={fileList}
        onPreview={handlePreview}
        onChange={handleChange}
        {...restUploadProps}
      >
        {fileList.length >= limitFileList ? null : uploadButton}
      </Upload>
      {previewImage && (
        <Image
          wrapperStyle={{ display: "none" }}
          preview={{
            visible: previewOpen,
            onVisibleChange: (visible) => setPreviewOpen(visible),
            afterOpenChange: (visible) => !visible && setPreviewImage(""),
          }}
          src={previewImage}
        />
      )}
    </>
  );
};

export default UploadPictures;
```



