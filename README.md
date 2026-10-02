# Skills

AI agent skill 索引 —— 每个 skill 独立成库，本仓库只做汇总。

| Skill | 一句话说明 | 仓库 |
|---|---|---|
| **bilibili-subtitle-fetch** | 把 B 站视频的字幕抓取为 `.srt` 文件，多分P 视频与合集（ugc_season）都能处理；每份 `.srt` 首行带该集来源链接 | [zhmge/skill-bilibili-subtitle-fetch](https://github.com/zhmge/skill-bilibili-subtitle-fetch) |
| **srt-course-outline** | 把 B 站课程字幕（`.srt`）整理为飞书文档中的课程大纲框架，每个标题带精准空降时间链接（基址取自 `.srt` 首行） | [zhmge/skill-srt-course-outline](https://github.com/zhmge/skill-srt-course-outline) |
| **web-to-epub** | 把网页文章或网页书直接转换为内容保真的 EPUB 3 电子书，附确定性打包与本地审计脚本 | [zhmge/skill-web-to-epub](https://github.com/zhmge/skill-web-to-epub) |

更多 skill 陆续整理中。

## 使用方式

本仓库是**纯索引**，不含任何 skill 内容。取用某个 skill 时，直接 clone 它的独立仓库：

```bash
git clone https://github.com/zhmge/skill-bilibili-subtitle-fetch.git
git clone https://github.com/zhmge/skill-srt-course-outline.git
git clone https://github.com/zhmge/skill-web-to-epub.git
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
每份 `.srt` 的第一行是该集/该分P 的来源链接，第二行是空行，之后才是字幕块。
重跑同一条命令会跳过已完成的、只补失败的。

已知限制：只在 B 站确实有字幕时有效；AI 字幕（`ai-zh`）由音频自动转写，**含识别错误**，
不要当逐字稿引用；串行 + 3 秒间隔是刻意设计，用于避开风控，调快会导致限流。

**搭配使用**：抓到的 `.srt` 可交给 [srt-course-outline](#srt-course-outline) 整理为课程大纲。

---

## srt-course-outline

把 B 站课程字幕（`.srt`）整理为飞书文档中的课程大纲框架 —— 每个标题带 B 站精准空降时间链接。

- **仓库**：<https://github.com/zhmge/skill-srt-course-outline>
- **类型**：Agent skill，需配合飞书连接器使用
- **依赖**：Python 3（`parse_srt.py` 零第三方依赖）；写入文档依赖飞书连接器提供的 `lark-doc` skill

### 用法

```bash
python3 scripts/parse_srt.py "<字幕1.srt>" ["<字幕2.srt>" ...]
```

先把 `.srt` 解析为结构化 JSON（来源链接、BV 号、字幕块列表），再由 agent 按 `SKILL.md`
的工作流概括 h1 课程主题、切分 h2／h3、填充内容与时间链接，最终追加到目标飞书文档末尾。

一次可传入多个 `.srt`，各自生成一个框架块，按顺序依次追加。

### 产物

目标文档末尾追加的大纲框架：h1 课程主题 → h2 小节（必要时 h3 细分）。

- 每个标题标注 B 站精准空降时间链接，显示 `HH:MM:SS`，点击秒级跳转
- 标题下填充导学概括与简单知识点：要点用 `- ` 短语，完整句作为要点下方的顶格段落
- 只有一层要点，不使用嵌套子列表；需要看视频理解的部分留白

### 前置条件

**单独 clone 本仓库无法端到端运行。** 写入飞书文档依赖飞书连接器提供的 `lark-doc` skill，
且需要对目标文档有编辑权限。缺少该连接器时，`parse_srt.py` 仍可独立把 `.srt` 解析为 JSON。

### 搭配使用

字幕来源由 [bilibili-subtitle-fetch](#bilibili-subtitle-fetch) 提供 —— 先用它把 B 站视频或
合集的字幕抓成 `.srt`（首行即该集来源链接），再交给本 skill 整理成大纲。

---

## web-to-epub

把网页文章或网页书（一本一章一个页面的在线阅读站）直接转换为**内容保真**的 EPUB 3 电子书 —— 自适应排版、语义化目录、内嵌离线资源、内部链接校验。

- **仓库**：<https://github.com/zhmge/skill-web-to-epub>
- **类型**：Agent skill，兼带两个可独立调用的 CLI 脚本（`scripts/`）
- **依赖**：Python 3.10 及以上，仅标准库 —— 不需要第三方库，也不需要先把页面存成离线 HTML

### 用法

把源页面链接与目标范围交给 agent，agent 按 `SKILL.md` 的工作流执行：确认影响结果的取舍 → 建立规范化源目录 → 打包 → 审计 → 交付。

源目录已按约定准备好时，两个脚本也可单独调用：

```bash
python scripts/epub_package.py SOURCE_DIR OUTPUT.epub [--no-ncx]
python scripts/epub_audit.py OUTPUT.epub [--source-dir SOURCE_DIR] [--report EPUB_AUDIT.txt] [--json]
```

安装为 agent skill 时，整个目录放入 skills 目录即可；`SKILL.md` 是入口。

### 产物

一个 EPUB 3 文件（`mimetype` 恒为 ZIP 首条且不压缩），外加一份审计报告：包结构与 ZIP 条目、manifest 与 spine、XML 合法性、内部文件与片段锚点、重复 ID、离线资源依赖、CSS 引用、注释引用配对，以及源文件与成书的逐字节比对计数。审计脚本发现错误时退出码为 1，可直接用于流水线判定。报告区分「EPUBCheck 校验」与「本地结构与内容核对」两种口径。

### 开工前会先确认的环节

章节顺序与非正文内容是否收录 · 注释与批注的归属和落点 · 可重排还是固定版式 · 源站导航是否保留 · 是否执行官方 EPUBCheck。

### 已知限制

- 可重排 EPUB 无法保证与网页**像素级一致**；需要固定尺寸时应在开工前说明
- 本地审计不等于 EPUBCheck 通过：结构项（`mimetype` 顺序与压缩方式、ZIP 条目安全）由审计覆盖，规范完整性与阅读系统兼容性仍需官方 EPUBCheck
- 注释仅以视觉坐标或空锚点定位时，归属需人工确认后再落笔
- 未执行 EPUBCheck 时，报告会明确标注口径，不声称成书通过了结构认证

---

## 不使用 submodule 的原因

本仓库**刻意不**通过 git submodule 串联各个 skill。submodule 会带来“一键拉取全部”的效果，
但代价是：clone 需要额外加 `--recurse-submodules`，在 GitHub 网页上点进子目录只能看到一个
commit 指针、看不到实际文件。对一个以“按需取用”为目的的索引仓库来说，直接给链接更简单也更透明。

## License

本索引仓库的文档与配置采用 [MIT](LICENSE) 许可。各 skill 仓库有各自的 LICENSE。
