
README
=============

## 简介
ffmpeg项目，主要适配avfilter中的scale_npp支持RGB24、BGR24格式，以减少CPU使用率。

* 20240125，下载ffmpeg官方源码:
```
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg/
```
* 当前最新log:  
```  
commit e0da916b8f5b079a4865eef7f64863f50785463d (HEAD -> master, origin/master, origin/HEAD)
Author: Anton Khirnov <anton@khirnov.net>
Date:   Wed Jan 24 20:21:37 2024 +0100

    fftools/ffmpeg: optimize inter-thread queue sizes
    
    Use 8 packets/frames by default rather than 1, which seems to provide
    better throughput.
    
    Allow -thread_queue_size to set the muxer queue size manually again.
```

## 编译方法
参考[Using_FFmpeg_with_NVIDIA_GPU_Hardware_Acceleration](doc/Using_FFmpeg_with_NVIDIA_GPU_Hardware_Acceleration.pdf)文档中的`2.2.2.1. Compiling for Linux`章节。

FFmpeg with NVIDIA GPU acceleration is supported on all Linux platforms.
To compile FFmpeg on Linux, do the following:

### 1. Clone ffnvcodec
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git

### 2. Install ffnvcodec
cd nv-codec-headers && sudo make install && cd –
### 3. Clone FFmpeg's public GIT repository.
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg/
### 4. Install necessary packages.
sudo apt-get install build-essential yasm cmake libtool libc6 libc6-dev unzip wget
libnuma1 libnuma-dev
### 5. Configure
./configure --enable-nonfree --enable-cuda-nvcc --enable-libnpp --extra-cflags=-I/usr/
local/cuda/include --extra-ldflags=-L/usr/local/cuda/lib64 --disable-static --enableshared
### 6. Compile
make -j 8
### 7. Install the libraries.
sudo make install

## 测试命令
```
# 保存文件
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -rtsp_transport tcp -i "rtsp://192.168.1.119/live/test" -vf "scale_npp=format=bgr24,hwdownload,format=bgr24" -fps_mode vfr out.rgb -y
# 不保存文件
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -rtsp_transport tcp -i "rtsp://192.168.1.119/live/test" -vf "scale_npp=format=bgr24,hwdownload,format=bgr24" -fps_mode vfr -f rawvideo pipe: > /dev/null
```

## 测试结论
使用1路1080p、25fps的视频，在`NVIDIA GeForce GTX 1080 Ti`上测试：

```
# nv12直接硬转yuv420p，软转rgb24
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -c:v hevc_cuvid -i "rtsp://192.168.1.119/live/test" -vf "scale_cuda=format=yuv420p,hwdownload,format=yuv420p,format=rgb24" -f rawvideo pipe: > /dev/null
212m, fps= 25, cpu 30

# nv12直接硬转yuv420p，不转rgb24
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -c:v hevc_cuvid -i "rtsp://192.168.1.119/live/test" -vf "scale_cuda=format=yuv420p,hwdownload,format=yuv420p" -f rawvideo pipe: > /dev/null
212m, fps= 25, cpu 12

# 软件转码，默认yuv420p
ffmpeg -i "rtsp://192.168.1.119/live/test" -f rawvideo pipe: > /dev/null
0m, fps= 25, cpu 100

# 软件转码, 再转rgb24
ffmpeg -i "rtsp://192.168.1.119/live/test" -vf "format=rgb24" -f rawvideo pipe: > /dev/null
ffmpeg -i "rtsp://192.168.1.119/live/test" -f rawvideo -pix_fmt rgb24 pipe: > /dev/null
0m, fps= 25, cpu 112

# nv12直接硬转yuv420p，软转bgr24
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -c:v hevc_cuvid -i "rtsp://192.168.1.119/live/test" -vf "scale_npp=format=yuv420p,hwdownload,format=yuv420p,format=bgr24" -f rawvideo pipe: > /dev/null
214m, fps= 25, cpu 28~31

# nv12直接硬转bgr24
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -c:v hevc_cuvid -i "rtsp://192.168.1.119/live/test" -vf "scale_npp=format=bgr24,hwdownload,format=bgr24" -f rawvideo pipe: > /dev/null
222m, fps= 25, cpu 16~20

# nv12直接输出，2中方式
ffmpeg -hwaccel cuda  -c:v hevc_cuvid -i "rtsp://192.168.1.119/live/test"  -f rawvideo pipe: > /dev/null
196m, fps= 25, cpu 11~13
ffmpeg -hwaccel cuda -hwaccel_output_format cuda -c:v hevc_cuvid -i "rtsp://192.168.1.119/live/test" -vf "hwdownload,format=nv12" -f rawvideo pipe: > /dev/null
204m, fps= 25, cpu 11~13

```
> 结论：
    1. 软解码缩放 -> 硬解码缩放（软转rgb24），cpu使用率从112%下降到30%；
    2. nv12硬转yuv420p软转bgr24，cpu 28~31%；
    3. nv12硬转bgr24, cpu 16~20%，cpu使用率从降约10%；
    4. nv12直接输出，cpu 11~13%。

FFmpeg README
=============

FFmpeg is a collection of libraries and tools to process multimedia content
such as audio, video, subtitles and related metadata.

## Libraries

* `libavcodec` provides implementation of a wider range of codecs.
* `libavformat` implements streaming protocols, container formats and basic I/O access.
* `libavutil` includes hashers, decompressors and miscellaneous utility functions.
* `libavfilter` provides means to alter decoded audio and video through a directed graph of connected filters.
* `libavdevice` provides an abstraction to access capture and playback devices.
* `libswresample` implements audio mixing and resampling routines.
* `libswscale` implements color conversion and scaling routines.

## Tools

* [ffmpeg](https://ffmpeg.org/ffmpeg.html) is a command line toolbox to
  manipulate, convert and stream multimedia content.
* [ffplay](https://ffmpeg.org/ffplay.html) is a minimalistic multimedia player.
* [ffprobe](https://ffmpeg.org/ffprobe.html) is a simple analysis tool to inspect
  multimedia content.
* Additional small tools such as `aviocat`, `ismindex` and `qt-faststart`.

## Documentation

The offline documentation is available in the **doc/** directory.

The online documentation is available in the main [website](https://ffmpeg.org)
and in the [wiki](https://trac.ffmpeg.org).

### Examples

Coding examples are available in the **doc/examples** directory.

## License

FFmpeg codebase is mainly LGPL-licensed with optional components licensed under
GPL. Please refer to the LICENSE file for detailed information.

## Contributing

Patches should be submitted to the ffmpeg-devel mailing list using
`git format-patch` or `git send-email`. Github pull requests should be
avoided because they are not part of our review process and will be ignored.
