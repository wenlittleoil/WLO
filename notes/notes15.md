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
第四步：在macOS命令行执行`adb devices`查看当前电脑总共连接了几台adb设备，输出结果如下
```
List of devices attached
10AE1K1SYA00148	device
emulator-5554	device
```
例如上述输出结果中的`10AE1K1SYA00148`即为您的android真机，然后命令行执行`adb -s 10AE1K1SYA00148 reverse tcp:8081 tcp:8081`将您的android真机端口代理到macOS电脑，从而可以访问本地dev server进行开发调试。
第五步：重新启动您android真机上已安装的apk应用。



