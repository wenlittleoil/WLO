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
## 50.
微信小程序两种不同轮播图效果的实现（使用Taro框架）
1. 左右滑动轮播
```
import {
  FC,
  useEffect,
  useMemo,
  useRef,
  useState,
} from 'react'
import Taro, {
  createVideoContext, pxTransform,
} from '@tarojs/taro'
import { 
  View, 
  Image,
  Video,
  Swiper,
  SwiperItem,
  ScrollView,
} from '@tarojs/components'
import classNames from 'classnames'
import { staticHost } from '@/config'
import CustomSwiperIndicator from '@/components/CustomSwiperIndicator'
import useIntersectionObserver from '@/hook/useIntersectionObserver'
import './CreativeColumns.scss'

export interface ISwiperItem {
  type: 'image' | 'video',
  url: string,
  cover?: string,
  dataItem?: any,
}

interface IProps {
  // 轮播列表
  swiperList: ISwiperItem[]
}

const CreativeColumns:FC<IProps> = (props) => {
  const {
    swiperList,
  } = props;

  // 轮播区元素ID
  const observerEleId = 'creative-swiper-wrapper';

  // 当前轮播处在哪项
  const [index, setIndex] = useState(0)
  const indexRef = useRef(0)

  const [isPlayVideo, setIsPlayVideo] = useState(false);
  const [inViewport, setInViewport] = useState(false); // 轮播区是否处于视口中

  const [videoMuted, setVideoMuted] = useState(true);

  const setVideoPlayStatus = (index: number, type: "play" | "pause" | "stop") => {
    let videoContext = createVideoContext(`hj-landing-creative-video-${index}`);
    if (type == "play") {
      videoContext.play();
    } else if (type == "pause") {
      videoContext.pause();
    } else if (type == "stop") {
      videoContext.stop();
    }
  };

  const onChange = (e) => {
    const current = e.detail.current;
    setIndex(current);
    indexRef.current = current;
    setIsPlayVideo(false);

    Taro.nextTick(() => {
      // 将所有其它视频暂停
      swiperList.forEach((sItem, sIndex) => {
        if (sIndex !== current && sItem.type === "video") {
          setVideoPlayStatus(sIndex, "pause");
        }
      });
      // 将当前视频播放
      const curItem = swiperList[current];
      if (curItem?.type === 'video') {
        setVideoPlayStatus(current, "play");
        setIsPlayVideo(true);
      }
    });
  }

  const [toChildViewId, setToChildViewId] = useState<string>()
  useEffect(() => {
    if (!swiperList?.length) return;
    Taro.nextTick(() => {
      const curId = `creative-tab-item-${index}`;
      setToChildViewId(curId);
    });
  }, [swiperList, index]);

  useEffect(() => {
    if (!swiperList?.length) return;
    // 进入视口后播放当前视频，离开视口后暂停当前视频。（初始状态下第一个视频会自动播放）
    const curItemIsVideo = swiperList?.[indexRef.current]?.type === "video";
    if (inViewport) {
      console.log("creative轮播区进入视口");
      if (curItemIsVideo) {
        setVideoPlayStatus(indexRef.current, "play");
        setIsPlayVideo(true);
      }
    } else {
      console.log("creative轮播区离开视口");
      if (curItemIsVideo) {
        setVideoPlayStatus(indexRef.current, "pause");
        setIsPlayVideo(false);
      }
    }
  }, [swiperList, inViewport]);

  useIntersectionObserver({
    type: "relativeToViewport",
    observeTarget: '#' + observerEleId,
    observeCallback: (res) => {
      console.log("observe-callback", res?.intersectionRatio);
      if (res.intersectionRatio > 0) {
        // 轮播区进入视口
        setInViewport(true);
      } else {
        // 轮播区离开视口
        setInViewport(false);
      }
    },
  });

  const activeItem = swiperList[index];
  const activeDataItem = activeItem?.dataItem;
  return (
    <View className='CreativeColumns'>
      <View className="sectionNameWrap">
        <ScrollView 
          className="sectionName"
          scrollX
          enableFlex
          scrollWithAnimation
          scrollIntoView={toChildViewId}
          showScrollbar={false}
          enhanced
        >
          {swiperList.map((item, ind) => {
            return (
              <View 
                id={`creative-tab-item-${ind}`}
                key={ind}
                className={classNames("sectionNameItem", {
                  "active": ind === index,
                })}
                onClick={() => {
                  if (ind !== index) {
                    setIndex(ind);
                    indexRef.current = ind;
                  }
                }}
              >
                {item?.dataItem?.sectionName}
              </View>
            );
          })}
        </ScrollView>
      </View>

      <View className='swiper-wrapper' id={observerEleId}>
        <Swiper
          className={'prod-pic-swiper'}
          indicatorDots={false}
          indicatorColor='#999'
          indicatorActiveColor='#333'
          autoplay={!isPlayVideo}
          current={index}
          onChange={onChange}
          circular // 无缝衔接轮播
          interval={5000} // 自动切换等待时间间隔
          duration={500} // 滑动动画时长
          nextMargin={pxTransform(35)} // 右侧留出多余部分显示下一帧轮播图
        >
          {swiperList.map((item, index) => {
            return (
              <SwiperItem 
                key={index}
                className='prod-pic-swiper-item'
                onClick={() => {
                  if (activeDataItem?.button?.redirectContent) {
                    jump(activeDataItem?.button?.redirectContent)
                  }
                }}
              >
                {item.type === 'image' && (
                  <Image 
                    className={'prod-pic-item'}
                    src={item.url}
                    mode="aspectFill"
                    lazyLoad
                  />
                )}
                {item.type === 'video' && (
                  <View className={classNames('prod-video-item')}>
                    <Video
                      id={`hj-landing-creative-video-${index}`}
                      className={'prod-video'}
                      objectFit='cover'
                      src={item.url}
                      poster={item.cover || undefined}
                      // onPlay={e => {
                      //   setIsPlayVideo(true);
                      // }}
                      // onPause={e => {
                      //   setIsPlayVideo(false);
                      // }}
                      onEnded={() => {
                        console.log("视频播放结束")
                        setIsPlayVideo(false);
                      }}
                      autoplay={false}
                      loop={false}
                      muted={videoMuted}
                      controls={false}
                      enableProgressGesture={false}
                      showFullscreenBtn={false}
                      showPlayBtn={false}
                      showCenterPlayBtn={false}
                    />
                    {/* {!isPlayVideo && (
                      <Image 
                        className={classNames('prod-video-play-icon')}
                        src={playIcon} 
                        onClick={() => {
                          setVideoPlayStatus('play');
                        }}
                      />
                    )} */}
                    <Image
                      className={'mute-btn'}
                      mode="aspectFit"
                      src={videoMuted ? `${staticHost}/muted.png?key=1` : `${staticHost}/mute.png?key=1`}
                      onClick={(e) => {
                        e.stopPropagation();
                        setVideoMuted(!videoMuted);
                      }}
                    />
                  </View>
                )}
              </SwiperItem>
            )
          })}
        </Swiper>
        <CustomSwiperIndicator 
          length={swiperList.length} 
          currentIndex={index} 
        />
      </View>
      <View className='btm-info'>
        {!!activeDataItem?.themeText && (
          <View className="themeText">{activeDataItem?.themeText}</View>
        )}
      </View>
    </View>
  )
}

export default CreativeColumns;

```
2. 点击切换轮播
```
      /* 状态逻辑 */
      const [interactiveImages, setInteractiveImages] = useState<string[]>([]); // 轮播图片列表
      const [index, setIndex] = useState(0); // 当前处在轮播图片列表的第几帧

      // 微信小程序动画实例
      const animationInstance = useMemo(() => {
        return Taro.createAnimation({
          duration: 1000,
          timingFunction: 'ease',
        });
      }, []);
      // 微信小程序动画列表
      const [animations, setAnimations] = useState<any[]>([]);
      // 初始动画状态
      useEffect(() => {
        if (!interactiveImages?.length) return;
        setAnimations(interactiveImages.map((url, ind) => {
          if (ind === 0) {
            // 第一帧图初始呈现
            animationInstance.opacity(1).step();
            return animationInstance.export();
          }
          return {}
        }));
      }, [interactiveImages]);

      /* jsx模板 */
      <View className="interpic-container">
        {interactiveImages?.map((picUrl, ind) => {
          return (
            <View 
              key={ind}
              onClick={() => {
                if (index === interactiveImages?.length - 1) {
                  // 当前处于最后一帧图，则重新回到第一帧图
                  setIndex(0)
    
                  const _animations = [...animations];
    
                  // 当前帧消失
                  animationInstance.opacity(0).step();
                  _animations[index] = animationInstance.export();
                  // 重新回到第一帧
                  animationInstance.opacity(1).step();
                  _animations[0] = animationInstance.export();
    
                  setAnimations(_animations);
                } else {
                  // 正常切换到下一帧图
                  setIndex(index + 1);
  
                  const _animations = [...animations];
  
                  // 当前帧动画消失
                  animationInstance.opacity(0).step();
                  _animations[index] = animationInstance.export();
                  // 下一帧动画出现
                  animationInstance.opacity(1).step();
                  _animations[index + 1] = animationInstance.export();
  
                  setAnimations(_animations);
                }
              }}
              className="interpic-item"
              animation={animations[ind]} // 给每一帧图片分别绑定自己的动画效果
            >
              <Image 
                className="pic"
                src={picUrl}
                mode="aspectFill"
                lazyLoad
              />
            </View>
          )
        })}
      </View>

      /* scss样式（750px设计稿） */
      .interpic-container {
        width: 610px;
        height: 610px;
        position: relative;
        .interpic-item {
          width: 100%;
          height: 100%;
          position: absolute;
          opacity: 0; // 默认隐藏
          .pic {
            width: 100%;
            height: 100%;
          }
        }
      }
```

