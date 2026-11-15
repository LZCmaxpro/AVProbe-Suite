# 技术实现拆解

## 一、音视频能力：三层验证

只看一处必然误判，做法是逐层收紧。

### 第一层：系统声明了哪些组件

```java
for (MediaCodecInfo info : new MediaCodecList(ALL_CODECS).getCodecInfos()) {
    for (String type : info.getSupportedTypes()) {
        // audio/eac3-joc、video/dolby-vision、video/hevc …
    }
}
```

杜比全景声一般对应 `audio/eac3-joc`（EAC3 + JOC 对象音频）。厂商私有组件也会被列出。

### 第二层：声明是否覆盖到 10bit

```java
CodecCapabilities cap = info.getCapabilitiesForType("video/hevc");
for (CodecProfileLevel pl : cap.profileLevels) {
    if (pl.profile == HEVCProfileMain10HDR10) { /* 真 10bit */ }
}
```

很多设备能查到 `video/hevc`，但 profile 只到 Main（8bit），HDR 实际不可用。

### 第三层：真放一次，看落到谁身上

```java
MediaExtractor ex = new MediaExtractor();
ex.setDataSource(path);                      // 逐轨道读 mime / 声道 / 采样率 / 码率
MediaCodec dec = MediaCodec.createDecoderByType(mime);
dec.configure(format, surface, null, 0);     // configure 成功才算这条路通
```

HDR 判定不看 mime，看视频轨道元数据：`color-transfer`（ST2084 = HDR10，HLG = HLG）、`hdr-static-info`（存在即带 MaxCLL / MaxFALL）。屏幕侧走 `Display.getHdrCapabilities()`。

## 二、相机能力：接口不会直说

`CameraCharacteristics` 不直接声明「支持杜比视界录制」，而是通过动态范围档位间接暴露：

```java
CameraCharacteristics cc = cm.getCameraCharacteristics(id);
// API 33+：DynamicRangeProfiles，用反射取，低版本跳过
// DOLBY_VISION_10B_HDR_ORIG → 管线可出 DV 录制；只有 STANDARD → 仅声明 SDR
```

三个容易看漏的点：

1. 「未声明」不等于「不能拍」：部分厂商的杜比视界录制走私有通路，只能翻自定义 tag，因此页面会列出名字含 `dolby` / `hdr` / `10bit` / `log` 的 tag。
2. 10bit 采集看 P010：相机能否输出 `YCBCR_P010`。
3. 链路三环全通才成立：采集 10bit → 编码器支持 HEVC Main10 → 系统允许录制。

## 三、多摄：为什么只显示一颗

`getCameraIdList()` 只返回「能直接打开的相机」，其余被收进逻辑多摄：

```java
Set<String> physicalIds = cc.getPhysicalCameraIds();
```

每颗物理镜头还需单独 `getCameraCharacteristics(physId)` 才能读到传感器、光圈、焦距。枚举因此分三步：公开列表 → 物理镜头补齐 → 0–19 号 id 兜底试探。

镜头名不由系统提供，按**后置组焦距相对大小**推断：最短判超广角，最长判长焦，相对差不足 1.35 倍时统一标「后置」。

## 四、单镜头取景

```java
// 公开 id：直接 open
cm.openCamera(id, callback, handler);

// 物理镜头：打开父相机，把输出绑定到具体物理 id（API 29+）
OutputConfiguration oc = new OutputConfiguration(surface);
oc.setPhysicalCameraId(physId);
device.createCaptureSessionByOutputConfigurations(list, cb, handler);
```

预览用 `Matrix` 做中心裁切铺满，前置自动镜像。厂商不允许物理镜头单独取流时，自动退回父相机画面，并在底部标「以下画面来自父相机（多摄合成视角）」。

## 五、Root 悬浮窗

```bash
su -c "logcat -b main,media,system"
```

匹配 `Created component [` 与 `codec = `，再由邻近的 `mime = ` 补齐格式；前台应用用 `dumpsys window | grep mCurrentFocus` 每 4 秒轮询。悬浮窗由 `WindowManager` + `TYPE_APPLICATION_OVERLAY` 承载。

只读日志，不注入、不 hook、不修改系统行为。

## 六、界面全部代码搭建

```java
GradientDrawable bg = new GradientDrawable();
bg.setColor(0xF20A0F1A);          // 半透明面板
bg.setCornerRadius(dp(20));        // 圆角
bg.setStroke(dp(1), 0x33FFFFFF);   // 描边
```

圆形头像用 `ViewOutlineProvider` 裁切：

```java
avatar.setClipToOutline(true);
avatar.setOutlineProvider(new ViewOutlineProvider() {
    public void getOutline(View v, Outline o) {
        int r = Math.min(v.getWidth(), v.getHeight()) / 2;
        o.setRoundRect(0, 0, v.getWidth(), v.getHeight(), r);
    }
});
```

## 七、两个必须记下来的坑

### 坑一：`The specified child already has a parent`

`buildPage()` 先把页面 `addView` 进临时 `ScrollView` 再返回，`showTab()` 又把它挂进新的 `ScrollView`，同一 View 被要求有两个父容器。修法：每个页签缓存自己的 `ScrollView`，切页前先 `removeView` 解绑。

### 坑二：启动弹窗把 App 搞挂

直接在 `onCreate` 里 `showDialog`，个别 ROM 上弹窗构造异常导致整个应用起不来。修法：弹窗整体套 try/catch，失败只记日志放行。
