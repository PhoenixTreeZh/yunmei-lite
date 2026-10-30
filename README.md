# 云莓Lite · PhoenixTree

Android 四功能客户端：开锁、云端取码、定位打卡、账号管理。版本 **0.3.3-dev**，包名 `com.yunmei.rewrite`，Android 8.0（API 26）及以上。可以覆盖之前的开发版，保留已保存账号与明确设置的偏好。

## 为什么做云莓Lite

这个项目起因于我在学校使用官方“云莓智能”App 时遇到的实际问题：操作流程繁琐，开门入口放在第二个页面，蓝牙连接与开锁过程等待较长，日常进出宿舍不够顺手。

为了让开门更直接、等待更少，我对官方 Android APK 进行了逆向分析，梳理账号接口、门锁通信流程及 BLE 协议，并在此基础上重新实现核心通信逻辑与客户端界面。云莓Lite将开锁放在首页，保留开锁、取码、打卡和账号管理四项日常功能，希望用更简洁的操作改善实际使用体验。

## 相关项目与交流

[云莓氛围（yunmei_vibe）](https://github.com/JustJoy122/yunmei_vibe) 为本项目提供了第三方客户端的思路参考。开发过程中，我们也就协议兼容性进行了交流；本项目向云莓氛围贡献了核心 BLE 逻辑与协议解析方案，相关成果已用于其门锁兼容性重构。感谢 JustJoy122 的交流与支持。

**鸿蒙版：[CloessyHarmony](https://github.com/Hannyueh/CloessyHarmony)**，由 Hannyueh 基于本项目开发和维护。欢迎关注不同平台上的实现，也感谢开发者对本项目的延伸与完善。

## 界面与配置

- 底部“首页 / 配置”。首页显示门锁卡片、大开锁按钮及实际发送阶段，密码获取后直接显示在密码按钮内，长按可复制。
- 主题可选“跟随系统 / 浅色 / 深色”，默认跟随系统。界面使用黑白配色，深色模式为黑色背景。
- “隐藏打卡按钮”默认开启，“隐藏密码按钮”默认关闭。已有手动设置保留；隐藏密码时停止自动取码并清除显示值。
- “快速连接”默认开启。已知有效蓝牙地址时直接连接，写入前连接失败可回退一次扫描；关闭时先扫描目标地址。
- “打开应用自动开锁”默认关闭。开启后从下一次进入应用开始，在账号、学校和所选门锁加载后自动尝试一次。权限/蓝牙窗口返回及屏幕重建不会重复触发，失败后不自动循环重试；手动开锁和停止操作会消耗本次自动尝试。
- “打开应用时自动获取密码”默认关闭。与自动开锁同时开启时先开锁，再取码；切换账号或门锁及退出前台会清除密码，迟到请求不能覆盖新门锁。
- 图标以官方 APK 图标为参考，使用不透明黑底。

首次使用在“配置 → 账号与门锁”添加账号，并选择学校、门锁。已保存账号会在进入应用时恢复登录；也可点“登录 / 刷新”。

## 本版成功提示的定义

**完整开锁指令写入完成后显示“开锁成功”**：支持带回调写入的特征需所有分包均收到 GATT 成功回调；无回调写入的特征需所有分包被系统接受，并保留最后一个分包的发送间隔。只写完第一包、任意失败或超时均不算完成。

这个提示只代表本地传输完成，不是锁舌传感器结果。本版不等待、解析门锁业务返回，也不再显示开锁后的现场选择。帧头、动作、ASCII 密钥及 AES 固定参数沿用已工作的实现；没有新增密码或其他附加写入。

本项目已在型号编号为 13 的宿舍门锁上完成实际开锁验证，其他型号的兼容性仍需实际测试。

## 存储与业务

登录凭据通过 AndroidKeyStore AES-GCM 加密保存在本机，明文密码不持久化。基础服务与学校 token 按请求独立绑定；取码必须来自服务器；打卡使用真实位置并由服务端判定结果。不伪造人脸、设备核验或学校授权。

型号能力由基础 `locktype/all` 与学校 `dormuser/getuserlock` 按 lockTypeId 合并。已匹配类型且 aesSupport 明确为 null 时按官方 4.2.2 的普通封装分支处理；完全缺字段、找不到型号、无效或冲突仍会拒绝发送。

## 构建

实际源码在 `app/src/four/`。界面使用 Kotlin + Compose + Miuix 0.5.1；Java 实现网络、账户、定位、自动触发控制和 BLE。需要 JDK 21、Gradle 8.9、AGP 8.7.2、Kotlin/Compose compiler 2.2.10、API 35 与 Build Tools 35.0.0，JVM 目标 17。

设置 JAVA_HOME，并配置本机 Android SDK 路径后执行：

```powershell
.\gradlew.bat :app:assembleDebug :app:lintDebug --no-daemon
powershell -File .\protocol-tests\run-entry-and-write.ps1
powershell -File .\protocol-tests\run-password-state.ps1
```

输出 `app/build/outputs/apk/debug/app-debug.apk` 使用调试签名，供本地测试。发布密钥和更新渠道未配置。`reference-upstream/` 仅为工作区历史参考，不参与构建或精简源码包。

## 来源与许可

除上述项目交流与贡献外，早期接口与兼容性研究也参考了 [云莓不智能（yunmei_unintelligent）](https://github.com/zxy19/yunmei_unintelligent)；历史参考说明保留在 [README.upstream.md](README.upstream.md)。

本工程保留 GPL-3.0 许可证，见 [LICENSE.txt](LICENSE.txt)。第三方组件说明见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。官方 APK 用于本地兼容性分析及图标参考，不随源码包分发。本应用由 **PhoenixTree** 维护。
