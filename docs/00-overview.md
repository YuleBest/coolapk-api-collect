# 通用约定与鉴权

本文档汇总多个开源项目的实现与文档。接口以酷安移动端 V6 API 为主，另含 Web/H5、账号域与开发者域路径。所有结论均标注来源，无法交叉确认的内容会明确标注。

## Host 与协议

| Host | 用途 | 来源 |
| - | - | - |
| `api.coolapk.com` | 主 API，绝大多数 V6 接口 | Coolapk-API-Collect、Coolapk-UWP、c001apk、coolapk-desktop |
| `api2.coolapk.com` | 只读回退与部分列表接口，路径与 api 相同 | coolapk-desktop、c001apk |
| `account.coolapk.com` | 登录、注册、验证码等账号域接口 | c001apk、CoolapkSDK |
| `www.coolapk.com` | 网页版与部分 HTML 页面 | coolapk-chrome-extensions、CoolapkUserStats |
| `developer.coolapk.com` | 开发者中心 HTML 页面 | CoolApk-Console |
| `m.coolapk.com` | 移动版网页 | c001apk 中的举报页跳转，未发现独立 JSON API |
| `image.coolapk.com` / `avatar.coolapk.com` / `static.coolapk.com` | 图片、头像与静态资源 CDN | 各客户端 |

协议为 HTTPS，数据格式以 JSON 为主；Web/H5 路径返回 HTML，需要按页面解析。

## 请求头

| 请求头 | 必要性 | 说明 |
| - | - | - |
| `User-Agent` | 推荐 | 形如 `Dalvik/2.1.0 (Linux; U; Android 16; ...) +CoolMarket/16.2.0-2604201-universal`；不设置可能报错 |
| `X-App-Id` | 必要 | `com.coolapk.market`；2017 年前的客户端使用 `coolmarket` |
| `X-App-Token` | 必要 | 按客户端版本有 v1/v2/v3 三代算法，见下文 |
| `X-Requested-With` | 必要 | `XMLHttpRequest` 返回 JSON；`com.coolapk.market` 返回 HTML |
| `X-Sdk-Int` | 必要 | Android SDK 版本，如 `36` |
| `X-Sdk-Locale` | 常用 | 如 `zh-CN`；Coolapk-API-Collect 将其必要性标为“未知” |
| `X-App-Version` | 必要 | 客户端版本名，如 `16.2.0` |
| `X-App-Code` | 必要 | 客户端版本代号，如 `2604201` |
| `X-Api-Version` | 必要 | API 版本；旧文档标注 `[6-13]`，2026 年客户端使用 `16` |
| `X-App-Device` | 常用 | 设备码，随机生成后持久化；生成方式见下文 |
| `X-Dark-Mode` | 可选 | `0`/`1` |
| `X-App-Channel` | 可选 | `coolapk` |
| `X-App-Mode` | 可选 | `universal` |
| `X-App-Supported` | 可选 | 通常与 `X-App-Code` 相同 |
| `Content-Type` | 按需 | 表单写接口使用 `application/x-www-form-urlencoded` |
| `Cookie` | 登录接口 | `token`、`username`、`uid` 三件套，另有 `SESSID`；部分写接口还需要 `ddid` 或 `_v2_post_token` |

来源：Coolapk-API-Collect `授权/HTTP 首部.md`；c001apk `util/AddCookiesInterceptor.kt`；coolapk-desktop `src-tauri/src/coolapk/client.rs`、`cookie_for_request`。

## X-App-Token 的三代实现

公开接口也要求 `X-App-Token`，它由设备码与时间戳生成，不依赖登录。不同年代的项目实现了三代方案，服务端兼容性以 2026 年客户端为准。

### v1：md5 + base64（约 2017-2020）

```
ts      = Unix 秒级时间戳
device  = 设备 UUID
raw     = BASE + md5_hex(str(ts)) + "$" + device + "&" + package_name
token   = md5_hex(base64_ascii(raw)) + device + "0x" + hex(ts)
```

