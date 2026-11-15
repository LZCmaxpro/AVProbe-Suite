# AVProbe Suite

Android 上用于自查「设备到底支持什么」的两款小工具：音视频能力探测与相机能力探测。

系统接口说「支持」和它真的能用，中间隔着一整套厂商定制。这个项目的全部工作，就是把那条链路一环一环拆开，让结果可验证。

| 项目 | 能力 | 安装包 |
|---|---|---|
| **AVProbe** | 音频编码 / 全景声 / 视频解码 / 10bit / 杜比视界 / HDR / 屏幕规格 | `AVProbe-v1.5.1.apk`（标准版）、`AVProbe-ROOT-v1.5.1.apk` |
| **AVProbeCamera** | 全量镜头枚举 / 动态范围档位 / 10bit 采集 / 单镜头取景 | `AVProbeCamera-v1.2.4.apk`（标准版）、`AVProbeCamera-ROOT-v1.2.4.apk` |

四包均小于 200 KB，无三方库、无广告、不联网、不收集数据。

## 功能

### AVProbe

- **音频**：全部音频编码格式与声道能力，覆盖杜比全景声（EAC3-JOC）、DTS、AC3、EAC3、AAC、FLAC
- **视频**：解码能力、HEVC 的 profile 与 level（区分 Main 8bit 与 Main10）、杜比视界、HDR10 / HLG
- **屏幕**：`Display.getHdrCapabilities()` 返回的 HDR 类型、峰值亮度、色域
- **实测**：直接播放测试文件，实时显示本次命中的解码器与 mime，附「已命中的」面板
- **Root 版额外**：悬浮窗实时显示其他应用播放时命中的组件（只读日志，不注入、不 hook）

### AVProbeCamera

- **镜头枚举**：`getCameraIdList()` + `getPhysicalCameraIds()` + id 兜底试探，把系统默认隐藏的超广角、长焦、微距全部列出
- **档位读取**：`DynamicRangeProfiles`（API 33+）判断是否声明 10bit / 杜比视界录制；同时列出名字含 `dolby` / `hdr` / `10bit` / `log` 的厂商 tag
- **采集与编码**：相机是否能输出 `YCBCR_P010`、编码器是否支持 HEVC Main10
- **取景确认**：点任意一颗镜头进入全屏预览，`setPhysicalCameraId` 单独取流，失败自动退回父相机并明确标注

## 安装

标准版与 Root 版**包名不同**，可同时安装在一台设备上；同包名的新旧版本直接覆盖升级，四个包共用同一张签名证书。

| 包 | 包名 |
|---|---|
| AVProbe | `com.lzc.avprobe` |
| AVProbe ROOT | `com.lzc.avprobe.root` |
| AVProbeCamera | `com.lzc.avprobe.camera` |
| AVProbeCamera ROOT | `com.lzc.avprobe.camera.root` |

最低 Android 7.0（API 24），目标 API 34。

## 使用

1. 打开 AVProbe，「总览」页逐项给出判定；「播放」页可选择文件实测。
2. Root 版开悬浮窗后，切到任意播放器播放杜比全景声内容，小窗实时显示命中的组件与 mime。
3. 打开 AVProbeCamera，「相机」页列出全部镜头；点「查看」进入取景页，画面广不广一眼可见。
4. 首次启动会弹出免责声明，必须同意后才能进入，不同意即退出，这是有意设计。

## 实现摘要

- 无 Gradle，全程命令行手工构建：`aapt2 → javac → d8 → zipalign → apksigner`，详见 [docs/BUILD.md](docs/BUILD.md)
- 一套源码出双变体：构建期替换包名与 label，并用注释块标记删除 Root 专属权限
- 判定采用「三层验证」：系统声明 → profile 是否覆盖 10bit → 真放文件 `configure` 成功
- 相机能力不看 mime，看动态范围档位与厂商 tag；多摄要额外取物理镜头 id
- 完整技术拆解见 [docs/TECH.md](docs/TECH.md)，文件职责见 [docs/SOURCE.md](docs/SOURCE.md)，常见问题见 [docs/FAQ.md](docs/FAQ.md)

## 仓库结构

```
README.md            项目说明
DISCLAIMER.md        免责声明全文
CHECKSUMS.txt        四个安装包的 MD5 / SHA1 / SHA256 / CRC32
CHANGELOG.md         版本记录
docs/BUILD.md        手工构建完整步骤
docs/TECH.md         技术实现拆解
docs/SOURCE.md       源码文件职责与获取方式
docs/FAQ.md          常见问题
```

## 下载

安装包统一从 [Releases](../../releases) 页获取，页内附四个 APK、文件校验值与源码说明。

下载后请对照 [CHECKSUMS.txt](CHECKSUMS.txt) 核对 SHA256，不一致不要安装。

测试素材说明：仅自制样片与公开授权文件，不含影视片段。

## 关于 AI 参与

代码初版大量由 AI 生成，再逐版在真机上验证、修改、回退。分工大致是：需求定义与取舍、真机验收由人完成；构建脚本由 AI 出模板后人工适配；UI 与探测逻辑由 AI 生成初版后人工修改；崩溃通过贴完整堆栈由 AI 定位根因（例如 `child already has a parent` 指向重复挂载的 View）。

AI 产物不直接发布：每个功能都要在真机上点通才算完成。

## 已知限制

- 厂商私有通路读不到：部分机型的杜比视界录制、ROM 隐藏的镜头，标准接口不返回，工具无法补齐
- 播放时画面发灰不必然是解码失败，多为显示端未切入 HDR 模式
- Root 版监控只能说明「命中了哪个组件」，不能证明画质正确
- 设备覆盖有限，异常情况请附机型与截图提 issue

## 免责与许可

使用前请阅读 [DISCLAIMER.md](DISCLAIMER.md)：检测结果仅供参考，不构成专业报告；Root 版风险自担；打赏为自愿赠与、款项不退。

许可见 [LICENSE](LICENSE)：允许学习、编译、修改与分发，禁止二次打包售卖或上架商店。
