## 31.
无限加载列表原理实现（以Taro/React/h5为案例）  
1、封装InfiniteLoadList.tsx组件
```
import {
  useState,
  useRef,
  ReactNode,
  forwardRef,
  useImperativeHandle,
  ForwardedRef,
  ReactElement,
} from 'react'
import { 
  View, 
  ScrollView,
} from '@tarojs/components'
import classnames from 'classnames'

export type TLoadMoreTrigger = 'auto' | 'click';

// 分页加载请求参数：由分页参数pageNum/pageSize和可选的自定义业务参数R组成
export type TRequestArgs<R> = {
  pageNum: number,
  pageSize: number,
} & R;

export type TRequestRes<T> = {
  list: T[], 
  total: number,
}

// 必须传入request方法的函数签名，接收分页加载请求参数，并返回TRequestRes结构体
export type TRequest<T, R> = (args: TRequestArgs<R>) => Promise<TRequestRes<T>>;

// 组件接收的props
export interface IProps<T, R> {
  // 触发load more时机: 'auto'滚动到底部自动加载; 'click'通过点击底部按钮的方式
  loadMoreTrigger?: TLoadMoreTrigger,
  // 如何渲染单条数据
  renderItem?: (item: T, index: number) => ReactNode,
  // 获取每页数据，一般是调用服务端API
  request: TRequest<T, R>,
  // 每一页加载数据条目数
  pageSize?: number,
  // 最外层的滚动容器类名
  className?: string,
  // 列表容器类名
  listContentClassName?: string,
  style?: React.CSSProperties,
}

// 通过ref对外暴露的方法或属性
export type TOpenState<R = undefined> = {
  reload: (customArgs?: R) => void,
}

// 参考https://stackoverflow.com/questions/58469229/react-with-typescript-generics-while-using-react-forwardref
export type IInfiniteLoadListWithForwardRef = <T, R = undefined>(
  props: (IProps<T, R> & { ref?: ForwardedRef<TOpenState<R>> })
) => ReactElement;

// 首页页码，由后端定义，可从0开始算起，也可以从1开始算起
const FIRST_PAGE_NUM = 1;

const InfiniteLoadList = forwardRef(<T, R = undefined>(
  props: IProps<T, R>, 
  ref: ForwardedRef<TOpenState<R>>,
) => {
  const {
    loadMoreTrigger = 'auto',
    renderItem,
    request,
    pageSize = 10,
    className,
    listContentClassName,
    style,
  } = props;

  // 分页加载相关
  const [loading, setLoading] = useState(false)
  const loadingRef = useRef(false)
  const setLoadingState = (bol: boolean) => {
    setLoading(bol)
    loadingRef.current = bol;
  }
  const [curPageNum, setCurPageNum] = useState(FIRST_PAGE_NUM)
  const [total, setTotal] = useState(0)
  const [list, setList] = useState<T[]>([])
  const notHasMore = list?.length >= total;

  const fetchList = async ({
    pageNum,
  }: { 
    pageNum: number,
  }) => {
    if (loadingRef.current) return;
    console.log('[fetchList executed]')
    try {
      setLoadingState(true)
      const params = {
        pageNum,
        pageSize,
        ...customArgs.current,
      } as TRequestArgs<R>;
      const res = await request(params);
      const pageList = res?.list || [];
      const newList = pageNum === FIRST_PAGE_NUM ? pageList : [...list, ...pageList];
      setList(newList);
      setCurPageNum(pageNum + 1);
      setTotal(res?.total || 0);
    } catch(err) {
      console.log('[request-error]:', err)
    } finally {
      setLoadingState(false)
    }
  }

  const customArgs = useRef<R>()
  useImperativeHandle(ref, () => {
    return {
      // 重新从第一页开始请求，可携带额外可选的自定义业务参数R
      reload: _customArgs => {
        // 每次重新从第一页开始请求时，是否重置当前页面状态
        // setList([])
        // setCurPageNum(FIRST_PAGE_NUM)
        // setTotal(0)
        if (_customArgs) customArgs.current = _customArgs;
        fetchList({
          pageNum: FIRST_PAGE_NUM,
        })
      }
    }
  }, [])

  const loadMore = (type: TLoadMoreTrigger) => {
    if (type === loadMoreTrigger) {
      if (notHasMore) return;
      // 后续加载更多
      fetchList({
        pageNum: curPageNum,
      });
    }
  }

  return (
    <ScrollView
      className={classnames('wrapper', className)}
      scrollY
      onScrollToLower={() => loadMore('auto')}
      style={style}
    >
      <View className='container'>
        <View className={classnames('list-content', listContentClassName)}> 
          {!!list?.length && list.map((item, index) => {
            const defaultItemEle = <View key={index}>{index}</View>
            const itemEle = typeof renderItem === 'function' ? 
              renderItem(item, index) : defaultItemEle;
            return itemEle;
          })}
        </View>
        {!loading && !list.length && (
          <View className='empty'>暂无数据</View>
        )}
        {!loading && list.length > 0 && !notHasMore && loadMoreTrigger === 'click' && (
          <View className='load-more' onClick={() => loadMore('click')}>
            点击加载更多
          </View>
        )}
        {!loading && list.length > 0 && notHasMore && (
          <View className='not-has-more'>--没有更多了--</View>
        )}
        {loading && (
          <View className='loading'>加载中...</View>
        )}
        {/* {loading && curPageNum === FIRST_PAGE_NUM && (
          <View className='loading'>首页加载中...</View>
        )}
        {loading && curPageNum > FIRST_PAGE_NUM && (
          <View className='loading'>分页加载中...</View>
        )} */}
      </View>
    </ScrollView>
  );
})

export default InfiniteLoadList as IInfiniteLoadListWithForwardRef;
```
2、使用InfiniteLoadList.tsx组件（不带额外业务参数的场景）
```
import {
  useState,
  useEffect,
  useRef,
} from 'react'
import { 
  View, 
} from '@tarojs/components'
import { 
  IProdItem, // 数据列表项的结构interface定义
  getProds, // 获取数据列表的后端API
} from '@/apis/prod'
// 引入InfiniteLoadList.tsx组件
import InfiniteLoadList, {
  TOpenState,
} from './component/InfiniteLoadList';

const Index = () => {
  const infiniteLoadListRef = useRef<TOpenState>(null)
  useEffect(() => {
    // 初始加载一次
    infiniteLoadListRef.current?.reload()
  }, [])
  const request = async ({
    pageNum,
    pageSize,
  }) => {
    // 组装后端需要的入参并调用后端API接口获取数据
    const params = {
      page_number: pageNum,
      page_size: pageSize,
    }
    const res = await getProds(params)
    // 按指定的结构体返回
    return {
      list: res?.data?.list || [],
      total: res?.data?.total || 0,
    }
  }
  return (
    <InfiniteLoadList<IProdItem>
      className='n-full-screen n-box-bd'
      request={request}
      ref={infiniteLoadListRef}
      renderItem={(item) => (
        <View style={{ height: '100px' }} key={item?.id}>
          {item?.id}
        </View>
      )}
    />
  );
}
export default Index;
```
3、使用InfiniteLoadList.tsx组件（带额外业务参数的场景）
```
import {
  useState,
  useEffect,
  useRef,
} from 'react'
import { 
  View, 
} from '@tarojs/components'
import Search from '@/components/Search'
import { 
  IProdItem,
  getProds,
} from '@/apis/prod'

// 引入InfiniteLoadList.tsx组件
import InfiniteLoadList, {
  TOpenState,
} from './component/InfiniteLoadList';

// 携带额外业务参数
type TCustomArgs = { searchWord: string }

const Index = () => {
  // 额外业务参数状态
  const [searchWord, setSearchWord] = useState('')

  const infiniteLoadListRef = useRef<TOpenState<TCustomArgs>>(null)
  useEffect(() => {
    // 每次searchWord发生变化时，重新从第一页开始加载
    infiniteLoadListRef.current?.reload({
      searchWord
    })
  }, [searchWord])

  const request = async ({
    pageNum,
    pageSize,
    searchWord,
  }) => {
    const params = {
      page_number: pageNum,
      page_size: pageSize,
      key_word: searchWord ? searchWord?.trim() : undefined,
    }
    const res = await getProds(params)
    return {
      list: res?.data?.list || [],
      total: res?.data?.total || 0,
    }
  }
  return (
    <View className='pagewrap_product-search'>
      <Search
        className='search-prods n-fix n-bg1 n-z1'
        defaultValue={keyword || ''}
        placeholder='请输入产品名称/产品编号'
        onSearch={setSearchWord}
      />
      <InfiniteLoadList<IProdItem, TCustomArgs>
        className='n-full-screen n-box-bd'
        style={{
          // 使用padding避免滚动区域被上方的Search悬浮组件遮盖
          padding: '148px 30px 30px',
        }}
        request={request}
        ref={infiniteLoadListRef}
        renderItem={(item) => (
          <View style={{ height: '100px' }} key={item?.id}>
            {item?.id}
          </View>
        )}
      />
    </View>
  );
}
export default Index;
```

