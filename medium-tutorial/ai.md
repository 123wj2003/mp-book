# AI智能图象解决方案

## 功能概述

本文对应实现例子为[tcb-demo-ai](https://github.com/TencentCloudBase/tcb-demo-ai)。本解决方案，AI智能图像能力是借助了腾讯云的[智能鉴黄](https://cloud.tencent.com/product/pornidentification)、[图片标签](https://cloud.tencent.com/product/image-tag)、[文字识别 OCR](https://cloud.tencent.com/product/ocr)、[人脸识别](https://cloud.tencent.com/product/facerecognition)、[人脸核身](https://cloud.tencent.com/product/facein)和[人脸融合](https://cloud.tencent.com/product/facefusion)功能，通过云开发的云函数和存储简化了素材的存储、配置的拉取和服务的调用 [image-node-sdk](https://github.com/TencentCloudBase/image-node-sdk) 来实现。

## 体验解决方案
敬请期待

## 解决方案 DEMO 源码
本章的案例代码，是在 [tcb-demo-ai](https://github.com/TencentCloudBase/tcb-demo-ai)。

## 解决方案 DEMO 接入流程
1. 请使用微信开发者工具打开 DEMO 源码，在根目录下的 project.config.json 文件，填写您的小程序 appid。

2. 通过此[链接](https://www.qcloud.com/login/mp?s_url=https%3A%2F%2Fconsole.cloud.tencent.com%2Fcam%2Fcapi)登录小程序对应的腾讯云帐号(需要小程序管理员权限)，然后在[云API密钥](https://console.cloud.tencent.com/cam/capi) 里获取 `SecretId` 和 `SecretKey`。

3. 本案例，前端页面(client/pages/)和云函数(cloud/functions)一一对应，如下：

|功能|前端页面|云函数|
|--|--|--|
|银行卡识别|bankCard|bankCard|
|名片识别（V2)|bizCard|bizCard|
|营业执照识别|bizLicense|bizLicense|
|行驶证驾驶证识别|drivingLicence|drivingLicence|
|人脸融合|faceFuse|faceFuse|
|通用印刷体识别|general|general|
|手写体识别|handWriting|handWriting|
|活体检测—获取唇语验证|idCardLiveDetectFour|idCardLiveDetectFour & faceLiveGetFour|
|身份证识别|idCard|idCard|
|车牌号识别|plate|plate|
|图片鉴黄|pornDetect|pornDetect|
|图片标签|tagDetect|tagDetect|

如果需要体验某个功能，需要在对应的云函数里参照 `config/example.js` 新建 `config/index.js`，并填入上面拿到的`SecretId` 和 `SecretKey`，然后创建并部署云函数。

## 解决方案源码介绍

### 活体检测—获取唇语验证

本案例实现了该服务的一些基础能力。整个逻辑流程如下：

<p align="center">
    <img src="https://main.qcloudimg.com/raw/440d2b156cff9803833e4423d1e96373.png" width="400px">
    <p align="center">实现逻辑</p>
</p>

其中云函数 `idCardLiveDetectFour` 的大体逻辑如下：

```js
// 首先把视频下载下来，获得视频内容的字符串内容
let res = await cloud.downloadFile({
    fileID: video
})

const buffer = res.fileContent

// 以 form-data 的格式，传到人脸核身服务进行校验
let formData = {
    validate_data: number,
    video: buffer,
    idcard_number: idcard,
    idcard_name: name
}

const result = await imgClient.faceIdCardLiveDetectFour({
    headers: {
        "content-type": "multipart/form-data"
    },
    formData,
});
```

在小程序端，需要有类似的遮罩，才能提供视频通过的概率。
<p align="center">
    <img src="https://main.qcloudimg.com/raw/c4991650a6adc8b21ffe619a5756788d.png" width="400px">
    <p align="center">视频遮罩</p>
</p>

那在小程序端怎么可以让图片等元素，盖在 `<camera>`, `<video>` 等原生的组件上面呢？答案是使用 `<cover-view>` 和 `<cover-image>`，譬如：

```html
<camera
    device-position="front"
    flash="off"
    binderror="error"
>
    <cover-view class="camera-cover">
        <cover-image 
            class="camera-image"    
            src="image path"
        >
        </cover-image>
    </cover-view>
    <cover-view
      class="number"
      wx-if="{{isRecording}}"
    >
        请念数字：{{number}}
  </cover-view>
</camera>
```