# loon-goofish-adblock

Loon 插件：闲鱼去广告（自托管版）。

规则逐条取自 kelee.one 的原插件 `FleaMarket_remove_ads`（作者 可莉🅥），原文 `#!date=2026-10-02 13:57:49`，
sha256 `9c897758cc4853e63d79de8e233f8144ee1545a8798715688e6a172976e26ff5`，副本见 [`original/`](original/)。
**拦截项一条不加、一条不减、一条不改**：24 条复写（7 条 `reject_dict` + 15 条 `response.json.jq` + 2 条 `response.json.delete`），
`[MitM]` 两个域名原样保留。唯一动过的是插件头部——图标与主页指向本仓库、描述加了版本号。

## 订阅地址

主文件（新语法，需要 Loon 3.5.1(998) 及以上）：

```
https://raw.githubusercontent.com/Wansensr/loon-goofish-adblock/main/Goofish-AdBlock.plugin
```

备用（经典 Rewrite 语法，**拦截项与主文件逐条相同**，只是写法不同）：

```
https://raw.githubusercontent.com/Wansensr/loon-goofish-adblock/main/Goofish-AdBlock-classic.plugin
```

`raw.githubusercontent.com` 连不上时用 jsDelivr 固定 commit 镜像（不用 `@main`，避免 CDN 缓存滞后）：

```
https://cdn.jsdelivr.net/gh/Wansensr/loon-goofish-adblock@7cfc1dcef40466201861716586a730044728df85/Goofish-AdBlock.plugin
```

（固定 commit 的好处：jsDelivr 对 `@main` 有缓存，可能滞后几小时；钉住 commit 就永远是这一份。
经典语法那份把文件名换成 `Goofish-AdBlock-classic.plugin` 即可。）

## 添加与更新

- 添加：Loon → 配置 → 插件 → 右上角 `+` → 从 URL 添加 → 粘贴订阅地址。
- 更新：进插件详情页点右上角 ⟳。**订阅地址不变，不用删除重加。**
- 判断更新有没有生效：看插件详情页「详细」里**描述那一行**，开头应该是当前版本号（现在应为 `v1 ·`）。
- 只保留这一个闲鱼插件，不要和 kelee 的原插件同时开启——同一批接口两边都有复写，会互相顶掉。

## 关于「失效」——实测结论

原插件**没有任何外部依赖**：没有 `[Script]` 段、没有远程 JS、没有 `jq_file`，24 条全部是 Loon 原生动作。
所以「kelee.one 挂了 → 需要改响应体的条目整块静默失效」这个判断**不适用于闲鱼这款**（那是高德那款的情况）。
实测 kelee.one 对非浏览器客户端返回 Cloudflare 403（`Attention Required!`），但 `hub.kelee.one` 正常（200，207 KB）
——这影响的是「抓原文」，不影响插件运行。

真正会让条目失效的是**接口本身没了**。把 24 条逐个打到闲鱼 mtop 网关（无签名请求，只看网关错误码：
`FAIL_SYS_PROTOVER_MISSED` = 接口在册，`FAIL_SYS_API_NOT_FOUNDED` = 路由表里没有这个名字；GET 与 POST 结果一致）：

在册 18 条，**已不存在 6 条**：

- `mtop.taobao.idlehome.home.nextfresh` —— 首页信息流
- `mtop.taobao.idle.item.recommend.list` —— 猜你喜欢 / 推荐流
- `mtop.taobao.idle.local.home` —— 本地（同城）首页
- `mtop.taobao.idlemtopsearch.search.shade` —— 搜索栏填充词
- `mtop.taobao.idlemtopsearch.item.search.activate` —— 搜索热词
- `mtop.taobao.idle.trade.full.info` —— 订单页精简

这 6 条在当前版本闲鱼上**永远不会触发**。对应入口的推广如果还在，是因为闲鱼换了新接口，而插件拦的仍是旧接口。
**这不代表可以顺手替换**：换接口等于改动拦截范围，本次没有做。

（补充：社区同类规则里的 `idle.local.home` 多数是**前缀**匹配，能顺带命中 `idle.local.home.top`、
`idle.local.home.feeds` 这些子接口；原插件这一条精确到 `idle.local.home/`，所以它单独失效。）

另外：上游 2026-10-02 那一版相对前一天（`#!date=2026-09-28 17:03:58`，经典语法）**规则逐条完全相同**
（24/24 且顺序一致），唯一差别是复写语法从经典换成新语法、最低 Loon 版本从 3.4.0(962) 提到 3.5.1(998)。
这是本仓库额外提供经典语法版的原因：如果你的 Loon 低于 3.5.1(998)，或新语法那份在你机器上不生效，换它。

## 版本记录

- **v1**（2026-10-13）：自托管建仓。拦截项与原插件 2026-10-02 版逐条一致，未增删改任何规则。

## 排查

- 描述行不是 `v1 ·` 开头 → 手机还没拉到：点插件详情页右上角 ⟳；仍旧不行就删掉这个插件重新从 URL 添加。
- 描述行已是 `v1 ·`，但首页 / 我的 / 搜索里的推广照旧 → 先换成**经典语法**那份订阅（写法不同、规则相同）。
  能被这一步修好，说明问题出在 Loon 对新语法的解析；换完依旧照旧，那就是上面那 6 条接口已经不存在，
  需要新增规则才能覆盖。
- 别和 kelee 的原插件同时开启。

## 来源与校验

- 原文来源：kelee.one 原插件镜像。本仓库 `original/FleaMarket_remove_ads.original.lpx` 即原文副本，
  6423 字节，sha256 见上（与 kelee.one 官方 `hub.kelee.one/list.json` 中该插件的条目对应）。
- 校验项（全部通过）：两份 `.plugin` 各解析出 24 条复写、正则序列与顺序一致；24 条正则全部可编译；
  24×24 交叉匹配中每条只命中自己（唯一例外是 `g-acs` 的搜索接口，原文就是**一条 URL 两条动作**）；
  15 段 jq 程序全部可编译（jq 1.7.1），且在两个文件中逐字节一致；`[MitM]` 与原文相同；
  文件中没有任何非注释行引用 kelee.one。
- 存疑项：无签名探测只能证明「网关路由表里没有这个名字」，真机签名请求能否路由**未验证**。
