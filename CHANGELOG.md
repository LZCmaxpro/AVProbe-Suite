# 版本记录

## AVProbe

### v1.5.1 (versionCode 7)
- 首次启动强制免责确认（7 条，不同意即退出）
- 关于页：作者、版本、打赏入口
- 修复部分 ROM 上启动异常导致进不去的问题

### v1.5 (versionCode 5)
- 新增关于页与免责声明（静态展示）
- 新增「已命中的」面板
- 修复切换文件后仍在播放上一个文件的问题

### v1.4 及更早
- 早期版本，无关于页与免责声明，播放页存在遮挡问题，已停止分发

## AVProbeCamera

### v1.2.4 (versionCode 7)
- 新增单镜头取景预览（`PreviewActivity`，`setPhysicalCameraId` 单独取流）
- 取流失败自动退回父相机并标注
- 首次启动强制免责确认

### v1.2.3
- 补齐物理镜头枚举与镜头名推断（按后置组焦距相对大小）

### v1.2.2
- 修复启动必崩：`IllegalStateException: The specified child already has a parent`
- 根因：`buildPage()` 与 `showTab()` 重复挂载同一个 View

### v1.2 / v1.2.1
- 有免责声明但启动必崩，已停止分发

### v1.0 / v1.1
- 无免责声明且启动必崩，已停止分发
