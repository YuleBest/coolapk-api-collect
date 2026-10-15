# 酷安 API 收集与整理

基于 21 个开源客户端、SDK 与协议分析项目交叉整理酷安（Coolapk）接口，时间 2026-10-15。

当前规模：1015 条原始记录，356 个去重端点（`verified` 102、`single-source` 178、`doc-only` 76），另有 42 条无法静态还原或非端点的记录被排除。

2026-10-15 做了一轮限量只读实测：25 次 GET，21 次正常返回，2 次需登录，2 次确认 404，详见 `docs/11-live-check.md`。写接口未实测。

## 文档结构

| 文件 | 内容 |
| - | - |
| [docs/00-overview.md](docs/00-overview.md) | host、请求头、鉴权、响应结构与分页 |
| [docs/01-auth-account.md](docs/01-auth-account.md) | 账号、登录、Token 与设备 |
| [docs/02-home-feed.md](docs/02-home-feed.md) | 首页、信息流、动态、评论 |
| [docs/03-user.md](docs/03-user.md) | 用户、社交关系、收藏与历史 |
| [docs/04-topic-dyh.md](docs/04-topic-dyh.md) | 话题、标签、看看号、问答与投票 |
| [docs/05-search.md](docs/05-search.md) | 搜索、候选词、热搜 |
| [docs/06-app-product.md](docs/06-app-product.md) | 应用市场、游戏、数码产品、二手交易 |
| [docs/07-message.md](docs/07-message.md) | 通知、私信与系统消息 |
| [docs/08-media-misc.md](docs/08-media-misc.md) | 上传、媒体、表情、配置与其他 |
| [docs/09-web-hosts.md](docs/09-web-hosts.md) | Web/H5 与其他 host |
| [docs/11-live-check.md](docs/11-live-check.md) | 只读实测记录 |
| [endpoints.json](endpoints.json) | 机器可读清单，含参数、鉴权、来源与 `live_checks` |
| [sources.md](sources.md) | 参考项目、commit 与提取方式 |

## 可信度与实测

- `verified`：至少两个独立项目源码出现
- `single-source`：仅一个项目源码出现
- `doc-only`：仅文档来源
- `live_checks`：2026-10-15 只读实测结果

`confidence` 不代表当前可用；可用性以 `live_checks` 和文档表格中的“实测”列为准。

## 收集方法

1. 浅克隆参考项目并固定 commit，见 `sources.md`。
2. 从源码、注解、URL 常量与自带文档提取方法、路径、参数、鉴权与来源行号。
3. 按 `host + 归一化路径` 合并并统计验证等级，生成 `endpoints.json` 与分领域文档。

## 声明

仅用于学习、研究与互操作性测试。接口归酷安所有，可能变更；请勿批量抓取或违反服务条款。第三方项目版权归各自作者。
