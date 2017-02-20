---
layout: "post"
title: "youtube-dl"
date: "2017-02-20 23:48"
tags:
- youtube
comments: true
---


# 下载YouTube视频

- 查看视频所有类型,只看不下载
    youtube-dl -F [url]

或者
    youtube-dl --list-formats [url]

这是一个列清单参数，执行后并不会下载视频，但能知道这个目标视频都有哪些格式存在，这样就可以有选择的下载啦！

![](http://upload-images.jianshu.io/upload_images/1294473-5c8620972c0957c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看YouTube视频所有类型

- 下载指定质量的视频和音频并自动合并
    youtube-dl -f [format code] [url]

通过上一步获取到了所有视频格式的清单，最左边一列就是编号对应着不同的格式.
由于YouTube的1080p及以上的分辨率都是音视频分离的,所以我们需要分别下载视频和音频,可以使用137+140这样的组合.
如果系统中安装了ffmpeg的话, youtube-dl 会自动合并下下好的视频和音频, 然后自动删除单独的音视频文件

![](http://upload-images.jianshu.io/upload_images/1294473-5fabdc8817bb3135.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载1080p的视频

-
下载字幕

    youtubd-dl --write-sub [url] //这样会下载一个vtt格式的英文字幕和mkv格式的1080p视频下来

    youtube-dl --write-sub --skip-download [url] //下载单独的vtt字幕文件,而不会下载视频

    youtube-dl --write-sub --all-subs [url] //下载所有语言的字幕(如果有的话)

    youtube-dl --write-auto-sub [url] //下载自动生成的字幕(YouTube only)

![](http://upload-images.jianshu.io/upload_images/1294473-191ff153e45eea04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下载字幕和视频

-
下载视频列表

    youtube-dl -f [format code] [palylist_url] //这种方式可以下载制定清晰度的mp4视频

    youtube-dl [playlist_url] //下载视频列表,这种方式下载的视频可能是mkv格式或者webm格式

    youtube-dl -cit [playlist_url] //下载视频列表,这种方式下载的视频可能是mkv格式或者webm格式

    youtube-dl --yes-playlist [url] //当链接为视频列表,则下载该列表视频,跟上面的一样,可能是mkv或者webm格式

---

# 下载Vimeo视频

Vimeo的视频下载起来比较方便,因为没有分离,可以直接下载1080p带音频的视频
命令与下载YouTube的基本一致;下面贴几张图

![](http://upload-images.jianshu.io/upload_images/1294473-4e1b9f4b18e94700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解查看Vimeo视频所有类型

![](http://upload-images.jianshu.io/upload_images/1294473-5720dba2d2ab972e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直接下载Vimeo最高质量视频

---

youtube-dl支持的网站很多,大家可以从作者整理的[这个列表](https://rg3.github.io/youtube-dl/supportedsites.html)里查看支持的网站(不过由于有的网站接口改变,可能当初支持的网站现在不能很好的支持了),如果您要下载的视频网站现在不能用youtube-dl下载的,不妨试试另外一个同样基于Python开发的下载工具You-Get,我将在下一篇文章中介绍,希望大家喜欢~

> youtube-dl官网：[https://yt-dl.org/](https://yt-dl.org/)
> GitHub项目：[https://github.com/rg3/youtube-dl/](https://github.com/rg3/youtube-dl/)
