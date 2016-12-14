title: 常用软件备忘
date: 2015-11-02 13:03:24
toc: true
tags: [vlc]
categories: tools
keywords: [app, 软件, vlc]
description: VLC 作为服务器记录。
---

1. “媒体” -> “流”，弹出“打开媒体窗口”。
2. 选择“文件”标签，“Add”一个视频源，再选择“串流”按钮，进入“流输出”窗口。
3. 在“来源”界面，直接点下一步。
4. 在“Destination setup”的“New destination”点击下拉菜单选择“RTSP”协议，再点击旁边的“添加”按钮。
5. 在“RTSP”标签下改端口号（默认8554）和路径，以路径“test”为例，点击下一步。
6. 在“Transcoding Options”中，选择一种，我选的是“Video - H.264 + MP3(MP4)”，点击右侧紧靠的按钮，选择封装，我的视频是MKV，我直接就点MKV了，点击保存。
7. 点击下一步，点击“STREAM”按钮即可。

访问地址为：`rtsp://your_ip:8554/test`
