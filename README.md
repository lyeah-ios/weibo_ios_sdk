# 微博 iOS SDK

微博 iOS 平台 SDK，为第三方应用提供简单易用的微博 API 调用服务：无需了解复杂的验证机制即可完成授权登录，并支持通过微博官方客户端分享内容。

## 功能特性

| 模块 | 说明 | 状态 |
| ---- | ---- | ---- |
|  认证授权 | OAuth 2.0 授权认证，集成 SSO 登录 | ✅ 支持 |
|  微博分享 | 分享文本、相册多图、视频、Live Photo 至微博客户端 | ✅ 支持 |
|  登录 / 登出 | 一键 SSO 登录与授权取消 | ✅ 支持 |
|  Universal Link | 支持 Universal Link 分享跳转（3.3.0+） | ✅ 支持 |
|  分享到 Story | 单图或视频分享至 Story（3.2.1+） | ✅ 支持 |
|  分享超话 | 需联系商务开通权限（3.3.1+） | ✅ 支持 |

## 安装

### CocoaPods

在 `Podfile` 中添加：

```ruby
pod "Weibo_SDK", :git => "https://github.com/sinaweibosdk/weibo_ios_sdk.git"
```

然后执行：

```bash
pod install
```

### 环境要求

| 项目 | 要求 |
| ---- | ---- |
| 平台 | iOS |
| 开发语言 | Objective-C |
| 微博客户端 | 按功能需求而定，详见[更新日志](#更新日志)备注 |

## 快速开始

### 1. 前置条件

使用此 SDK 需满足以下条件：

- 在新浪微博开放平台注册并创建应用；
- 已定义本应用的授权回调页；
- 已选择应用为 iOS 平台，并正确填写 Bundle ID 和 Apple ID。

> **注**：授权回调页对移动客户端应用来说对用户是不可见的，因此定义为何种形式都不影响，但**没有定义将无法使用 SDK 认证登录**。建议使用默认回调页 <https://api.weibo.com/oauth2/default.html>。

### 2. 配置 info.plist

<details open>
<summary><b>iOS 9/10 ATS 白名单配置</b></summary>

iOS 系统中默认需要为每次网络传输建立 SSL，需将以下白名单添加到应用的 plist 中，否则可能遇到 `An SSL error has occurred and a secure connection to the server cannot be made.` 的问题。

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>sina.cn</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
        <key>weibo.cn</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
        <key>weibo.com</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSThirdPartyExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
        <key>sinaimg.cn</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSThirdPartyExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
        <key>sinajs.cn</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSThirdPartyExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
        <key>sina.com.cn</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSThirdPartyExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.0</string>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
        </dict>
    </dict>
</dict>
```

> 如仅需快速验证，也可强制将 `NSAllowsArbitraryLoads` 设置为 `YES`（不推荐用于上架）。
>
> 从 iOS 10 起，每个域名下需添加 `NSExceptionMinimumTLSVersion`，值为 `TLSv1.0`（上述示例已在 `sina.com.cn` 中体现）。

</details>

<details open>
<summary><b>应用跳转 scheme 白名单</b></summary>

如需跳转到微博客户端（登录、分享等），需在 plist 中添加以下键值对，否则 `canOpenURL` 会返回 `NO`。

了解详情请至 [WWDC 2015 Session 703](https://developer.apple.com/videos/wwdc/2015/?id=703)。

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>sinaweibohd</string>
    <string>sinaweibo</string>
    <string>weibosdk</string>
    <string>weibosdk2.5</string>
    <string>weibosdk3.3</string>
</array>
```

> **注**：SDK 升级到 3.4.2 后，只需保留 `weibosdk`。

</details>

### 3. 名词解释

| 名词 | 注解 |
| ---- | ---- |
| AppKey | 分配给每个第三方应用的 app key。用于鉴权身份、显示来源等功能。 |
| RedirectURI | 应用回调页面，可在“新浪微博开放平台 → 我的应用 → 应用信息 → 高级应用 → 授权设置 → 应用回调页”中找到。 |
| AccessToken | 表示用户身份的 token，用于微博 API 的调用。 |
| Expire in | 过期时间，用于判断登录是否过期。 |

## 平台适配说明

### iOS 9

- **传输安全**：默认每次网络传输需建立 SSL，需配置 ATS 白名单（见上方[-配置 info.plist](#2-配置-infoplist)）。
- **应用跳转**：需在 plist 中添加 `LSApplicationQueriesSchemes`，否则 `canOpenURL` 返回 `NO`。
- **bitcode**：最新代码已支持应用瘦身与 bitcode。

### iOS 10

原有 ATS 设置在 iOS 10 上会出现 HTTPS 网络访问限制，需在每个域名下添加 `NSExceptionMinimumTLSVersion`，值为 `TLSv1.0`。

### ADSupport

- **如需集成**：提交 App Store 时勾选用于跟踪用户广告效果的选项即可，具体操作可参照友盟：[参考链接](http://bbs.umeng.com/forum.php?mod=viewthread&tid=6242&aid=1611&from=album&page=1&mobile=2)
- **如不集成**：直接删去即可，SDK 并非强制要求集成，不影响正常使用。

### IPv6

苹果商店自 6 月 1 日起强制要求 app 支持 IPv6-only 网络，微博 SDK 已更新底层网络代码（包括 reachability 库）予以支持。

## 文档

- 📖 [API 文档](http://sinaweibosdk.github.io/weibo_ios_sdk/index.html)
- ❓ [常见问题 FAQ](./FAQ.md)
- 💻 [GitHub Repository](https://github.com/sinaweibosdk/weibo_ios_sdk)

## 常见问题

关于 SDK 的 Bug 反馈、用户体验以及好的建议，请尽量提交到 [GitHub Issues](https://github.com/sinaweibosdk/weibo_ios_sdk/issues)，我们会尽快处理。更多问题请参阅 [FAQ](./FAQ.md)。

## 联系我们

这里有 4 个 QQ 群，iOS 开发者请优先加入 453830884（其余 3 个已满）。

| 渠道 | 信息 |
| ---- | ---- |
| QQ 群（iOS 请加此群） | 453830884 |
| QQ 群（已满） | 226214250 |
| QQ 群（已满） | 263989257 |
| QQ 群（已满） | 284084420 |
| 开发者 | 北京微梦创科网络技术有限公司 |
| 邮箱 | wbkf@staff.weibo.com  |
| 微博 | 移动新技术 |

目前，我们正在逐步完善微博 SDK，争取为第三方开发者提供一个规范、简单易用、可靠、可扩展、可定制的 SDK，敬请期待。

## 更新日志

| 版本 | 更新内容 | 备注 |
| ---- | -------- | ---- |
| **3.4.2** | scheme 白名单精简为仅保留 `weibosdk` | 解决xcode27白名单缩减到25 |
| **3.3.2** | 支持分享 Live Photo | 微博 App ≥ 11.7.2 |
| **3.3.1** | 支持分享超话 | 微博 App ≥ 11.6.2；需联系商务开通权限 |
| **3.3.0** | 支持 Universal Link 跳转；修复 iOS 14 多图/视频丢数据；可关闭 IDFA 获取；去掉相册权限 | scheme 白名单需添加 `weibosdk3.3` |
| **3.2.7** | 替换 `NSURLConnection` / `UIAlertView` API；增加 i386 架构 | — |
| **3.2.6** | 修复页面适配及 iTunes 下载链接；修复部分 issue；补充 FAQ | — |
| **3.2.5** | 替换 `UIWebView`；适配 iOS 13；精简包体积 | — |
| **3.2.1** | 分享支持相册多图、视频；新增分享到 Story | — |
| **3.2.0** | 精简为 4 大模块，多项功能下线 | 热评、私信、短信登录下线，多媒体分享改为文本链接 |

## 开源协议

微博 SDK 以二进制形式分发，使用须遵守《微博开放平台开发者协议》及微博 SDK 许可条款。详见 [新浪微博开放平台](https://open.weibo.com/)。
