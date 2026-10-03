# RealVolumeNormalizer

一个基于 **Web Audio API** 的浏览器视频自动音量标准化 Tampermonkey 脚本。

RealVolumeNormalizer 通过分析网页视频的实时音频信号，计算当前视频的实际响度，并自动调整视频网站播放器自身的音量条，使视频声音保持在用户设定的舒适范围内。

---
# Special Statement
本脚本和Readme由chatgpt编写，由本人监督和测试修改。

---
## Features

* 🎧 实时检测网页视频真实音量
* 📊 基于 RMS 音频分析进行响度估计
* 🔊 自动调整 HTML5 Video 音量
* ⬆️ 自动提升过低音量的视频
* ⬇️ 自动降低过高音量的视频
* 🎚️ 支持自定义目标音量范围
* ⏱️ 支持自定义检测时间窗口
* 🖥️ 内置悬浮控制面板
* 🔘 支持开启/关闭自动调整
* 📦 支持 Tampermonkey 安装

---

## How It Works

RealVolumeNormalizer 不直接修改音频输出，而是采用以下方式：

```
网页视频
   |
   v
Web Audio API Analyzer
   |
   v
实时检测音频响度
   |
   v
计算平均音量
   |
   v
调整 video.volume
   |
   v
正常播放输出
```

与传统 Gain 增益方案不同，本脚本不会持续修改音频采样，因此可以避免：

* 音量抽动
* 人声忽大忽小
* 音频处理造成的卡顿

---

## Installation

### 1. 安装 Tampermonkey

浏览器安装：

* Tampermonkey 浏览器扩展

### 2. 安装脚本

下载：

```
RealVolumeNormalizer.user.js
```

打开文件，复制内部文本，再打开浏览器插件油猴，创建新脚本，把复制内容放进去，保存，确保脚本打开。


---

## Usage

安装后打开支持 HTML5 Video 的视频网站。

右下角会出现控制面板。

默认参数：

```
最低响度:
-35 dB

最高响度:
-20 dB

检测窗口:
5 秒
```

脚本会：

* 当视频声音低于最低值时，提高播放器音量
* 当视频声音高于最高值时，降低播放器音量
* 当声音处于范围内时保持不变

---

## Interface

控制面板：

```
🔊 RealVolumeNormalizer

自动调节:
ON / OFF

当前响度:
-xx dB

播放器音量:
xx%

最低 dB:
-35

最高 dB:
-20

检测时间:
5s
```

---

## Compatibility

支持大多数基于 HTML5 Video 的视频网站。

理论支持：

* YouTube
* Bilibili
* 其他 HTML5 视频网站

部分网站可能拥有自定义播放器逻辑，可能会覆盖脚本对 `video.volume` 的修改。

---

## Limitations

由于浏览器安全策略和网站播放器实现不同：

* 无法保证所有视频网站完全兼容
* 用户手动开启播放器静音时，部分网站可能阻止自动恢复声音
* 当前版本使用 RMS 作为响度估计，不等同于专业 LUFS 标准

---

## Development History

### v1.x

最初尝试：

```
Web Audio API
+
GainNode
+
实时音量调整
```

发现：

* 正常语音会产生频繁调整
* 容易产生音量波动

---

### v2.x

改进：

```
Web Audio API
+
响度检测
+
video.volume 控制
```

目标：

让浏览器行为更接近系统级音量标准化。

---

## Future Plans

可能加入：

* LUFS 响度测量
* Jump Scare 爆音保护
* 不同视频网站适配
* 音量变化历史显示
* 自动保存用户配置
* 更智能的人声/音乐识别

---

## License

MIT License

Copyright (c) 2026

---

## Disclaimer

This project is for personal use and learning purposes.

The author is not responsible for unexpected behavior caused by third-party websites or browser updates.
