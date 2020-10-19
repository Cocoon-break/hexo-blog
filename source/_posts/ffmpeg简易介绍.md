---
title: ffmpeg简易介绍
date: 2017-05-22 14:34:12
tags: 
- 扩展
categories:
- 开源工具
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

   #### ffmpeg常用的日常命令

   **关于FFmpeg的具体技术及参数细节，可以参考**[**ffmpeg官方文档**](https://ffmpeg.org/documentation.html)**，以下介绍一些常用的ffmpeg命令。**
   
   1. H264编码的其他格式转换TS
   
      ```shell
      ffmpeg -i input.(mp4、avi、264) -vcodec copy output.ts
      ffmpeg -i input.(mp4、aiv、264) -vcodec libx264 -an out.ts
      ```
   
      说明： 一般转换格式用于离线视频识别和live555转发，多数因使用ts格式文件，故需要使用ffmpeg进行转换。
   
      -vcodec： vcodec 参数使用copy，表示复制原编码，而不进行重编码，效率比较高，默认都是h264编码视频。
   
   2. 视频转帧图片
   
      ```shell
      ffmpeg -i input.(mp4、avi、264) -r 25 -f image2 ./path/%05d.png
      ```
   
      将视频中每一帧切出来，可用作测试人脸检测模块。
   
      -r：表示帧率，25即每秒25帧来处理帧。
   
      -f：表示输出格式，此处为image2格式。
   
       ./path/%05d.png：以png格式输出，path为路径，%05d为通配符，表示以5位数字命名。
   
      批量转图：
   
      ```shell
      find ./ -name "*.mp4" | cut -c 3- | xargs -i ffmpeg -i ./{} -r 25 -f image2 ./path/%05d{}.png
      ```
   
   3. 帧图片转视频
   
      ```shell
      ffmpeg -i ./path/%05d.png -r 25 -vcodec h264 output.ts
      ```
   
      将连续的帧图片，转制为视频。
   
   4. 视频帧率处理
   
      ```shell
      ffmpeg -i input.ts -vcode h264 -r 20 output.ts
      ```
   
      模拟不同帧率下的视频。根据参数-r，决定处理帧率的数值。
   
   5. 从相机视频流录制ts或mp4格式的视频
   
      ```shell
      ffmpeg -t 3600 -rtsp_transport tcp -i rtsp://10.201.105.51/live1.sdp -vcodec copy test06223600.ts
      ```
   
      说明：tcp协议
   
   6. 截取视频指令
   
      ```shell
      ffmpeg -ss START -vsync 0 -t DURATION -i INPUT -vcodec VIDEOCODEC-acodec AUDIOCODEC OUTPUT
      ```
   
      说明：
   
      ​     -ss 视频开始时刻，格式为00:00:00，比如，从第5秒开始录制，则为00:00:05
   
      ​     -t 视频持续时长，格式为00:00:00，比如，要录制10秒钟的视频，则为00:00:10
   
   7. 合并视频指令
   
      ```shell
      ffmpeg -i "concat:intermediate1.ts|intermediate2.ts" -c copy -bsf:a aac_adtstoasc output.ts
      ```
   
   8. 视频旋转
   
       ```shell
      find ./ -name "*.mp4" | cut -c 3- | xargs -i ffmpeg -i ./{} -vf "transpose=1" -vcodec libx264 -an ./rota/{}
       ```
   
   9. 图片选择
   
      ```shell
      convert example.jpg -rotate 90 example-rotated.jpg
      ```
   
   10. 获取视频文件的帧数
   
       ```shell
       ffmpeg -i 1234.mp4 -vcodec copy -acodec copy -f null /dev/null 2>&1 | grep 'frame='
       ```
   
   [文章参考](https://github.com/FiveYellowMice/how-to-convert-videos-with-ffmpeg-zh) 
   
   


