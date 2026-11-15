# 手工构建

不使用 Gradle。全部步骤只用 Android SDK 自带工具，一条链跑完两个变体。

## 环境

- JDK 17（`javac` 需在 PATH 中）
- Android SDK，`build-tools/34.0.0` 与 `platforms/android-34/android.jar`

```bash
export JAVA_HOME=/path/to/jdk-17
export PATH=$JAVA_HOME/bin:$PATH
BT=/path/to/sdk/build-tools/34.0.0
PLATFORM=/path/to/sdk/platforms/android-34/android.jar
```

## 五个步骤

```bash
# 1. 资源：编译 + 链接（生成 R.java 与未签名 apk）
"$BT/aapt2" compile --dir res -o build/res.zip
"$BT/aapt2" link -o build/unsigned.apk \
  -I "$PLATFORM" --manifest AndroidManifest.xml --java build/gen \
  --min-sdk-version 24 --target-sdk-version 34 \
  --version-code 7 --version-name 1.5.1 build/res.zip

# 2. Java：源码 + 生成的 R.java 一起编译
find build/java build/gen -name "*.java" > build/sources.txt
javac -source 11 -target 11 -encoding UTF-8 -nowarn \
  -classpath "$PLATFORM" -d build/classes @build/sources.txt

# 3. dex
d8 --release --min-api 24 --lib "$PLATFORM" \
  --output build/dex $(find build/classes -name "*.class")

# 4. 塞 dex + 对齐
cp build/unsigned.apk build/dexed.apk
(cd build/dex && zip -q -X ../dexed.apk classes.dex)
"$BT/zipalign" -f -p 4 build/dexed.apk build/aligned.apk

# 5. 签名 + 校验
keytool -genkeypair -keystore debug.keystore -alias avprobe \
  -storepass android -keypass android -keyalg RSA -keysize 2048 -validity 10950 \
  -dname "CN=AV Probe, OU=AudioTest, O=LZC, L=Zhangzhou, ST=Fujian, C=CN"

"$BT/apksigner" sign --ks debug.keystore --ks-pass pass:android --key-pass pass:android \
  --v1-signing-enabled true --v2-signing-enabled true \
  --out dist/AVProbe-v1.5.1.apk build/aligned.apk
"$BT/apksigner" verify dist/AVProbe-v1.5.1.apk
```

`zipalign -p 4` 的资源 4 字节对齐不可省：不齐会导致部分 ROM 加载失败。

## 一套源码出双变体

构建前对源码副本做改造，原始源码不动：

```bash
# 1) 改包名
sed -i "s/^package com\.lzc\.avprobe;/package com.lzc.avprobe.root;/" build/java/*.java

# 2) 改 Manifest 的 package 与 label
sed "s/package=\"com\.lzc\.avprobe\"/package=\"com.lzc.avprobe.root\"/; \
     s/android:label=\"AV Probe\"/android:label=\"AV Probe ROOT\"/g" \
  AndroidManifest.xml > build/AndroidManifest.xml

# 3) 生成变体标记类，代码里据此隐藏/显示 Root 专属入口
cat > build/java/Feature.java <<'EOF'
package com.lzc.avprobe.root;

public final class Feature {
    public static final boolean ROOT_EDITION = true;
    private Feature() {}
}
EOF
```

资源目录同样分两套：`res`（青图标）与 `res_root`（红图标），分别喂给 `aapt2 compile`。

权限用注释块标记，构建时整段删除或保留：

```xml
<!--ROOT_PERM_START-->
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<!--ROOT_PERM_END-->
```

## 常见构建问题

| 现象 | 原因 |
|---|---|
| `aapt2 link` 报找不到资源 | `res` 目录里少了引用的图片，先补齐图标与占位图 |
| `javac` 报找不到 R | `--java` 输出目录没加进 `sources.txt` |
| 装不上，提示签名冲突 | 旧包用了别的证书，先卸载旧版 |
| 启动崩溃 | 多数是页面容器重复挂载，见 docs/TECH.md 的坑二 |
