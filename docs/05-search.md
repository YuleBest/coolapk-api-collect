# 搜索

搜索、候选词与热词接口。搜索类型通过 `type` 区分（如 `app`、`user`），结果分页与首页卡片流一致。Coolapk-API-Collect 的候选词文档把示例 URL 误写成 `/v6/account/checkLoginInfo`，实际路径为 `suggestSearchWordsNew`；coolapk-desktop 文档中的 `search/suggest` 与 `search/hotSearch` 在代码中由 `/v6/search?type=...` 与候选词接口承担。

共 6 个端点。标记含义：`verified` 至少两个独立项目源码出现；`single-source` 仅一个项目源码出现；`doc-only` 仅文档来源。
参数列保留原始提取文本，Retrofit 注解（如 `@Query("id")`）与类型信息未做二次改写；完整未截断内容见 `endpoints.json`。

| 方法 | 路径 | 参数 | 鉴权 | 验证 | 实测 | 来源 |
| - | - | - | - | - | - | - |
| GET | `/v6/search` | type:str:必要:app; page:num:必要:从1起; firstItem:num:非必要:首个卡片ID; lastItem:num:非必要:末个卡片ID; showAnonymous:num:非必要:-1; query: type=all\|feed\|apk\|game\|topic\|feedTopic\|hotSearch; searchValue; page; show_flag; sortType; sort; feedType; isStrict; pageType; refresh; returnType; query: p… | X-App-Token; X-App-Token/device headers; public read; apiDio headers + cookie jar; login cookie not asserted in code; AddCookiesInterceptor.kt:23-42 注入 User-Agent/X-Requested-With… | verified | ok | Coolapk-API-Collect, Coolapk-Lite, Coolapk-UNO, Coolapk-UWP, CoolapkApi, Flutter-Coolapk, c001apk, coolapk-desktop, coolapk-for-mac, coolapk-mcp |
| GET | `/v6/search/suggestSearchWordsNew` | searchValue:str:必要:搜索内容; type:str:非必要:app; query: searchValue; optional type=app; [{'name': 'searchValue', 'loc': 'query', 'required': True, 'inferred': False}, {'name': 'type', 'loc': 'query', 'required': True, 'inferred': False, 'values': …; {0}=keyWord; query: searchValue, ty… | X-App-Token; X-App-Token/device headers; public read; app_token_v3; app_token_v2; Cookie + X-App-Token + X-App-Device + X-Requested-With: XMLHttpRequest | verified | ok | Coolapk-API-Collect, Coolapk-Lite, Coolapk-UNO, Coolapk-UWP, coolapk-desktop, coolapk-for-mac |
| GET | `/v6/album/search` | query: q, page; @Query("q") String str, @Query("page") int i, @Query("firstItem") String str2, @Query("lastItem") String str3 | X-App-Token/device headers; public read; README.md:16-27 声称的统一 Header，X-App-Token 由 so/native 生成; 仓库无对应实现代码 | single-source | - | Coolapk-kotlin, coolapk-desktop |
| GET | `/v6/search/hotSearch` | - | unknown (doc claim) | doc-only | ok | coolapk-desktop |
| GET | `/v6/search/suggest` | - | unknown (doc claim) | doc-only | - | coolapk-desktop |
| GET | `/v6/{feedType}/search` | @Path("feedType") String str, @Query("q") String str2, @Query("page") int i, @Query("firstItem") String str3, @Query("lastItem") String str4 | README.md:16-27 声称的统一 Header，X-App-Token 由 so/native 生成; 仓库无对应实现代码 | doc-only | - | Coolapk-kotlin |