- `BASE` 是内嵌常量，常见值 `token://com.coolapk.market/c67ef5943784d09750dcfbb31020f0ab?`；旧版客户端使用 `token://coolmarket/e85188a3bf413a7d0d3abe80a3b9f36d?`。
- 有效期约 5 分钟（`CoolapkUserStats` 注释），过期响应 `status == 1004`。
- 来源：`CoolapkTokenCrack/coolapk.py`、`coolapk_web/coolapk/src/api/token.ts`、`CoolApkApiTokenGenerator`、`CoolapkSDK`/`CoolapkTokenGetter` 的 `liba.so`。
- 已知矛盾：两个 2020 年前后的实现分别使用秒/毫秒时间戳、是否保留 `$`、尾部是否带 `0x`，无法判定哪一个在线上有效；本仓库不选择其中一版作为唯一答案。

### v2：bcrypt（约 2023-2024，c001apk 13.4.1）

```
ts            = System.currentTimeMillis() / 1000   # 秒，浮点字符串
base64Ts      = base64(ts)
md5Ts         = md5_hex(ts)
md5Device     = md5_hex(device_code)
raw           = "token://com.coolapk.market/dcf01e569c1e3db93a3d0fcf191a622c?" + md5Ts + "$" + md5Device + "&com.coolapk.market"
md5Base64Raw  = md5_hex(base64_ascii(raw))
salt          = ("$2a$10$" + base64Ts + "/" + md5_hex(raw))[0:31] + "u"
token         = "v2" + base64(bcrypt_hashpw(md5Base64Raw, salt).replace("$2a", "$2y"))
```

来源：c001apk `util/TokenDeviceUtils.kt:47-62`。

### v3：bcrypt + auth blob（2026，酷安 16.x）

```
ts       = Unix 秒级时间戳
index    = ((ts + 2604201) % 100) * 4 + 0x80
segment  = base64_decode(auth_blob.b64[index .. index + 0x80])
plain    = "com.coolapk.market" + "&" + segment + "&" + md5_hex(device) + "&" + ts + "&" + 2604201
password = md5_hex(base64_ascii(plain))
salt22   = shift_last_b64_char(base64_ascii(hex(ts) + "/" + md5_hex(plain))[0:22], -5)
token    = "v3" + base64_ascii(bcrypt_hashpw(password, salt22, cost=4, "$2y")).rstrip("=")
```

- `auth_blob.b64` 是一段分段存放的密码表，按时间戳索引取出片段后 Base64 解码；bcrypt 盐从 `hex(ts)/md5(plain)` 的 Base64 前 22 字符再往前移位得到。
- coolapk-desktop 使用 `APP_CODE=2604201`，并在当前时间点向后探测 15 秒（`timestamp + 0..15`）以寻找有效盐。
- 来源：coolapk-desktop `src-tauri/src/coolapk/auth.rs`、`docs/coolapk-api.md`（2026-08-06 实测批次）。
- `docs/COOLAPK_API_DOCUMENTATION.md` 声称 v3 使用 bcrypt cost=10，但同仓库 `auth.rs` 实际为 cost=4，且代码注释说明 cost=10 会导致登录信息错误。以代码为准。
- 2026-10-15 的只读实测中，cost=4 与 cost=10 生成的 v3 token 都能通过公开接口校验；写接口未验证，不能据此判断登录或写操作的取盐要求。
- 其他实现：`coolapk-mcp/coolapk_mcp/auth/token.py`、`token_v3.py` 也实现了 bcrypt 系方案；细节以各仓库源码为准。

### X-App-Device

常见生成方式是构造设备信息串后做 Base64、反转并去掉 `=`：

```
Base64(szlmId + "; ; ; mac; manufacturer; brand; model; buildNumber; null")
字符串反转，去掉 "="
```

来源：c001apk `util/DeviceCode.kt`、`util/TokenDeviceUtils.kt:25-44`；CoolapkSDK 注释中保留了另一种设备头算法。coolapk-desktop 对游客与登录用户分别生成设备码。

## 响应结构

成功响应有两种常见形态：

```json
{"data": {}}
```

