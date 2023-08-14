## 32.
抖音视频列表实现原理
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
import { View, Swiper, SwiperItem, } from "@tarojs/components";
import {
  IPostItem,
  getRecommendList,
} from "@/apis/posts"
import { toast } from '@/utils/tools'
import Loading from "./Loading"
import VideoItem, {
  TOpenState,
} from './VideoItem'
import styles from "./ExploreList.module.scss";

type TPosts = Array<IPostItem | null>

const ExploreList = () => {
  // 虚拟列表（原始数据，即存储在内存中的总动态列表）
  const [virtualPosts, setVirtualPosts] = useState<TPosts>([]) 

  // 当前显示的那条post动态在virtualPosts中的索引位置
  const [indexInVirtualPosts, setIndexInVirtualPosts] = useState(0)
  // virtualPosts进行分页加载的页码，用于前端标示请求批次，后端暂时没有用到该参数
  const [pageNum, setPageNum] = useState(1)
  //  是否virtualPosts进行分页加载中
  const [loadingMore, setLoadingMore] = useState(false)
  const loadingMoreRef = useRef(false)
  const setLoadingState = (bol: boolean) => {
    setLoadingMore(bol)
    loadingMoreRef.current = bol;
  }

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
      fetchVirtualPosts({
        pageNum,
      })
    }
  }

  const fetchVirtualPosts = async (args: { 
    pageNum: number,
  }) => {
    const {
      pageNum,
    } = args;
    if (loadingMoreRef.current) return;
    setLoadingState(true)
    // 调接口获取视频动态列表
    try {
      const params = {
        pageSize: 10, // 单批次获取智能推荐10条视频
        pageNum, // 请求页码，第几批次
      }
      console.log('[拉取虚拟列表入参]:', params)
      const res = await getRecommendList(params);
      const list = res?.data || [];
      const latestList = pageNum === 1 ? list : [...virtualPosts, ...list];
      setVirtualPosts(latestList);
      setPageNum(pageNum + 1);
    } catch(err) {
      toast(err?.message || '获取视频列表失败');
    } finally {
      setLoadingState(false)
    }
  }

  // 点赞或取消点赞后更新状态
  const onActionSuccess = (actionType, dataItem: IPostItem) => {
    if (actionType === 'collectCancel') {
      dispatch(updateIsCollect({
        dataItem,
        isCollect: false,
      }))
    } else if (actionType === 'collect') {
      dispatch(updateIsCollect({
        dataItem,
        isCollect: true,
      }))
    }
  }

  return (
    <View className={styles["post-list-comp_wrap"]}>
      <Swiper 
        indicator-dots={false}
        autoplay={false} 
        vertical={true}
        className="f-full-screen"
        current={indexInVirtualPosts}
        onChange={onChange}
      >
        {renderPosts.map((item, index) => {
          return (
            <SwiperItem key={index}>
              {item === null && <View className="f-full-screen" />}
              {item !== null && (
                <VideoItem 
                  index={index}
                  dataItem={item}
                  ref={instance => videoItemRefs.current[index] = instance}
                  isCurrent={index === indexInVirtualPosts}
                  onActionSuccess={onActionSuccess}
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

export default ExploreList


```
