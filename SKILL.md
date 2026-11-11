---
name: 游戏站选品与上架
description: iframe 游戏站建好之后的持续运营方法论——去哪里找能嵌的游戏、怎么验证 embed 源不是空壳、新词游戏怎么抓、上架流程与 SEO 维护节奏。适用于已经有一个 iframe 游戏聚合站、需要持续上新和扩库的项目。与「游戏站开发与部署」互补：那个管建站上线，这个管上线之后的每天。
agent_created: true
---

# 游戏站选品与上架（运营期）

## 0. 适用范围

`游戏站开发与部署` 管的是建站与上线（技术栈、部署、GSC/GA）。
本 skill 管的是**上线之后**：游戏从哪来、怎么验、怎么上、怎么持续做。

核心矛盾：站点早期最大的风险不是没流量，是**上了不能玩的游戏**。玩家点 Play
得到白屏，跳出率飙升，Google 也会把站判成低质量。所以本 skill 把「验证」放在
「找」之后、上架之前的硬关卡。

---

## 1. 找游戏的渠道（按性价比排序）

来源：生财有术 PeterZEN《游戏网站如何获取更多游戏？7招助你拓展游戏库》
（精华帖，entityId `8852554411414582`）与《出海游戏网站如何玩？如何挖掘潜力游戏？》
（`4845251428522458`）。以下是在 pixlarc 项目上实测过的版本。

### 1.1 聚合站榜单（最省力，优先做）

聚合站已经替我们筛过一轮。直接抓它们的 new / trending 列表，减去已有的，
剩下的就是候选池。

| 源 | URL | 可用性（2026-09 实测） |
|---|---|---|
| zapgames.io 新游戏 | `https://zapgames.io/new` | ✅ 服务端渲染，`href="/<slug>"` 可直接正则抽 |
| zapgames.io 热门 | `https://zapgames.io/trending` | ✅ 同上 |
| onlinegames.io 新游戏 | `https://www.onlinegames.io/new-games/` | ✅ 列表页可抓（绝对链接），**具体游戏页 403** |
| poki.com 新游戏 | `https://poki.com/en/new` | ✅ 服务端渲染；会按 Accept-Language 跳到 `/zh/g/<slug>`，pattern 要通配语言段 |
| azgames.io 最近更新 | `https://azgames.io/recent` | ❌ JS 渲染，HTML 里只有 `/${game.slug}` 模板 |
| crazygames.com | `/new-games` `/newest` | ❌ `/new-games` 404，`/new` 是 JS 渲染 |
| itch.io / reddit | `/games/newest/html5`、`/r/webgames/new.json` | ❌ 本机直连被拦（HTTP 000） |

加新源 = 在脚本 `SOURCES` 里加一条 `{name, url, pattern}`，其余代码不动。

### 1.2 搜索引擎挖词

关键词：`iframe games`、`embed games`、`game distribution`、`html5 games embed`。
让 AI 把这些词拓展开，逐个去搜。**看到一个新站就记下来**——站越多，池子越大。

### 1.3 GitHub 代码搜索

搜索框输入 `<iframe language:html path:games/`。
搜出来的是别人的代码仓库，里面直接写着 embed 地址，等于白捡一批已验证的源。
（GitHub 左下角 Advanced search 可以进一步收窄。）

### 1.4 以图搜游戏（Google 镜头）

看上 Poki/CrazyGames 上的某款游戏但不给 iframe 地址时：
**拿游戏图标做以图搜图** → 右侧会出一堆同样收录了这款游戏的小站 → 进去看它们
有没有提供 embed，没有就用开发者模式找。

### 1.5 开发者模式兜底

进游戏页 → F12 → Elements 里搜 `iframe`，或 Network 里看加载了哪个域名。
没有 X-Frame-Options / CSP frame-ancestors 的地址就能拿过来嵌。

### 1.6 主动提供 embed 的平台

`crazygames.com`（游戏页找 **Embed** 按钮）、`gamedistribution.com`、
`onlinegames.io`、`twoplayergames.org`、`html5games.com`。
⚠️ 这类站给的 embed **会带它们自己的品牌和广告**（它们靠这个赚流量和钱）。
能用，但要接受这个代价。

