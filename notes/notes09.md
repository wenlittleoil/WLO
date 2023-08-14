## 32.
抖音视频列表实现原理(以Taro/React/微信小程序为案例)  
1、VideoList组件：
```
import { 
  FC, 
  useEffect, 
  useState, 
  useRef,
} from "react";
import Taro, {
  useLoad,
} from "@tarojs/taro";
import {
  View,
  Swiper,
  SwiperItem,
} from "@tarojs/components";
import {
  IVideoItem,
  fetchVideoList,
} from "@/apis/posts"
import Loading from "./Loading"
import VideoItem, {
  TOpenState,
} from './VideoItem'
import styles from "./VideoList.module.scss";

type TPosts = Array<IVideoItem | null>

const VideoList = () => {
  // 虚拟列表（原始数据，即存储在内存中的总动态列表）
  const [virtualPosts, setVirtualPosts] = useState<TPosts>([]) 
  // 当前显示的那条post动态在virtualPosts中的索引位置
  const [indexInVirtualPosts, setIndexInVirtualPosts] = useState(0)
  // virtualPosts进行分页加载的页码，标示请求批次
  const [pageNum, setPageNum] = useState(1)
  // 虚拟列表总数
  const [total, setTotal] = useState(0)
  //  是否virtualPosts进行分页加载中
  const [loadingMore, setLoadingMore] = useState(false)
  const loadingMoreRef = useRef(false)
  const setLoadingState = (bol: boolean) => {
    setLoadingMore(bol)
    loadingMoreRef.current = bol;
  }
  // 虚拟列表是否还有更多
  const notHasMore = virtualPosts?.length >= total;

  // 实际渲染列表（真正渲染到页面中的展示动态列表，最多只有三项，前后以null填充）
  const [renderPosts, setRenderPosts] = useState<TPosts>([])
  // 视频实例列表
  const videoItemRefs = useRef<Array<TOpenState | null>>([]);

  useEffect(() => {
    // 当前视频的上一条视频索引
    const startIndex = Math.max(0, indexInVirtualPosts - 1);
    // 当前视频的下一条视频索引
    const endIndex = Math.min(indexInVirtualPosts + 1, virtualPosts.length - 1);
    // 真正渲染到页面中的视频项(包括当前视频，总共3条)
    const realRenderList = virtualPosts.slice(startIndex, endIndex + 1);

    // 渲染新的视频列表
    const newRenderList = [
      ...virtualPosts.slice(0, startIndex).fill(null),
      ...realRenderList, 
      ...virtualPosts.slice(endIndex + 1).fill(null),
    ];
    setRenderPosts(newRenderList)
  }, [indexInVirtualPosts, virtualPosts])

  useEffect(() => {
    // 新的渲染完成后，将之前的视频暂停，然后播放新切换后的那条视频，即当前indexInVirtualPosts视频
    if (!renderPosts.length) return;
    const prevIndex = indexInVirtualPosts - 1;
    const nextIndex = indexInVirtualPosts + 1;
    if (prevIndex >= 0) {
      const prevVideoItem = videoItemRefs?.current[prevIndex];
      prevVideoItem?.playing && prevVideoItem?.operateVideo('pause');
    }
    if (nextIndex < virtualPosts.length) {
      const nextVideoItem = videoItemRefs?.current[nextIndex];
      nextVideoItem?.playing && nextVideoItem?.operateVideo('pause');
    }
    const currentVideoItem = videoItemRefs?.current[indexInVirtualPosts];
    !currentVideoItem?.playing && currentVideoItem?.operateVideo('play');
  }, [renderPosts])

  useLoad(() => {
    // 刚进入页面初始化，获取视频列表，注意初始加载是在onLoad的时候
    fetchVirtualPosts({
      pageNum,
    });
  })

  const onChange = (e) => {
    // 切换视频
    const current = e.detail.current
    setIndexInVirtualPosts(current)
    // 如果已滑动到最后一个视频，则调接口加载更多视频到虚拟列表
    if (current === virtualPosts.length - 1) {
      if (notHasMore) return;
      fetchVirtualPosts({
        pageNum,
      })
    }
  }

  const fetchVirtualPosts = async (args: { pageNum: number, }) => {
    const { pageNum, } = args;
    if (loadingMoreRef.current) return;
    setLoadingState(true)
    try {
      // 调接口获取视频动态列表
      const params = {
        pageSize: 10, // 单批次获取智能推荐10条视频
        pageNum, // 请求页码，第几批次
      }
      console.log('[拉取虚拟列表入参]:', params)
      const res = await fetchVideoList(params);
      const list = res?.data || [];
      const latestList = pageNum === 1 ? list : [...virtualPosts, ...list];
      setVirtualPosts(latestList);
      setPageNum(pageNum + 1);
      setTotal(res?.total || 0);
    } catch(err) {
      toast(err?.message || '获取视频列表失败');
    } finally {
      setLoadingState(false)
    }
  }

  return (
    <View className={styles["post-list-comp_wrap"]}>
      <Swiper 
        indicator-dots={false}
        autoplay={false} 
        vertical={true}
        className="n-full-screen"
        current={indexInVirtualPosts}
        onChange={onChange}
      >
        {renderPosts.map((item, index) => {
          return (
            <SwiperItem key={index}>
              {item === null && <View className="n-full-screen" />}
              {item !== null && (
                <VideoItem 
                  index={index}
                  dataItem={item}
                  ref={instance => videoItemRefs.current[index] = instance}
                  isCurrent={index === indexInVirtualPosts}
                />
              )}
            </SwiperItem>
          )
        })}
      </Swiper>
      {loadingMore && pageNum === 1 && <Loading position="center" />}
      {loadingMore && pageNum > 1 && <Loading position="bottom" />} 
    </View>
  )
}

export default VideoList;
```
2、VideoItem组件：  
```
import { 
  useState, 
  forwardRef, 
  useImperativeHandle,
  memo,
  useMemo,
  ReactNode,
  useEffect,
  useRef,
} from "react";
import Taro from "@tarojs/taro";
import { 
  View,
  Image,
  Video,
} from "@tarojs/components";
import cx from "classnames";
import {
  IVideoItem,
} from "@/apis/posts"
import styles from "./VideoItem.module.scss";

export interface IVideoItemProps {
  index: number, // 视频项在虚拟列表中的索引位置
  dataItem: IVideoItem, // 视频项
  isCurrent?: boolean, // 是否为当前屏的视频
}

// 对视频进行播放或暂停
export type TOperateType = 'play' | 'pause';

export type TOperateVideoFunc = (
  type: TOperateType, 
) => void;

export type TOpenState = {
  operateVideo: TOperateVideoFunc,
  playing: boolean,
}

const VideoItem = forwardRef<TOpenState, IVideoItemProps>((props, ref) => {
  const {
    index,
    dataItem,
    isCurrent = true,
  } = props

  const [playing, setPlaying] = useState(false)
  const operateVideo: TOperateVideoFunc = type => {
    // 对指定索引位置的那条视频进行播放或暂停
    const videoContext = Taro.createVideoContext(`post-vid-${index}`);
    if (type === 'play') {
      setPlaying(true);
      videoContext.play();  
    } else if (type === 'pause') {
      setPlaying(false);
      videoContext.pause();
    }
  }

  const onTap = () => {
    if (playing) operateVideo('pause');
    else operateVideo('play');
  }

  useImperativeHandle(ref, () => {
    return {
      operateVideo,
      playing,
    }
  }, [playing])

  return (
    <View className="n-relative">
      <View 
        className="video-wrapper" 
        onTap={onTap}
      >
        <Video 
          className="n-full-screen"
          id={`post-vid-${index}`}
          src={dataItem?.video?.url}
          autoplay={false}
          loop={true}
          show-fullscreen-btn={false}
          show-center-play-btn={false}
          show-play-btn={false}
          objectFit="cover"
          onLoadedMetaData={console.log}
        />
      </View>
      {/* 视频中央的播放图标按钮 */}
      {isCurrent && !playing && (
        <Image
          className={cx(styles['play-icon'], 'n-absolute n-z9')}
          src={`${STATIC_HOST}/icon_play.png`}
          onTap={onTap}
        />
      )}
      {/* 视频内其他悬浮元素 */}
      <View></View>
    </View>
  )
});

export default memo(VideoItem)
```