- 裸 `data`：2026-10-15 实测中，`main/init`、`feed/detail`、`user/space`、`search`、`apk/detail` 等只读接口都返回这种结构。
- 信封结构：`{"code": 200, "message": "", "status": 1, "messageStatus": "", "data": {}}`。部分接口与写操作使用这种结构，`code == 200` 或 `status == 1` 表示成功。
- 错误响应通常是信封：`status` 为 `-1`、`401`、`404` 等，`message` 给出原因，例如未登录返回 `登录信息有误` 或 `你还没有登录，请先登录`，路径不存在返回 `xxx does not exists.`。
- `data` 为对象或数组；列表接口通常同时返回 `firstItem`、`lastItem`、`pageContext`、`hasMore`、`total` 等分页字段。
- `messageStatus` 为 `err_request_captcha_v2` 或存在 `messageExtra` 时，表示需要走验证码流程。
- Web/H5 接口返回 HTML，状态通过页面元素判断。

来源：coolapk-desktop `wrap_api_data`、`wrap_page_data_response`；Coolapk-API-Collect 各端点文档；c001apk。

## 分页与 ID

| 参数 | 说明 |
| - | - |
| `page` | 从 `1` 开始；部分旧接口使用 `p`，同样从 1 开始 |
| `firstItem` | 下拉刷新游标，取当前列表首个卡片的 ID；可为空 |
| `lastItem` | 上拉加载游标，取上一页最后一个卡片的 ID；可为空 |
| `firstLaunch` / `installTime` | c001apk 首页首次加载参数，首次为 `1`，之后为 `0` |
| `id` / `uid` | 动态、用户、评论等 ID；多数为数字，部分响应返回字符串 |
| `packageName` | 应用包名，旧接口也写作 `id` |

来源：Coolapk-API-Collect `主页/主页内容.md`、`搜索/搜索页面.md`；c001apk `ui/base/BaseViewModel.kt`；coolapk-desktop `wrap_page_data_response`。

## 已知冲突与可靠性说明

1. 多个写接口的方法存在矛盾。coolapk-desktop 的 `probe_all_write_endpoints_http_method` 断言 `/v6/feed/like`、`/v6/feed/unlike`、`/v6/feed/reply`、`/v6/feed/createFeed`、`/v6/user/follow`、`/v6/user/unfollow`、`/v6/message/send` 的 GET 请求可用，POST 返回“请求方式错误”；但生产代码与旧文档仍写 POST。表格同时保留两种方法，实际调用需以服务端当下行为为准。
2. `docs/COOLAPK_API_DOCUMENTATION.md` 声称来自反编译源码，但仓库内没有对应的反编译产物可复核。它独有的端点在本清单中标记为 `doc-only`，不能当作已验证事实。
3. 路径别名与真实路由不一致，例如 `/v6/feed/unfavorite` 对 `/v6/feed/unFavorite`、`/v6/message/delChatSession` 对 `/v6/message/deleteChat`、`/v6/member/recentHistoryList` 对 `/v6/user/recentHistoryList`。
4. `Coolapk-kotlin/README.md` 中内嵌 134 条反编译 Retrofit 接口样例，属于仓库内文档而不是编译代码，本清单按 `doc-only` 来源处理。
5. `account/checkLoginInfo` 的 `uid` 在参数文档中标注为 num，但响应示例为字符串；多数客户端按字符串处理。
6. Coolapk-API-Collect 的搜索候选页 curl 示例误写成 `/v6/account/checkLoginInfo`，实际标题路径为 `/v6/search/suggestSearchWordsNew`。

## 如何使用本仓库

- 先按领域文档查路径与参数，再在 `endpoints.json` 中查完整来源。
- `confidence` 只代表“出现在多少个独立项目中”，不代表当前可用。带 `doc-only` 的记录请先自行验证。
- 已实测的端点会在表格“实测”列显示 `ok`、`need-login` 或 `not-found`，完整过程见 `docs/11-live-check.md`。
- 需要真实可用性判断时，优先参考 coolapk-desktop 的探针记录（`probe` 字段，2026-08 批次）以及各客户端最近提交时间。
