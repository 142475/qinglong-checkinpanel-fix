# qinglong-checkinpanel-fix

[OreosLab/checkinpanel](https://github.com/OreosLab/checkinpanel) 的 `ck_bilibili.py` 修复版。

## 为什么

上游仓库最后更新于 2023-01（`ck_cloud189.py` / `ck_bilibili.py` 之后停更），B 站接口已变更，直接跑会崩：

```
File "ck_bilibili.py", line 352, in get_dynamic_videos
  for one in res.get("data", {}).get("archives", [])
AttributeError: 'NoneType' object has no attribute 'get'
```

## 修了什么（4 处）

| 位置 | 问题 | 修法 |
| --- | --- | --- |
| `get_dynamic_videos()` | `x/web-interface/dynamic/region` 已下线，返回 `code=-404 啥都木有` `data=null` | 改用 `x/web-interface/ranking/v2?rid={rid}&type=all`，取 `data.list`（字段 `aid`/`cid`/`title`/`owner.name` 与原 `archives` 一致） |
| `get_dynamic_videos()` | `res.get("data", {})` 在 `data=null` 时 `None.get()` 崩 | `(res.get("data") or {})` |
| `search_space_arc()` | `x/space/arc/search` 现需 WBI 签名，恒返回 `code=-799` `data=null` | 同样判空，返回 `[]`（该接口已不可用，不致命） |
| `main()` | `followings` / `aid_list[0]` 在接口异常时崩 | 判空 + `aid_list = aid_list or [{}]` |

## 用法

本仓库是**自包含**的：`ck_bilibili.py` 依赖的 `utils.py` / `utils_env.py` / `utils_ver.py` / `notify_mtr.py` 一并放在仓库根目录，无需依赖 checkinpanel 原目录。

1. 青龙面板 → 订阅管理 → 新建
   - 类型：公开仓库
   - 链接：`https://github.com/142475/qinglong-checkinpanel-fix.git`
   - 分支：`main`
   - 定时规则：如 `0 3 * * *`
2. 配置 `CHECK_CONFIG` 指向的 `check.toml`（`[BILIBILI]` 段，字段 `cookie` / `coin_num` / `coin_type` / `silver2coin`），或直接在青龙里设置 `CHECK_CONFIG` 环境变量。
3. 定时任务命令：`task 142475_qinglong-checkinpanel-fix_main/ck_bilibili.py`

## 上游同步

若上游将来更新了 `ck_bilibili.py`，把新版覆盖到本仓库再重新拉取即可；青龙侧订阅指向本仓库，**不会被 checkinpanel 的拉取覆盖**。