### 1.7 源码与模拟器（进阶，非必要不做）

- 源码交易平台：`buy.html5games.com`、`codecanyon.net/category/html5/games`、Fiverr
- DOS 游戏：`js-dos` + `dos.zone`；Flash：`.swf` + `ruffle`
- 参考仓库：`github.com/GeekGallifrey/GamesPlayOnWeb`

---

## 2. 🔴 验证 embed 源：唯一不能跳过的关卡

**这是整个 skill 里最重要的一节。**

### 2.1 教训：只看状态码会放进白屏游戏

`azgames.io/<任意字符串>.embed` 对**任何** slug 都返回 **HTTP 200**，
响应头没有 X-Frame-Options，body 却是一个 2584 字节的空壳——`<body></body>`，
没有 iframe 没有 canvas。玩家点 Play 就是一片白。

pixlarc 曾经就这样上线了 3 款白屏游戏，且健康检查脚本报「全部 OK」——
因为脚本只查了状态码和响应头。

### 2.2 硬判据

一个 embed 地址**必须**同时满足：

1. HTTP 200 且 `Content-Type` 含 html
2. 响应头无 `X-Frame-Options: DENY/SAMEORIGIN`，CSP 无 `frame-ancestors`
3. **页面 body 里必须有 `<iframe>` 或 `<canvas>`**

第 3 条是关键。**没有载体的 200 就是软 404。**

### 2.3 引擎动态建 canvas 的情况

Unity / Godot / Construct / Phaser 这些引擎是 JS 运行时创建 canvas 的，
静态 HTML 里看不到。判据要放宽一档：

```
无 iframe 且无 canvas，但 body 含 unity|wasm|godot|phaser|pixi|construct|defold
  → JSLOADER(engine)：不判死，标记"需实机确认"
无 iframe 且无 canvas 且无引擎关键词
  → EMPTY：确定空壳，不能上架
```

### 2.4 实机确认闭环：`verified` 字段

JSLOADER 这一类静态判不了，**必须人工打开站点页面点一次 Play**（等 10 秒左右），
然后把结论写回数据，否则每周巡检会一直重复报同一条。

在 `games.json` 里给该游戏加：

```json
"verified": true,
"verifiedAt": "2026-09-26",
"verifiedNote": "Unity/WASM 加载，实机点 Play 确认可玩"
```

`check_games.py` 见到 `verified` 就把 JSLOADER 判成 `OK(verified)`，不再报警。
人工确认白屏的则直接 `disabled: true` + `disabledReason`。

顺带一提：源托管在个人 GitHub Pages（`*.github.io` 下的个人仓库）时，即使当前
能玩也要在 `games.json` 里记 `sourceRisk`，并优先找正规源替换——这种源随时会消失。

### 2.5 抓取客户端：用 curl 不用 urllib

本机两个坑：

- 默认走 `127.0.0.1` 代理 → curl 必须 `--noproxy '*'`
- `zapgames.io` 这类站挡 urllib 的 TLS 指纹 → 同样请求 curl 200、urllib 403

所以脚本里统一走 `subprocess` 调 curl，别用 `urllib.request.urlopen`。

### 2.6 探测时不要被第一个失败源下结论

`probe()` 要**试遍所有 embed 模板**再判定。azgames 对任意 slug 都返回空壳 200，
如果一遇到 EMPTY 就 return，那么「azgames 上没有、但 zapgames 上有」的游戏会被
整批误杀——pixlarc 实测同 12 个候选，修之前只出 3 个可用、修之后出 11 个。

同理，探测必须和 harvest 一样走 curl（urllib 会被 TLS 指纹挡掉 403）。

### 2.7 定期复查

源站会失效、会加防盗链。**每周跑一次健康检查**，别等用户反馈。

---

## 3. 新词游戏：怎么抓趋势

「新词游戏站」的玩法是：一款新游戏火了 → 用户去搜 → 提供这个游戏的网站还很少
→ 你上了就能吃这波流量。核心是**快**，其次是**选对**。

### 3.1 为什么用 Google Trends 而不是 Semrush/Ahrefs

