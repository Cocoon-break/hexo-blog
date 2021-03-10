---
title: ffmpeg
date: 2017-05-22 14:34:12
index_img:
- /images/shell/003_ffmpeg_1.jpg
tags: 
- shell
categories:
- 开源工具
---

### 音视频的基础概念

在开始使用ffmpeg 之前，我们需要对音视频一些基本概念有一定的了解。

1. 编码和格式

   很多人会把这两个概念混淆在一起，因为编码和格式有不少相同的命名，但他们是两个概念。

   - **编码codec :**是对图像和声音进行压缩的方法。与之对应的是**解码**。常见的编码有H264，H265，MPEG等
   - **格式（容器）**：是将不同编码后的信息打包在一起。常用的格式有mp4，avi，mkv等

2. 流

   一般来说，视频文件很大，很难全部一次性读进内存，只能一点一点的读。操作系统对文件的操作就是基于**流**的抽象。

3. 帧

   我们知道传统的动画是有一张一张图画持续滚动的。基于这个原理我们能够理解帧其实就是这一张一张的图画。当然这只是一个基本的概念，实际上的ffmpeg上的帧有更多的含义。

### ffmpeg是什么 

ffmpeg 是一个命令行工具同时也是一个开源库，开发者可以基于现有的库进行二次开发。

Ffmpeg 安装可以自行下载使用系统安装对应的安装包[官网下载](https://ffmpeg.org/download.html)

### 命令行使用

**关于FFmpeg的具体技术及参数细节，可以参考**[**ffmpeg官方文档**](https://ffmpeg.org/documentation.html)**，以下介绍一些常用的ffmpeg命令。**

1. 打开终端，查看ffmpeg帮助文档，简要了解ffmpeg的使用

   ```shell
   # 查看ffmpeg 帮助信息
   ffmpeg -help
   # 查看ffmpeg 支持的格式
   ffmpeg -formats
   # 查看ffmpeg支持的编码格式
   ffmpeg -codecs
   ```

    usage: ffmpeg [options][[infile options] -i infile]... {[outfile options] outfile}… 这句是说ffmpeg 的主要用法

2. H264编码的其他格式转换TS

   ```shell
   ffmpeg -i input.(mp4、avi、264) -vcodec copy output.ts
   ffmpeg -i input.(mp4、aiv、264) -vcodec libx264 -an out.ts
   ```

   说明： 一般转换格式用于离线视频识别和live555转发，多数因使用ts格式文件，故需要使用ffmpeg进行转换。

   -vcodec： vcodec 参数使用copy，表示复制原编码，而不进行重编码，效率比较高，默认都是h264编码视频。

3. 视频转帧图片

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

   视频帧转YUV数据

   ```shell
   ffmpeg -i a.mp4 -f segment -segment_time 0.01 -r 30 -pix_fmt nv21 ./path
   ```

4. 帧图片转视频

      ```shell
      ffmpeg -i ./path/%05d.png -r 25 -vcodec h264 output.ts
      ```

      将连续的帧图片，转制为视频。

5. 视频帧率处理

      ```shell
      ffmpeg -i input.ts -vcode h264 -r 20 output.ts
      ```

      模拟不同帧率下的视频。根据参数-r，决定处理帧率的数值。

6. 从相机视频流录制ts或mp4格式的视频

      ```shell
      ffmpeg -t 3600 -rtsp_transport tcp -i rtsp://10.201.105.51/live1.sdp -vcodec copy test06223600.ts
      ```

      说明：tcp协议

7. 截取视频指令

      ```shell
      ffmpeg -ss START -vsync 0 -t DURATION -i INPUT -vcodec VIDEOCODEC-acodec AUDIOCODEC OUTPUT
      ```

      说明：

      ​     -ss 视频开始时刻，格式为00:00:00，比如，从第5秒开始录制，则为00:00:05

      ​     -t 视频持续时长，格式为00:00:00，比如，要录制10秒钟的视频，则为00:00:10

8. 合并视频指令

      ```shell
      ffmpeg -i "concat:intermediate1.ts|intermediate2.ts" -c copy -bsf:a aac_adtstoasc output.ts
      ```

9. 视频旋转

   ```shell
   find ./ -name "*.mp4" | cut -c 3- | xargs -i ffmpeg -i ./{} -vf "transpose=1" -vcodec libx264 -an ./rota/{}
   ```

10. 图片旋转

      ```shell
      convert example.jpg -rotate 90 example-rotated.jpg
      ```

11. 图片等比缩小

       ```shell
       convert k.jpg -resize 50% k2.jpg
       ```
12. 获取视频文件的帧数
       ```shell
          ffmpeg -i 1234.mp4 -vcodec copy -acodec copy -f null /dev/null 2>&1 | grep 'frame='
       ```

   [文章参考](https://github.com/FiveYellowMice/how-to-convert-videos-with-ffmpeg-zh) 

### 搭建rtsp视频流服务

利用live555工具提供rtsp服务，需要从官网下载[live555源码](http://www.live555.com/liveMedia/public/live555-latest.tar.gz)进行编译成可执行文件。

```shell
# 将源码解压
tar xzf live555-latest.tar.gz
cd live
# 生成编译Linux 64位执行程序的Makefile。编译macos ./genMakefiles macosx
./genMakefiles linux-64bit 
# 编译
make
```

编译完成后会在当前目录下生成`live555MediaServer`可执行文件。该文件可以放到任何一个Linux64位系统下使用。通过执行`live555MediaServer`可提供服务。

举个例子
利用ffmpeg 将一个mp4 文件转成ts格式文件

```shell
ffmpeg -i a.mp4 -vcodec copy a.ts
```

将a.ts 和 `live555MediaServer`放在同一个文件夹下就可对外提供rtsp服务。具体地址**rtsp://ip/a.st**


