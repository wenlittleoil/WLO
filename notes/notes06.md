## 29.  
css原子化  
```
// atomization.css
// 作用是降低css代码体积
// 使用n-开头与普通样式类名区分，避免冲突

.n-fw {
  font-weight: 500;
}

.n-c1 {
  color: #000;
}

.n-c2 {
  color: #777;
}

.n-c3 {
  color: #fff;
}

.n-fs24 {
  font-size: 24px;
}

.n-fs26 {
  font-size: 26px;
}

.n-fs28 {
  font-size: 28px;
}

.n-fs30 {
  font-size: 30px;
}

.n-fs32 {
  font-size: 32px;
}

.n-fs34 {
  font-size: 34px;
}

.n-flex {
  display: flex;
  flex-direction: row;
}

.n-flex-col {
  display: flex;
  flex-direction: column;
}

.n-justify-sb {
  justify-content: space-between;
}

.n-justify-ct {
  justify-content: center;
}

.n-align-ct {
  align-items: center;
}

.n-fx0 {
  flex-shrink: 0;
  flex-grow: 0;
}

.n-fx1 {
  flex-shrink: 1;
  flex-grow: 1;
}

.n-rel {
  position: relative;
}

.n-abs {
  position: absolute;
}

.n-fix {
  position: fixed;
}

.n-z1 {
  z-index: 1;
}

.n-z2 {
  z-index: 2;
}

.n-z9 {
  z-index: 9;
}

.n-z99 {
  z-index: 99;
}

.n-z100 {
  z-index: 100;
}

.n-full {
  width: 100%;
  height: 100%;
}

.n-full-screen {
  width: 100vw;
  height: 100vh;
}

.n-box-bd {
  box-sizing: border-box;
}

.n-nowrap {
  white-space: nowrap;
}

.n-text-ct {
  text-align: center;
}

.n-w100 {
  width: 100%;
}

.n-w50 {
  width: 50%;
}

.n-bg1 {
  background-color: #fff;
}

.n-bg2 {
  background-color: transparent;
}

.n-bg3 {
  background-color: #000;
}

.n-nowrap {
  white-space: nowrap;
}
```
