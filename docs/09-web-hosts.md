# Web/H5 与其他 Host

不属于 `api.coolapk.com`/`api2.coolapk.com` 的路径，主要来自网页版、开发者控制台、浏览器扩展与第三方登录流程。其中一部分是返回 HTML 的页面路由，不是 JSON API；表内一并保留，供抓取与互操作参考。

共 21 个端点。

| 方法 | 路径 | Host | 参数 | 鉴权 | 验证 | 实测 | 来源 |
| - | - | - | - | - | - | - | - |
| GET | `/emoticons/v9/{filename}` | static.coolapk.com | 无; - | 无（img src，明文 http）; 无 | verified | - | FuckCoolapkFeeds, coolapk-chrome-extensions |
| GET | `/n/{name}` | www.coolapk.com | 路径参数: {name}=用户名; {0}=name (inferred: user name or UID) | 无（直接 new URL(...).openConnection()，未加自定义头）; Cookie + X-App-Token + X-Requested-With: XMLHttpRequest | verified | - | Coolapk-Lite, Coolapk-UWP, CoolapkSDK |
| GET | `/` | account.coolapk.com | 无 | 无（浏览器跳转） | single-source | - | coolapk-chrome-extensions |
| GET | `/` | developer.coolapk.com | - | Cookie: SESSID, uid, username, token | single-source | - | CoolApk-Console |
| GET | `/UserHelper/getFollowRows` | www.coolapk.com | [{'name': 'uid', 'loc': 'query', 'required': True, 'inferred': False}, {'name': 'type', 'loc': 'query', 'required': False, 'inferred': False}] | none | single-source | - | coolapk-for-mac |
| GET | `/apk/{packageName}` | www.coolapk.com | - | 无 | single-source | - | CoolApk-Console |
| GET | `/apk/{param}` | www.coolapk.com | [{'name': 'packageName', 'loc': 'path', 'required': True, 'inferred': True}] | none | single-source | - | coolapk-for-mac |
| GET | `/do` | developer.coolapk.com | c=apk&m=myList&p={page}; c=apk&m=edit&id={id}; c=apk&m=list&apkType={type} \| c=apk&m=list&catid={catid} | Cookie: SESSID, uid, username, token; Cookie: SESSID, uid, username, token（未直接请求，按同类 /do 请求推断） | single-source | - | CoolApk-Console |
| GET | `/feed/{feedID}` | www.coolapk.com | 路径参数: {feedID}（源码再截掉 .shareKey 之后的部分） | 无（浏览器页面） | single-source | - | coolapk-chrome-extensions |
| GET | `/feed/{id}` | www.coolapk.com | -; shareKey={...}&shareUid={uid}&shareFrom=com.coolapk.app_4.8.1 | 无 | single-source | - | FuckCoolapkFeeds |
| GET | `/feed/{param}` | www.coolapk.com | [{'name': 'id', 'loc': 'path', 'required': True, 'inferred': True}] | none | single-source | - | coolapk-for-mac |
| GET | `/message/{param}` | www.coolapk.com | [{'name': 'uid', 'loc': 'path', 'required': True, 'inferred': True}] | none | single-source | - | coolapk-for-mac |
| GET | `/mp/do` | m.coolapk.com | query: c, m | browser cookie flow | single-source | - | Coolapk-Lite |
| GET | `/n/{param}` | www.coolapk.com | [{'name': 'name', 'loc': 'path', 'required': True, 'inferred': True}] | none | single-source | - | Coolapk-UNO |
| GET | `/picture/{id}` | www.coolapk.com | - | 无 | single-source | - | FuckCoolapkFeeds |
| GET | `/search` | www.coolapk.com | query: q={搜索框内容} | 无（浏览器跳转） | single-source | - | coolapk-chrome-extensions |
| GET | `/u/{param}` | www.coolapk.com | [{'name': 'uid', 'loc': 'path', 'required': True, 'inferred': True}] | none | single-source | - | coolapk-for-mac |
| GET | `/u/{uid}/album` | coolapk.com | 路径参数: {uid} | 无（request.urlopen 默认头） | single-source | - | CoolapkUserStats |
| GET | `/apk/com.coolapk.market` | www.coolapk.com | - | Cookie(token/username/uid 必要, SESSID 非必要)+X-App-Token/X-App-Id/X-App-Version/X-App-Code/X-Api-Version/X-Sdk-Int/X-Sdk-Locale/X-Requested-With/X-App-Device | doc-only | - | Coolapk-API-Collect |
| GET | `/u/{uid}` | www.coolapk.com | 路径参数: {uid} | 无 | doc-only | - | CoolapkUserStats |
| GET | `/u/{uid}/album` | www.coolapk.com | 路径参数: {uid} | 无 | doc-only | - | CoolapkUserStats |
