# Skills

AI agent skill 索引 —— 每个 skill 独立成库，本仓库只做汇总。

| Skill | 一句话说明 | 仓库 |
|---|---|---|
| **bilibili-subtitle-fetch** | 把 B 站视频的字幕抓取为 `.srt` 文件，多分P 视频与合集（ugc_season）都能处理 | [zhmge/skill-bilibili-subtitle-fetch](https://github.com/zhmge/skill-bilibili-subtitle-fetch) |

更多 skill 陆续整理中。

## 使用方式

本仓库是**纯索引**，不含任何 skill 内容。取用某个 skill 时，直接 clone 它的独立仓库：

```bash
git clone https://github.com/zhmge/skill-bilibili-subtitle-fetch.git
```

整个目录放入 agent skills 目录即可，`SKILL.md` 是入口。

---

## bilibili-subtitle-fetch

把 B 站视频的字幕抓取为 `.srt` 文件 —— **多分P 视频**与**合集（ugc_season）**都能处理。

- **仓库**：<https://github.com/zhmge/skill-bilibili-subtitle-fetch>
- **类型**：Agent skill，也可作为独立 CLI 脚本使用
- **依赖**：Node.js（零第三方依赖）

### 用法

```bash
BILI_SESSDATA="<值>" node scripts/fetch_subtitles.js "<BV号或视频URL>" --out "<输出目录>"
```

常用选项：`--p N` 指定分P · `--from N --to N` 范围 · `--all` 全部分P ·
`--scan --sample N` 只探测不下载 · `--force` 忽略断点记录重跑。

几百个目标时，先跑探针再决定是否全量：

```bash
BILI_SESSDATA="<值>" node scripts/fetch_subtitles.js "<URL>" --out "<目录>" --scan --sample 5
```

### 需要登录态

必须提供 B 站登录后的 `SESSDATA` cookie 值。原因：`player/v2` 接口对匿名请求
**固定返回空字幕列表**，与“视频真的没字幕”完全无法区分（都是 `code=0` + `subtitles: []`）。
所以缺凭据时脚本直接报错退出，一个请求都不发。

获取方式：登录 B 站 → `F12` → **Application** → **Cookies** → `https://www.bilibili.com`
→ 复制 `SESSDATA` 的 Value。它是 HttpOnly cookie，页面控制台读不到，只能手动复制。

**凭据只走环境变量 `BILI_SESSDATA`，不落盘；且只发给 `api.bilibili.com`，字幕 CDN 不带凭据。**

### 两个反直觉之处

**合集不是多分P。** 合集里每一集都是独立视频（各自的 `bvid`/`cid`/`aid`），
其 `pages` 数组**恒为 1 个元素** —— 只读 `pages` 的脚本最终只会抓到 1 集，
且退出码为 0、没有任何报错。

**接口会随机“投毒”。** 指 `player/v2` 给非浏览器客户端返回的可能是**他人视频**的字幕，
结构完全正常（`code=0`、真实 `auth_key`、合理 `lan`），只有内容是错的，实测占比
约 50%–75%。脚本用“字幕 CDN 路径是否以 `<aid><cid>` 开头”校验并重试（最多 8 次），
校验在 `player/v2` 响应上完成，所以被投毒的响应不会触发 CDN 下载。

结论：**某个分P 失败通常是暂时性的，重跑同一条命令通常即可成功。**

### 输出与限制

产物：`.srt` 文件（合集按章节建子目录）、`_index.md` 索引表、`_manifest.json` 断点台账。
重跑同一条命令会跳过已完成的、只补失败的。

已知限制：只在 B 站确实有字幕时有效；AI 字幕（`ai-zh`）由音频自动转写，**含识别错误**，
不要当逐字稿引用；串行 + 3 秒间隔是刻意设计，用于避开风控，调快会导致限流。

---

## 为什么不用 submodule

本仓库**刻意不**通过 git submodule 串联各个 skill。submodule 会带来“一键拉取全部”的效果，
但代价是：clone 需要额外加 `--recurse-submodules`，在 GitHub 网页上点进子目录只能看到一个
commit 指针、看不到实际文件。对一个以“按需取用”为目的的索引仓库来说，直接给链接更简单也更透明。

## License

本索引仓库的文档与配置采用 [MIT](LICENSE) 许可。各 skill 仓库有各自的 LICENSE。
