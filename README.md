# 计件计时工资计算器 - Android APK 云构建工程

这是一个 WebView 壳工程，把单文件 HTML（`app/src/main/assets/index.html`）封装成安卓 App，
通过 GitHub Actions 自动编译出 APK，不需要本地安装 Android Studio。

---

## 一、工程结构

```
WageCalc-apk/
├── .github/workflows/build-apk.yml     # 云构建流程（自动编译 APK）
├── build.gradle                        # 根构建脚本
├── settings.gradle                     # 模块与仓库配置
├── gradle.properties                   # 构建参数
└── app/
    ├── build.gradle                    # App 模块配置（compileSdk 34 / minSdk 26）
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/example/wagecalc/MainActivity.java   # WebView 壳
        ├── assets/index.html                             # 你的页面（原样放入）
        └── res/                                          # 名称、图标
```

## 二、上传到 GitHub

### 方式 A：用电脑（推荐，5 分钟）

1. 登录 https://github.com ，右上角 `+` → `New repository`
2. 仓库名填 `wagecalc-apk`，选 **Private**（私有）或 Public，点击 `Create repository`
3. 解压本工程压缩包，得到 `WageCalc-apk` 文件夹
4. 在仓库页面点 `uploading an existing file`，把文件夹里**所有内容**拖进去（包括 `app`、`build.gradle` 等）
5. 点 `Commit changes`
6. 因为 `.github` 是隐藏目录，网页拖拽可能漏掉。手动补：仓库页面 `Add file` → `Create new file`，
   文件名输入 `.github/workflows/build-apk.yml`，把本工程里该文件的内容整段粘贴进去，`Commit changes`

> 也可以直接勾选拖拽时包含隐藏文件（Windows 显示隐藏项目后拖整个文件夹）。

### 方式 B：只用一个仓库根上传

如果不想处理子目录，可以把 `WageCalc-apk` 里面的内容直接作为仓库根目录结构上传，
即仓库根下就有 `app/`、`build.gradle`、`settings.gradle`。workflow 脚本默认就是这个结构。

## 三、触发构建并下载 APK

1. 仓库页面顶部点 `Actions`
2. 左侧选 `Build APK`
3. 若未自动运行，点右侧 `Run workflow` → `Run workflow`
4. 等 3~5 分钟，状态变成绿色对勾
5. 点进这次运行记录，页面底部 `Artifacts` 区域下载 `wagecalc-debug-apk.zip`
6. 解压得到 `app-debug.apk`，传到手机上安装即可

> APK 是 debug 签名，安装时手机会提示"未知来源"，允许即可。可以直接用，但不能上架应用商店。

## 四、修改页面内容后如何更新

只改 `app/src/main/assets/index.html`，改完 `Commit changes`，Actions 会自动重新构建，重新下载新版 APK 即可。

> 注意：新版 APK 与旧版签名一致（都是 debug 签名），可以直接覆盖安装，本地数据（localStorage）会保留。

## 五、技术说明

- **WebView 加载方式**：使用 `WebViewAssetLoader`，以 `https://appassets.androidplatform.net/assets/index.html`
  加载本地页面，而不是 `file://`。原因：`file://` 协议 origin 为 `null`，`localStorage` 在部分
  WebView 版本下不持久化，会导致单价表、参数设置重启后丢失。
- **setDomStorageEnabled(true)**：必须开启，否则页面里的 `localStorage` 完全不生效。
- **setWebChromeClient**：必须设置，否则页面里的 `alert()`、`confirm()` 不会弹窗（静默失效）。
- **minSdk = 26**：直接使用自适应图标（`mipmap-anydpi-v26`），无需打包多套 PNG 图标。
- **targetSdk = 34**：符合当前 Google Play 要求。

## 六、自定义项

| 想改什么 | 改哪里 |
| --- | --- |
| 应用名称（桌面显示） | `app/src/main/res/values/strings.xml` 的 `app_name` |
| 应用图标 | `app/src/main/res/drawable/ic_launcher_foreground.xml`（图形）、`values/colors.xml`（背景色） |
| 包名 | `app/build.gradle` 的 `namespace` / `applicationId`，以及 Java 文件的 package 与目录 |
| 版本号 | `app/build.gradle` 的 `versionCode` / `versionName` |
| 页面内容 | `app/src/main/assets/index.html` |

## 七、出正式签名 APK（可选）

若要发布，需要生成 keystore 并把签名配置写进 GitHub Secrets，然后跑 `assembleRelease`。
需要时告知，可补一套带密钥签名的 workflow。
