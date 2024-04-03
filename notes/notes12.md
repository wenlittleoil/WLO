## 39.
微信小程序封装页面公共header。
1. 组件的封装：
```
// index.tsx
import { 
  FC, 
  ReactNode, 
  memo, 
  useEffect, 
  useMemo, 
  useState,
} from "react";
import Taro from "@tarojs/taro";
import { 
  View, 
} from "@tarojs/components";
import cx from "classnames";
import styles from "./index.module.scss";

interface ICommonNavBarProps {
  children?: ReactNode,
  mode?: 'default' | 'fixed', // default: 占位(不脱离文档流); fixed: 不占位(脱离文档流)
  background?: string, // 顶部栏背景色
  spaceBelowMenuBtn?: number, // 菜单胶囊按钮下方预留空间，单位px
}

const CommonNavBar:FC<ICommonNavBarProps> = props => {
  const {
    children = null,
    mode = 'default',
    background = 'transparent',
    spaceBelowMenuBtn,
  } = props;

  // 状态栏高度（电池信息栏）
  const [statusBarHeight, setStatusBarHeight] = useState(0)
  // 菜单胶囊按钮和状态栏之间的间隙
  const [spaceBetweenMenuBtnAndStatusBar, setSpaceBetweenMenuBtnAndStatusBar] = useState(0)
  // 菜单胶囊按钮高度
  const [menuBtnHeight, setMenuBtnHeight] = useState(0)
  
  // 核心区域两侧padding，即菜单胶囊按钮右边和屏幕右边的距离
  const [bothSideSpace, setBothSideSpace] = useState(0)

  useEffect(() => {
    const systemInfo = Taro.getSystemInfoSync(); // 获取系统信息
    const menuBtnInfo = Taro.getMenuButtonBoundingClientRect(); // 菜单胶囊按钮位置信息
    setStatusBarHeight(systemInfo.statusBarHeight as number);
    setSpaceBetweenMenuBtnAndStatusBar(menuBtnInfo.top - (systemInfo.statusBarHeight as number));
    setMenuBtnHeight(menuBtnInfo.height);
    setBothSideSpace(systemInfo.screenWidth - menuBtnInfo.right);
    console.log('【systemInfo, menuBtnInfo】', systemInfo, menuBtnInfo);
  }, [])

  // 菜单胶囊按钮下方预留空间，默认是spaceBetweenMenuBtnAndStatusBar
  const realSpaceBelowMenuBtn = useMemo(() => {
    if (typeof spaceBelowMenuBtn === 'number') return spaceBelowMenuBtn;
    return spaceBetweenMenuBtnAndStatusBar;
  }, [spaceBelowMenuBtn, spaceBetweenMenuBtnAndStatusBar])

  // 整个顶部栏的总高
  const totalHeight = useMemo(() => {
    return statusBarHeight + spaceBetweenMenuBtnAndStatusBar + 
      menuBtnHeight + realSpaceBelowMenuBtn;
  }, [statusBarHeight, spaceBetweenMenuBtnAndStatusBar, menuBtnHeight, realSpaceBelowMenuBtn])

  return (
    <View 
      className={cx(styles['CommonNavBar'], styles[`CommonNavBar__${mode}`])}
      style={{
        background,
        height: `${totalHeight}px`,
      }}
    >
      <View 
        className={styles["status-bar"]} 
        style={{
          height: `${statusBarHeight}px`,
        }}
      >
      </View>
      <View 
        className={styles["cont"]}
        style={{
          height: `${menuBtnHeight}px`,
          paddingTop: `${spaceBetweenMenuBtnAndStatusBar}px`,
          paddingBottom: `${realSpaceBelowMenuBtn}px`,
        }}
      >
        <View 
          className={styles["core"]}
          style={{
            padding: `0 ${bothSideSpace}px`,
          }}
        >
          <View className={styles["slot"]}>
            {children}
          </View>
        </View>
      </View>
    </View>
  )
}

export default memo(CommonNavBar);
```
```
// index.module.scss
.CommonNavBar {
  width: 100%;
  &__default {
    position: static;
  }
  &__fixed {
    position: fixed;
    z-index: 9;
    top: 0;
    left: 0;
  }
  .status-bar {}
  .cont {
    box-sizing: content-box;
    .core {
      width: 100%;
      height: 100%;
      box-sizing: border-box;
      .slot {
        width: 100%;
        height: 100%;
      }
    }
  }
}
```
2. 组件的使用：
```
<CommonNavBar mode="fixed">
  <View className="n-full n-flex"> 
    <Image 
      src="icon-back.png"
      style={{
        width: '50rpx',
        height: '50rpx',
      }}
      onClick={() => {
        Taro.navigateBack()
      }}
    />
  </View>
</CommonNavBar>
```
