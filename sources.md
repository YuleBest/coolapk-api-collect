# 参考项目与来源

所有仓库均在 2026-10-15 浅克隆，commit 为当日最新。提取方式：源码静态分析加自带文档交叉比对；文档类来源单独标记为 `doc-only`。

## 主要交叉验证来源

| 项目 | 语言 | Commit | 最后提交 | 提取重点 |
| - | - | - | - | - |
| [Coolapk-UWP/Coolapk-API-Collect](https://github.com/Coolapk-UWP/Coolapk-API-Collect) | Markdown | `a27a25a` | 2026-08-21 | 端点、参数、响应字段的文档描述 |
| [Coolapk-UWP/Coolapk-UWP](https://github.com/Coolapk-UWP/Coolapk-UWP) | C# | `c8c02ad` | 2026-09-11 | UWP 客户端真实调用与参数拼接 |
| [Coolapk-UWP/Coolapk-Lite](https://github.com/Coolapk-UWP/Coolapk-Lite) | C# | `ab70f9e` | 2026-10-11 | UWP 精简版真实调用 |
| [daimiaopeng/coolapk-desktop](https://github.com/daimiaopeng/coolapk-desktop) | Rust/Vue | `c71c5c9` | 2026-10-15 | 桌面端调用、自诊断探针、自带 API 文档 |
| [bggRGjQaUbCoE/c001apk](https://github.com/bggRGjQaUbCoE/c001apk) | Kotlin | `becc810` | 2024-05-08 | 安卓客户端注解与拼接接口 |
| [bjzhou/Coolapk-kotlin](https://github.com/bjzhou/Coolapk-kotlin) | Kotlin | `8c5b204` | 2017-10-13 | 早期客户端接口与参数形态 |
| [yeuban/Flutter-Coolapk](https://github.com/yeuban/Flutter-Coolapk) | Dart | `acde625` | 2020-05-01 | Flutter 客户端调用 |

## 补充实现

| 项目 | 语言 | Commit | 最后提交 | 提取重点 |
| - | - | - | - | - |
| [Lniosy/coolapk-mcp](https://github.com/Lniosy/coolapk-mcp) | Python | `e55264b` | 2026-07-06 | 新版 CLI/MCP 的接口调用 |
| [Mora-han/coolapk-for-mac](https://github.com/Mora-han/coolapk-for-mac) | Swift | `521ffb3` | 2026-09-25 | macOS 原生客户端调用 |
| [Coolapk-UWP/Coolapk-UNO](https://github.com/Coolapk-UWP/Coolapk-UNO) | C# | `411eff3` | 2026-01-24 | UNO 客户端调用 |
| [euanguo/CoolApkPC](https://github.com/euanguo/CoolApkPC) | TypeScript | `a485759` | 2022-08-05 | 桌面端接口调用 |
| [CoolapkLinux/coolapk_web](https://github.com/CoolapkLinux/coolapk_web) | HTML/JS | `9940998` | 2021-10-06 | Web/H5 接口 |
| [TaRGroup/CoolApk-Console](https://github.com/TaRGroup/CoolApk-Console) | Java | `cf7fa1b` | 2017-08-01 | 开发者控制台接口 |
| [zsakvo/coolapk-chrome-extensions](https://github.com/zsakvo/coolapk-chrome-extensions) | JavaScript | `cde4fe0` | 2020-01-13 | 浏览器扩展与 Web 接口 |
| [w568w/CoolapkSDK](https://github.com/w568w/CoolapkSDK) | Java | `bda75d8` | 2018-11-14 | 第三方登录与详情接口 |
| [by-syk/CoolapkUserStats](https://github.com/by-syk/CoolapkUserStats) | Python | `6a0b3ba` | 2017-05-23 | 用户数据爬取路径 |
| [onism68/CoolapkApi](https://github.com/onism68/CoolapkApi) | Go | `6df7e13` | 2020-07-24 | Token 生成与简单接口调用 |

## 协议与 Token 分析

| 项目 | 语言 | Commit | 最后提交 | 提取重点 |
| - | - | - | - | - |
| [ZCKun/CoolapkTokenCrack](https://github.com/ZCKun/CoolapkTokenCrack) | JavaScript | `d447da8` | 2020-04-26 | X-App-Token 算法与常量 |
| [by-syk/CoolapkTokenGetter](https://github.com/by-syk/CoolapkTokenGetter) | Kotlin | `11a65dd` | 2017-05-20 | 早期 Token 生成 |
| [PinkD/CoolApkApiTokenGenerator](https://github.com/PinkD/CoolApkApiTokenGenerator) | Python | `da831b0` | 2019-03-18 | Token 生成实现 |
| [XiaoMengXinX/FuckCoolapkFeeds](https://github.com/XiaoMengXinX/FuckCoolapkFeeds) | JavaScript/Go | `7638847` | 2026-10-13 | 网页转发站与上游调用线索（非 Xposed 模块） |

## 未能纳入

- 各仓库的 fork 与纯 UI 项目未纳入；如后续需要，可按同样流程追加。
