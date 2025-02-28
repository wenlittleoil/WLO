## 51.
使用命令行工具ffmpeg处理源视频文件video.mp4，从而生成hls协议的m3u8索引和ts片段文件
```
ffmpeg -i ./video.mp4 -codec: copy -start_number 0 -hls_time 10 -hls_list_size 0 -f hls output.m3u8
```


