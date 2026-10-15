# 只读实测记录

2026-10-15 对重点只读接口做了限量真实请求验证。共 25 次请求，全部为 GET，单线程、间隔约 0.6 秒，未携带登录 Cookie，不涉及点赞、发帖、关注等写操作。

Token 使用 coolapk-desktop 的 v3 实现（`auth_blob.b64` + bcrypt）。本次只读测试中 cost=4 与 cost=10 都能通过校验；写接口是否要求特定 cost 未经验证。请求直接连接，未走代理。

结果分布：`ok` 21 次，`need-login` 2 次，`not-found` 2 次，其他 0 次。`need-login` 是预期行为：只读探测没有登录 Cookie。

主要结论：

- `main/init`、`main/indexV8`、`main/headline`、`search`、`search/suggestSearchWordsNew`、`apk/detail`、`apk/commentList`、`user/space`、`user/profile`、`topic/tagDetail`、`collection/list`、`feed/detail`、`feed/replyList`、`feed/hotReplyList` 均正常返回数据。
- `feed/detail` 在 `api.coolapk.com` 与 `api2.coolapk.com` 上返回结构一致。
- 只读接口多数返回裸 `{"data": ...}`，错误时返回带 `status`/`message` 的信封结构。
- 文档标为旧版的 `/v6/album/index`、`/v6/apk/hotList`、`/v6/search/hotSearch` 仍然可用。
- `/v6/feed/followFeedList` 与 `/v6/user/pictureList` 返回 `status=404`，确认已下线。
- `/v6/notification/list` 返回 `status=401`，接口存在但需要登录。

| 名称 | 方法 | Host + 路径 | 参数 | 结果 | HTTP | 返回形态 | 数据规模 |
| - | - | - | - | - | - | - | - |
| main_init_token_cost_4 | GET | `api.coolapk.com/v6/main/init` | {"t": "now"} | ok | 200 | data-only | list[6] |
| main_init_token_cost_10 | GET | `api.coolapk.com/v6/main/init` | {"t": "now"} | ok | 200 | data-only | list[6] |
| main_indexV8 | GET | `api.coolapk.com/v6/main/indexV8` | {"page": 1, "firstLaunch": 0, "installTime": 0, "firstItem": "", "lastItem": "", "ids": ""} | ok | 200 | data-only | list[11] |
| main_headline | GET | `api.coolapk.com/v6/main/headline` | {"page": 1} | ok | 200 | data-only | list[10] |
| search_suggest | GET | `api.coolapk.com/v6/search/suggestSearchWordsNew` | {"searchValue": "酷安", "type": "all"} | ok | 200 | data-only | list[21] |
| search_feed | GET | `api.coolapk.com/v6/search` | {"type": "feed", "feedType": "all", "sort": "default", "searchValue": "酷安", "page": 1, "showAnonymous": -1} | ok | 200 | data-only | list[20] |
| apk_detail | GET | `api.coolapk.com/v6/apk/detail` | {"id": "com.coolapk.market"} | ok | 200 | data-only | object keys=['id', 'apktype', 'group_id', 'catid', 'yyb_app_id', 'title', 'subtitle', 'version'] |
| apk_comment_list | GET | `api.coolapk.com/v6/apk/commentList` | {"id": "com.coolapk.market", "listType": "lastupdate_desc", "page": 1} | ok | 200 | data-only | list[20] |
| user_space | GET | `api.coolapk.com/v6/user/space` | {"uid": "536381"} | ok | 200 | data-only | object keys=['uid', 'username', 'admintype', 'groupid', 'usergroupid', 'level', 'experience', 'status'] |
| user_profile | GET | `api.coolapk.com/v6/user/profile` | {"uid": "536381", "installTime": 0, "showSettingTab": 1} | ok | 200 | data-only | object keys=['uid', 'username', 'admintype', 'groupid', 'usergroupid', 'level', 'experience', 'status'] |
| topic_tag_detail | GET | `api.coolapk.com/v6/topic/tagDetail` | {"tag": "酷安"} | ok | 200 | data-only | object keys=['id', 'hash', 'tag_type', 'category_id', 'second_category_id', 'tag_category', 'title', 'logo'] |
| collection_list | GET | `api.coolapk.com/v6/collection/list` | {"uid": "536381", "showDefault": 1, "page": 1} | ok | 200 | data-only | list[0] |
| notification_check | GET | `api.coolapk.com/v6/notification/checkCount` | {} | ok | 200 | data-only | object keys=['cloudInstall', 'notification', 'contacts_follow', 'message', 'atme', 'atcommentme', 'commentme', 'feedlike'] |
| account_check_login | GET | `api.coolapk.com/v6/account/checkLoginInfo` | {} | need-login | 200 | envelope | NoneType |
| album_index_doc_only | GET | `api.coolapk.com/v6/album/index` | {"page": 1} | ok | 200 | data-only | list[7] |
| apk_hot_list_doc_only | GET | `api.coolapk.com/v6/apk/hotList` | {"page": 1} | ok | 200 | data-only | list[20] |
| search_hot_doc_only | GET | `api.coolapk.com/v6/search/hotSearch` | {} | ok | 200 | data-only | list[2] |
| notification_list_doc_only | GET | `api.coolapk.com/v6/notification/list` | {"page": 1} | need-login | 200 | envelope | NoneType |
| follow_feed_list_offline? | GET | `api.coolapk.com/v6/feed/followFeedList` | {"page": 1} | not-found | 200 | envelope | NoneType |
| user_picture_list_offline? | GET | `api.coolapk.com/v6/user/pictureList` | {"uid": "536381", "page": 1} | not-found | 200 | envelope | NoneType |
| feed_hot_reply_offline? | GET | `api.coolapk.com/v6/feed/hotReplyList` | {"id": "58805277", "page": 1} | ok | 200 | data-only | list[1] |
| feed_detail | GET | `api.coolapk.com/v6/feed/detail` | {"id": "74360444"} | ok | 200 | data-only | object keys=['id', 'type', 'fid', 'forwardid', 'source_id', 'uid', 'username', 'dyh_id'] |
| feed_reply_list | GET | `api.coolapk.com/v6/feed/replyList` | {"id": "74360444", "listType": "lastupdate_desc", "page": 1, "discussMode": 1, "feedType": "feed", "blockStatus": 0, "fromFeedAuthor": 0} | ok | 200 | data-only | list[24] |
| feed_hot_reply_real_id | GET | `api.coolapk.com/v6/feed/hotReplyList` | {"id": "74360444", "page": 1} | ok | 200 | data-only | list[19] |
| feed_detail_api2 | GET | `api2.coolapk.com/v6/feed/detail` | {"id": "74360444"} | ok | 200 | data-only | object keys=['id', 'type', 'fid', 'forwardid', 'source_id', 'uid', 'username', 'dyh_id'] |

实测只是 2026-10-15 的快照，服务端随时可能变更；表中不包含返回的用户内容。
