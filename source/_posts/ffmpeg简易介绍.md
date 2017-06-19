---
title: ffmpeg简易介绍
date: 2017-05-22 14:34:12
tags: 扩展

---

## 媒体文件结构

一个媒体文件并不像许多人想象的那样，是将媒体内容编码起来直接作为文件的。实际上，它通常是由多个不同种类的媒体流（ Stream ）组成，再以特定的封装格式封装起来的。

比较常见的媒体流就是视频流跟音频流了，顾名思义，视频流存储的就是视频信息，音频流存储的就是音频信息。一个视频流或者音频流的内容，就是以特定的编码格式所存储的视频或音频信息。

**一个文件的里面的媒体流所采用的编码格式跟这个文件的后缀名并没有完全的必然联系。** **文件的后缀名通常就代表这个文件的封装格式。**

## ffmpeg 安装

自行下载使用系统安装对应的安装包[官网下载](https://ffmpeg.org/download.html)

以下使用的平台都为Mac系统下

<!-- more -->

## 开始使用

1. 打开终端，查看ffmpeg帮助文档，简要了解ffmpeg的使用

   ```ssh
   ffmpeg -help
   ```

   ```ssh
   ffmpeg version 3.3 Copyright (c) 2000-2017 the FFmpeg developers
     built with Apple LLVM version 8.1.0 (clang-802.0.41)
     configuration: --prefix=/usr/local/Cellar/ffmpeg/3.3 --enable-shared --enable-pthreads --enable-gpl --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --disable-lzma --enable-vda
     libavutil      55. 58.100 / 55. 58.100
     libavcodec     57. 89.100 / 57. 89.100
     libavformat    57. 71.100 / 57. 71.100
     libavdevice    57.  6.100 / 57.  6.100
     libavfilter     6. 82.100 /  6. 82.100
     libavresample   3.  5.  0 /  3.  5.  0
     libswscale      4.  6.100 /  4.  6.100
     libswresample   2.  7.100 /  2.  7.100
     libpostproc    54.  5.100 / 54.  5.100
   Hyper fast Audio and Video encoder
   usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...
   ```

   重点是关注 usage: ffmpeg [options][[infile options] -i infile]... {[outfile options] outfile}… 这句是说ffmpeg 的主要用法

   ​

2. 查看ffmpeg 支持的格式

   查看封装格式，包括音频，视频等封装格式

   ```ssh
   ffmpeg -formats
   ```

   查看编解码器包括音频，视频等封装格式

   ```ssh
   ffmpeg -codecs
   ```

3. 进行格式转换

   ```ssh
   ffmpeg -i a.mp4 b.mkv
   ```

   这里是将mp4格式的视频转换成mkv格式。

   **注：** 查看默认编码格式，以下的Matroska也就是指 mkv

   ```code
   ffmpeg -help muxer=Matroska

   #以下为摘要信息，从中可以看出默认的视频编码为h264,音频编码为ac3,字幕流编码为ass，部分格式视频不支持字幕流
   Muxer matroska [Matroska]:
       Common extensions: mkv.
       Mime type: video/x-matroska.
       Default video codec: h264.
       Default audio codec: ac3.
       Default subtitle codec: ass.
   ```

4. 指定编码器进行转换

   ```sh
   ffmpeg -i a.mp4 -c:v hevc -c:a aac b.mkv
   ```

   **注：** `-c:v` 可以用`-vcodec`替换，当它们的值为copy时，就表示编码格式不进行转换

5. 在转换时可以进行的调整

   在 `ffmpeg -help` 时可以看到有一些和音频，视频，字幕相关的选项

   ```ssh
   Video options:
   -vframes number     set the number of video frames to output
   -r rate             set frame rate (Hz value, fraction or abbreviation)
   -s size             set frame size (WxH or abbreviation)
   -aspect aspect      set aspect ratio (4:3, 16:9 or 1.3333, 1.7777)
   -bits_per_raw_sample number  set the number of bits per raw sample
   -vn                 disable video
   -vcodec codec       force video codec ('copy' to copy stream)
   -timecode hh:mm:ss[:;.]ff  set initial TimeCode value.
   -pass n             select the pass number (1 to 3)
   -vf filter_graph    set video filters
   -ab bitrate         audio bitrate (please use -b:a)
   -b bitrate          video bitrate (please use -b:v)
   -dn                 disable data

   Audio options:
   -aframes number     set the number of audio frames to output
   -aq quality         set audio quality (codec-specific)
   -ar rate            set audio sampling rate (in Hz)
   -ac channels        set number of audio channels
   -an                 disable audio
   -acodec codec       force audio codec ('copy' to copy stream)
   -vol volume         change audio volume (256=normal)
   -af filter_graph    set audio filters

   Subtitle options:
   -s size             set frame size (WxH or abbreviation)
   -sn                 disable subtitle
   -scodec codec       force subtitle codec ('copy' to copy stream)
   -stag fourcc/tag    force subtitle tag/fourcc
   -fix_sub_duration   fix subtitles duration
   -canvas_size size   set canvas size (WxH or abbreviation)
   -spre preset        set the subtitle options to the indicated preset
   ```

   当然还有对整个文件进行调整的参数

   ```ssh
   Per-file main options:
   -f fmt              force format
   -c codec            codec name
   -codec codec        codec name
   -pre preset         preset name
   -map_metadata outfile[,metadata]:infile[,metadata]  set metadata information of outfile from infile
   -t duration         record or transcode "duration" seconds of audio/video
   -to time_stop       record or transcode stop time
   -fs limit_size      set the limit file size in bytes
   -ss time_off        set the start time offset
   -sseof time_off     set the start time offset relative to EOF
   -seek_timestamp     enable/disable seeking by timestamp with -ss
   -timestamp time     set the recording timestamp ('now' to set the current time)
   -metadata string=string  add metadata
   -program title=string:st=number...  add program with specified streams
   -target type        specify target file type ("vcd", "svcd", "dvd", "dv" or "dv50" with optional prefixes "pal-", "ntsc-" or "film-")
   -apad               audio pad
   -frames number      set the number of frames to output
   -filter filter_graph  set stream filtergraph
   -filter_script filename  read stream filtergraph description from a file
   -reinit_filter      reinit filtergraph on input parameter changes
   -discard            discard
   -disposition        disposition
   ```

   [文章参考](https://github.com/FiveYellowMice/how-to-convert-videos-with-ffmpeg-zh) 

   ​


