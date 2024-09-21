

开源VAD

https://github.com/snakers4/silero-vad

https://github.com/dpirch/libfvad



一、VAD算法原理

WebRTC的VAD算法基于高斯混合模型（GMM）进行语音和噪声的建模。GMM是一种无监督学习方法，通过对输入数据进行概率建模，可以实现对语音和噪声的有效区分。在WebRTC中，VAD算法将输入的频谱分为六个子带，并计算每个子带的能量。然后，使用GMM的概率密度函数对这些子带能量进行建模，得到一个对数似然比函数。通过对数似然比的全局和局部判断，实现对语音活动的检测。

二、VAD算法实现

WebRTC的VAD算法实现主要包括以下几个步骤：

1. 频谱分割：将输入的音频信号进行频谱分析，得到其频谱表示。然后，将频谱分成六个子带，分别对应不同的频率范围。
2. 能量计算：对每个子带的频谱能量进行计算，得到子带能量值。
3. GMM建模：使用GMM对子带能量进行建模，得到噪声和语音的概率密度函数。
4. 对数似然比计算：根据GMM的概率密度函数，计算每个子带的对数似然比。
5. 语音活动判断：根据对数似然比的全局和局部判断，判断当前是否有语音活动。

三、VAD算法应用

WebRTC的VAD算法在实时音视频通信中具有广泛的应用价值。首先，通过准确检测语音活动，可以减少非语音部分的传输，从而降低带宽消耗，提高通信效率。其次，VAD算法还可以用于[语音识别](https://cloud.baidu.com/product/speech.html)的前端处理，提高语音识别的准确率。此外，VAD算法还可以用于音频信号的压缩和编码，实现更高效的音频[存储](https://cloud.baidu.com/product/bos.html)和传输



## VAD语音活动检测开源库

libfvad是一个开源的语音活动检测（VAD）库，属于WebRTC本机代码包的一部分，可以用作独立于WebRTC代码其余部分的独立库(1)。

通过使用libfvad，开发人员可以方便地将语音活动检测功能集成到自己的应用程序中，以实现更精确的语音识别、语音合成、语音过滤等功能。在实际应用中，libfvad可以与其他开源库和工具（如PortAudio、PulseAudio、FFmpeg等）结合使用，以实现更丰富的音频处理功能(2)。

### libfvad

libfvad 是一个语音活动检测（Voice Activity Detection，简称VAD）的开源库，它提供了一组函数用于实现语音活动检测的功能。以下是libfvad中常用的函数：

1. `fvad_init()`: 初始化libfvad库，分配必要的内存和资源。
2. `fvad_set_parameters()`: 设置VAD参数，如信号处理参数、噪声抑制参数等。
3. `fvad_process()`: 执行语音活动检测，输入音频数据并返回检测结果。
4. `fvad_get_speech_segments()`: 获取语音活动检测结果中的语音段信息。
5. `fvad_get_speech_probability()`: 获取语音活动检测结果中的语音概率信息。
6. `fvad_get_noise_probability()`: 获取语音活动检测结果中的噪声概率信息。
7. `fvad_get_frame_size()`: 获取语音活动检测的帧大小。
8. `fvad_get_sample_rate()`: 获取语音活动检测的采样率。
9. `fvad_deinit()`: 释放libfvad库的资源，清理内存。