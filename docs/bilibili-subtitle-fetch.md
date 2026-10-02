# bilibili-subtitle-fetch

> 把 B 站视频的字幕抓成 `.srt` 文件 —— 多分P 视频和**合集（ugc_season）**通吃。

| | |
|---|---|
| **仓库** | [zhmge/skill-bilibili-subtitle-fetch](https://github.com/zhmge/skill-bilibili-subtitle-fetch) |
| **状态** | 已发布 |
| **类型** | Agent skill（也可当独立 CLI 脚本用） |
| **依赖** | Node.js（零第三方依赖） |

## 用途

给一个 B 站视频地址或 BV 号，把它的字幕抓下来落成 `.srt` 文件。

支持两种目标形态，脚本会自己从 `view` 响应里分辨：

- **多分P 视频** —— 一个 BV 号底下多个分P，产物平铺；
- **合集 / ugc_season** —— 一系列独立视频（比如一整套课程），产物按章节建子目录。

## 前置条件：需要登录态

必须提供 B 站登录后的 `SESSDATA` cookie 值。

原因是 B 站的 `player/v2` 接口对匿名请求**固定返回空字幕列表**，并且和"这个视频真的
没有字幕"完全无法区分（都是 `code=0` + `subtitles: []`）。所以脚本在缺凭据时直接
报错退出（退出码 3），一个请求都不发 —— 避免把"没登录"误判成"没字幕"。

获取方式：浏览器登录 B 站 → `F12` → **Application** → **Cookies** →
`https://www.bilibili.com` → `SESSDATA` → 复制 Value。

它是 **HttpOnly** cookie，页面控制台读不到；浏览器磁盘上的 cookie 库也是加密的
（Chrome/Edge 的 App-Bound Encryption）。**只能手动复制。**

**凭据只走环境变量 `BILI_SESSDATA`，不落盘。** 脚本也只在目标是 `api.bilibili.com`
时才附带它，字幕 CDN 的请求不带凭据。

## 用法

```bash
BILI_SESSDATA="<值>" node scripts/fetch_subtitles.js "<BV号或视频URL>" --out "<输出目录>"
```

常用选项：

| 选项 | 说明 |
|---|---|
| `--out DIR` | 输出目录（**必填**） |
| `--p N` | 只取第 N 个分P |
| `--from N --to N` | 取一个范围 |
| `--all` | 取全部分P（URL 带 `?p=N` 时用它覆盖） |
| `--interval MS` | 请求间隔毫秒（默认 3000，不建议调低） |
| `--scan --sample N` | 只探测字幕可用性、不下载 |
| `--force` | 忽略断点记录重跑 |

几百个目标时，先跑探针再决定是否全量：

```bash
BILI_SESSDATA="<值>" node scripts/fetch_subtitles.js "<URL>" --out "<目录>" --scan --sample 5
```

## 输出

| 文件 | 说明 |
|---|---|
| `<视频标题>_P<n>_<分P名>.srt` | 多分P 视频：每分P 一个文件 |
| `<NN_章节名>/<NN_标题>.srt` | 合集：一章节一子目录，一集一文件 |
| `_index.md` | 全部条目索引表 |
| `_manifest.json` | 断点续跑台账 |

只需要重跑同一条命令，已完成的会跳过、失败的会重试。

## 设计要点

这个 skill 的价值主要在两处「不显然」的地方：

**1. 合集不是多分P。** 合集里每一集都是独立视频（各自的 `bvid`/`cid`/`aid`）。
合集成员的 `pages` 数组**恒为 1 个元素** —— 只读 `pages` 的脚本会"成功地"只抓到
1 集、退出码还是 0，没有任何报错。

**2. `player/v2` 会随机投毒。** 该接口给非浏览器客户端会随机返回**别的视频的字幕**，
响应结构完全正常（`code=0`、真实 `auth_key`、合理 `lan`），只有内容是错的，实测占比
约 50%–75%。脚本用「字幕 CDN 路径是否以 `<aid><cid>` 开头」来校验，不匹配就重试
（最多 8 次），且校验在 `player/v2` 响应上完成，所以被投毒的响应不会触发 CDN 下载。

因此：**某个分P 失败通常是暂时性的**，重跑往往就过；把失败当成"没有字幕"是错的。

## 已知限制

- 只在 B 站确实有字幕时有效
- AI 字幕（`ai-zh`）由音频自动转写，**含识别错误**，不要当逐字稿引用
- 多人对话、强剪辑、音乐为主的视频，AI 字幕质量差
- 串行 + 3 秒间隔是刻意设计，用于避开风控

## 合规提示

抓取字幕用于个人学习是一回事；批量抓取或再分发涉及 B 站的服务条款，请自行判断。

## 相关文档

在[独立仓库](https://github.com/zhmge/skill-bilibili-subtitle-fetch)内：

- `SKILL.md` —— 给 AI agent 的完整指令入口
- `references/api-behavior.md` —— 接口行为、投毒机制、校验原理
- `references/troubleshooting.md` —— 排错手册：状态区分、风控、断点续跑
