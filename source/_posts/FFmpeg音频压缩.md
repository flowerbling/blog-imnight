---
title: FFmpeg音频压缩
date: 2023-12-08 10:59:56
tags: FFmpeg
categories:
    FFmpeg
---

### FFmpeg压缩音频
> 有的时候对音频质量要求不高的时候 可以降低音频的比特率来减小音频文件的大小

```bash
ffmpeg -i input.mp3 -b:a 128k output.mp3
```

```
这个命令将把名为 input.mp3 的音频文件压缩为比特率为128kbps的MP3格式，并将输出保存为 output.mp3 文件。

在这个命令中，-i 参数用于指定输入文件的路径和名称，-b:a 参数用于设置音频的目标比特率，128k 表示128kbps。你可以根据需要调整比特率的值来控制音频的压缩质量。较低的比特率会导致较小的文件大小，但可能会降低音频质量。
```

#### 压缩前先看看这个音频的比特率原先多大
```bash
ffmpeg -i ~/Desktop/7aKdmDC9koXfxT9nueM8iK.mp3
ffmpeg version 5.1.1-tessus Copyright (c) 2000-2022 the FFmpeg developers
  built with Apple clang version 11.0.0 (clang-1100.0.33.17)
  configuration: --cc=/usr/bin/clang --prefix=/opt/ffmpeg --extra-version=tessus --enable-avisynth --enable-fontconfig --enable-gpl --enable-libaom --enable-libass --enable-libbluray --enable-libdav1d --enable-libfreetype --enable-libgsm --enable-libmodplug --enable-libmp3lame --enable-libmysofa --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenh264 --enable-libopenjpeg --enable-libopus --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvmaf --enable-libvo-amrwbenc --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxavs --enable-libxvid --enable-libzimg --enable-libzmq --enable-libzvbi --enable-version3 --pkg-config-flags=--static --disable-ffplay
  libavutil      57. 28.100 / 57. 28.100
  libavcodec     59. 37.100 / 59. 37.100
  libavformat    59. 27.100 / 59. 27.100
  libavdevice    59.  7.100 / 59.  7.100
  libavfilter     8. 44.100 /  8. 44.100
  libswscale      6.  7.100 /  6.  7.100
  libswresample   4.  7.100 /  4.  7.100
  libpostproc    56.  6.100 / 56.  6.100
[mp3 @ 0x7feea6104c80] Estimating duration from bitrate, this may be inaccurate
Input #0, mp3, from '/Users/bzy/Desktop/7aKdmDC9koXfxT9nueM8iK.mp3':
  Duration: 00:00:43.90, start: 0.000000, bitrate: 160 kb/s
  Stream #0:0: Audio: mp3, 24000 Hz, mono, fltp, 160 kb/s
At least one output file must be specified
```

可以看到是160kb/s的比特率

这里压缩到64就足够了 前端在播放的时候 对音频没有什么要求

#### 开始转码

```bash
ffmpeg -i ~/Desktop/7aKdmDC9koXfxT9nueM8iK.mp3 -b:a 64k ~/Desktop/7aKdmDC9koXfxT9nueM8iK_64k.mp3
```

```bash
ffmpeg -i ~/Desktop/7aKdmDC9koXfxT9nueM8iK_64k.mp3
ffmpeg version 5.1.1-tessus Copyright (c) 2000-2022 the FFmpeg developers
  built with Apple clang version 11.0.0 (clang-1100.0.33.17)
  configuration: --cc=/usr/bin/clang --prefix=/opt/ffmpeg --extra-version=tessus --enable-avisynth --enable-fontconfig --enable-gpl --enable-libaom --enable-libass --enable-libbluray --enable-libdav1d --enable-libfreetype --enable-libgsm --enable-libmodplug --enable-libmp3lame --enable-libmysofa --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenh264 --enable-libopenjpeg --enable-libopus --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvmaf --enable-libvo-amrwbenc --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxavs --enable-libxvid --enable-libzimg --enable-libzmq --enable-libzvbi --enable-version3 --pkg-config-flags=--static --disable-ffplay
  libavutil      57. 28.100 / 57. 28.100
  libavcodec     59. 37.100 / 59. 37.100
  libavformat    59. 27.100 / 59. 27.100
  libavdevice    59.  7.100 / 59.  7.100
  libavfilter     8. 44.100 /  8. 44.100
  libswscale      6.  7.100 /  6.  7.100
  libswresample   4.  7.100 /  4.  7.100
  libpostproc    56.  6.100 / 56.  6.100
Input #0, mp3, from '/Users/bzy/Desktop/7aKdmDC9koXfxT9nueM8iK_64k.mp3':
  Metadata:
    encoder         : Lavf59.27.100
  Duration: 00:00:43.94, start: 0.046042, bitrate: 64 kb/s
  Stream #0:0: Audio: mp3, 24000 Hz, mono, fltp, 64 kb/s
At least one output file must be specified
```
#### 集成到代码
```python
# if len(audio_files) == 1:
#     final_file = audio_files[0]
# else:
#     final_file = get_cache_path().joinpath(f"{uuid4()}.mp3")
#     os.system(f"ffmpeg -i \"concat:{'|'.join(audio_files)}\" -acodec copy {final_file}")
if len(audio_files) == 1:
    final_file = audio_files[0].replace(".mp3", "-zip.mp3")
    command = f"ffmpeg -i {audio_files[0]} -b:a {bitrate}k {final_file}"
    os.system(command)
else:
    final_file = get_cache_path().joinpath(f"{uuid4()}.mp3")
    os.system(f"ffmpeg -i \"concat:{'|'.join(audio_files)}\" -acodec copy {final_file}")
    command = f"ffmpeg -i \"concat:{'|'.join(audio_files)}\" -b:a {bitrate}k {final_file}"
    os.system(command)
```

#### 文件大小比对
```bash
ls -l | grep 442801
-rw-r--r--@  1 bzy  staff   241440 Dec  8 14:30 442801.mp3
-rw-r--r--@  1 bzy  staff   100845 Dec  8 14:39 44280164k.mp3
```
可以看到 文件从241kb变成了100kb OK大功告成

#### 另外
降低`比特率`或者`采样率`都能降低音频文件的大小 可以考虑同时降低比特率和采样率 这里前端对音频质量要求并不高 只是试听所以只是降低比特率了
```bash
ffmpeg -i input.mp3 -ar 44khz output.mp3
```