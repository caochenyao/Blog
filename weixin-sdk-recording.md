# 微信SDK使用总结

---

> * 序言
> * 开发过程中遇到的坑以及解决方法
> * 附录

---

## 序言
> 之前一段时间尝试了微信录音相关的开发工作，在开发过程中踩了不少的坑，现在已经慢慢步入尾声，于是抽时间总结了一下这段时间围绕微信录音开发过程中遇到的一些问题。希望可以帮到之后使用微信SDK开发的小伙伴避开一些陷阱。

---

## 开发过程中遇到的坑
### 1.关于startRecord和stopRecord
#### 遇到的问题描述：
> 1. startRecord和stopRecord不能过于频繁地调用，否则就会造成页面卡死。
> 2. 录音结束时，微信弹出提示显示录音失败，但却进入了stopRecord的success回调中，并且返回的信息与录音成功时返回的信息相同。
> 3. 关闭录音权限再开始录音，弹出是否打开录音权限的提示，点击“是”，进入到startRecord的success回调中，但页面上的正在录音却消失掉了，停止录音时会进入到stopRecord的fail回调中。

#### 解决方法：
> 1. 需要人为地控制startRecord和stopRecord调用的频率，目前将调用的时间间隔控制在了1s以上，比如如果调用startRecord开启录音，在1s以内用户点击想要停止录音，是不会调用stopRecord的，1s之后点击才会调用stopRecord来停止录音(ps: 此处感觉可以优化一下，如果用户在1s之内点击的话可以显示弹窗提示用户操作太快，目前是没有提示的)。
> 2. 最初遇到这个问题的时候我是崩溃的，因为不仅进入到success回调中，而且返回的数据结构感觉没有任何用处(最初认为是这样)，之后仔细考虑了考虑，既然微信判断录音失败了，那失败时的返回的数据结构中的内容肯定有所不同，后来抱着尝试的想法，将录音失败时与录音成功时返回的localId进行了对比，惊喜的发现录音失败时返回的localId其实是之前录音成功时的localId(推测是最近一次成功时返回的localId，不过还未验证)，以此为依据终于可以判断录音是否成功了。
> 3. 在正式进入录音之前，最好进行一个录音测试，可以有效地避免这种情况。

### 2.关于uploadVoice
#### 遇到的问题描述：
> 1. 调用uploadVoice时Android系统会出现页面卡死的现象。
> 2. Android系统中虽然将isShowProgressTips的值设为1，但也会出现一直在上传的情况。

#### 解决方法：
> 1. 最初遇到这个问题时，在网上查找了一些人的回答，给出的解释是Android系统中调用uploadVoice，在进入回调之前，是不能进行其他的操作的，而且页面也会出现UI卡死的情况，所以解决办法是将isShowProgressTips的值设为1(**也就是在Android系统中需要显示上传进度，iOS系统中可以视情况而定，一般而言iOS系统不需要这样做**)，但这样做虽然解决了Android系统中页面卡死的情况，但也由此引发了上面描述中的第二个问题。
> 2. 微信在出现上传超时的情况时也会进行一些处理，就是会进入到fail回调中，可以通过在fail回调中进行处理来解决出现的一直在上传的问题。(ps: 但这种解决办法还是有“上传中”的提示会显示三分钟左右的情况)

### 3.关于playVoice和stopVoice以及onVoicePlayEnd
#### 遇到的问题描述：
> 1. 最初遇到的问题与录音的两个接口相似，不能过于频繁的调用，否则就会出现页面卡死的情况。
> 2. onVoicePlayEnd有时候会出现不调用的情况

#### 解决方法：
> 1. playVoice和stopVoice的时间间隔控制在0.5~1s以上
> 2. 最初发现这个问题的时候感觉很奇怪，音频分明已经播放结束了，但却没有调用onVoicePlayEnd的success回调，当时一度以为是自己的代码有问题，后来在网上查找了一些人的回答才知道原来问题是因为onVoicePlayEnd这个接口不稳定。之后的解决办法如下(简写代码)：

```javascript
// playVoice
const startReplay = () => {
    wx.playVoice({
        localId: voiceId, // 完成录音后返回的本地音频的Id
        success: () => {
            this.replayTimer = setTimeout(this.stopReplay, duration);
        }
    });
    wx.onVoicePlayEnd({
        success: () => {
            this.replayTimer && clearTimeout(this.replayTimer);
        }
    });
}

// stopVoice
const stopReplay = () => {
    wx.stopVoice({
        localId: voiceId
    });
}
```

---

## 附录
[微信JS-SDK说明文档](http://qydev.weixin.qq.com/wiki/index.php?title=%E5%BE%AE%E4%BF%A1JS-SDK%E6%8E%A5%E5%8F%A3)