新词足够新，第三方数据工具有更新滞后，查不到。Trends 是唯一能第一时间看到
飙升曲线的地方。看两个板块：**探索**（按词根找飙升词）、**时下流行**。

### 3.2 其他提前量渠道

- **聚合站 New games 分类** → 拿到新游戏词再去 Trends 验证趋势和体量
- **社媒异常值**：TikTok / YouTube 上「近期 + 低粉 + 数据异常高」的视频。
  Crazy Cattle 3D 就是在 itch.io 上数据平平、却在 TikTok 爆的——只看平台数据会漏
- **sitemap 挖宝**：`sitemap-extractor.gsctool.com` 直接看同行的 sitemap
  和最近收录的页面；aitdk 浏览器插件快速查 sitemap
- **关注开发者/系列**：过去出过爆款的开发者，新作更容易起量（如 neal.fun）
- **数据工具**：SimilarWeb / Semrush 找近期流量飙升的新页面，从词找站、从站找词
- **预告游戏**：发布前就有热度的最香，但风险是跳票或不及预期

### 3.3 决策标准

搜索体量够大 **且** 趋势向上 **且** 是新游戏 → 做。
下场成本不大，错过的代价更大。

---

## 4. 上架流程（每一步都要做）

```
1. 找候选     tools/hunt_games.py                 （聚合站榜单 − 已有）
2. 验源       tools/hunt_games.py --probe         （必须出 OK，不是 SOFT404/EMPTY）
2b. 人工确认   静态判为 JSLOADER(engine) 的，自己点一次 Play；
             能玩 → games.json 加 verified/verifiedAt；白屏 → disabled
3. 抓真实文案 从源站页面抓 title / description / controls
             ⛔ 不许按游戏名推断着写——见 §5.2
4. 分类+标签  分类用现有体系；标签必须跟着真实玩法走
5. 封面图    存 static/thumbs/<slug>.*，没有就回退 SVG
6. 写数据    games.json（幂等合并用 tools/merge_games.py）
7. 构建      python3 build.py
8. 校验      全站无残留 token、内联 JS 语法通过、sitemap 条数对得上
9. 部署      git push → Cloudflare Pages 自动构建
10. 推送     Bing/Yandex 走 IndexNow（tools/indexnow.py）
             ⚠️ Google 不走 IndexNow，必须去 GSC 手动提交 sitemap
11. 复查     一周后再跑 check_games.py，确认源还活着
```

---

## 5. 红线

### 5.1 版权 / DMCA

- 被投诉的后果很实在：GitHub 仓库被删、Google 把页面移出索引。
  `stimulationclicker.com` 因 iframe 了 neal.fun 的游戏被投诉，首页直接从 SERP 消失。
- 规避方向：优先用**主动提供 embed** 的平台（对方允许）；拿不准就换成跳转或自己
  做一版，别硬嵌。
- 敏感题材（涉真实人物/恶性事件）直接排除，AdSense 品牌安全会卡。

### 5.2 不许编造

- 游戏描述必须来自源站真实页面。**按游戏名推断出的文案会错**——
  pixlarc 原 12 款里有 6 款写错（"雪橇下滑"写成"山路开车"、"2D 射击"写成"物理格斗"）。
- 评分、播放数、发行日期这类**我们没有真实数据源**的字段，一律不填。
  编出来的数字放在页面上等于告诉用户"这是假的"，且违反 Google 结构化数据政策。
- 标签要跟着真实玩法走：改完文案要回头检查标签是否还成立。

---

## 6. 日常节奏

| 频率 | 动作 |
|---|---|
| 每周 | `check_games.py` 复查全部源；`hunt_games.py --probe` 出候选；上 3–5 款新游 |
| 每周 | 看 GSC 收录数变化、GA 实时/来源；Bing 侧靠 IndexNow 推 |
| 每周 | 看 GSC 收录数变化、GA 实时/来源；Bing 侧靠 IndexNow 推 |
| 每月 | 用 Google Trends + 社媒异常值挑新词；评估是否开新的长尾标签页 |
| 上线后 | 新页推 IndexNow + GSC 手动提 sitemap；1–2 周后再看收录 |

变现侧：AdSense 批不下来时先挂 Monetag（申请秒过），不要空窗等。
