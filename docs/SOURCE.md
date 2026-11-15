# 源码说明

本仓库当前托管的是项目文档与构建配方，源码文件与资源图、构建脚本以源码压缩包形式另行提供（含两个工程的完整目录、`build.sh`、`ic_launcher` 图标、`avatar` 与空白占位二维码）。

## 源码文件职责

### AVProbe（约 2.3k 行）

| 文件 | 行数 | 职责 |
|---|---|---|
| `MainActivity.java` | ~1580 | 六页签主界面、总览/播放/HDR/关于、启动免责弹窗、播放与命中面板 |
| `Probe.java` | ~230 | `MediaCodecList` 枚举、`MediaExtractor` 轨道解析、HDR 元数据与 `Display.getHdrCapabilities()` |
| `OverlayService.java` | ~360 | Root 版悬浮窗：`logcat` 匹配组件名与 mime、`WindowManager` overlay、前台应用轮询 |
| `Glass.java` | ~150 | 液态玻璃 UI 工具类：`GradientDrawable` 面板、pill 按钮、描边与圆角 |
| `Feature.java` | 7 | 变体标记（`ROOT_EDITION`），构建时由脚本重写 |
| `AndroidManifest.xml` | ~40 | 权限、`MainActivity`、`OverlayService`（`specialUse` 前台服务） |

### AVProbeCamera（约 2.7k 行）

| 文件 | 行数 | 职责 |
|---|---|---|
| `MainActivity.java` | ~1150 | 相机页、HDR 档位页、编码页、概览统计、免责弹窗、页签容器（`pageScrollers`） |
| `Probe.java` | ~600 | `scanCameras()` 三步枚举、`labelCams()` 镜头名推断、`fill()` 读 `CameraCharacteristics`、`readProfiles()` 动态范围档位、`scanEncoders()`、`report()` |
| `PreviewActivity.java` | ~470 | 单镜头取景：`TextureView` + Camera2、`setPhysicalCameraId`、退回父相机、中心裁切 Matrix、前置镜像 |
| `OverlayService.java` | ~290 | Root 版悬浮窗监控 |
| `Glass.java` | ~150 | 与 AVProbe 相同的 UI 工具类 |
| `AndroidManifest.xml` | ~50 | 双 Activity、`CAMERA` 权限、悬浮窗与前台服务权限 |

## 为何不逐文件上传

资源图含个人收款二维码，公开托管存在被他人滥用的风险，因此公开仓库只保留说明与构建配方；自行构建时请将 `res/drawable-nodpi/` 下的空白占位图替换为你自己的图片，或直接从自己的渠道获取源码包。

## 获取源码包

见仓库 Releases 页或 README「下载」一节。
