# Skills

AI agent skill 索引 —— 每个 skill 独立成库，本仓库只做汇总。

| Skill | 一句话说明 | 仓库 |
|---|---|---|
| **bilibili-subtitle-fetch** | 把 B 站视频的字幕抓取为 `.srt` 文件，多分P 视频与合集（ugc_season）都能处理；每份 `.srt` 首行带该集来源链接 | [zhmge/skill-bilibili-subtitle-fetch](https://github.com/zhmge/skill-bilibili-subtitle-fetch) |
| **srt-course-outline** | 把 B 站课程字幕（`.srt`）整理为飞书文档中的课程大纲框架，每个标题带精准空降时间链接（基址取自 `.srt` 首行） | [zhmge/skill-srt-course-outline](https://github.com/zhmge/skill-srt-course-outline) |
| **web-to-epub** | 把网页文章或网页书直接转换为内容保真的 EPUB 3 电子书，附确定性打包与本地审计脚本 | [zhmge/skill-web-to-epub](https://github.com/zhmge/skill-web-to-epub) |
| **excel-table-export** | 把 Unity 项目的 Excel 配置表安全导出为运行时 JSON：预校验 → Unity 导出 → 产物核对 → 提交四件套，附只读预校验脚本 | [zhmge/skill-excel-table-export](https://github.com/zhmge/skill-excel-table-export) |

更多 skill 陆续整理中。

## 使用方式

本仓库是**纯索引**，不含任何 skill 内容。取用某个 skill 时，直接 clone 它的独立仓库：

```bash
git clone https://github.com/zhmge/skill-bilibili-subtitle-fetch.git
git clone https://github.com/zhmge/skill-srt-course-outline.git
git clone https://github.com/zhmge/skill-web-to-epub.git
git clone https://github.com/zhmge/skill-excel-table-export.git
```

整个目录放入 agent skills 目录即可，`SKILL.md` 是入口。

## License

本索引仓库的文档与配置采用 [MIT](LICENSE) 许可。各 skill 仓库有各自的 LICENSE。
