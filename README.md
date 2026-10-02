# Skills

我整理的 AI agent skill 合集 —— 每个 skill 独立成库，这里是索引。

## 索引

| Skill | 说明 | 仓库 | 状态 |
|---|---|---|---|
| **bilibili-subtitle-fetch** | 把 B 站视频的字幕抓成 `.srt` 文件，多分P 视频与合集（ugc_season）通吃 | [zhmge/skill-bilibili-subtitle-fetch](https://github.com/zhmge/skill-bilibili-subtitle-fetch) | 已发布 |

更多 skill 陆续整理中。

## 使用方式

这个仓库是**纯索引**，不含任何 skill 内容。取用某个 skill 时，直接 clone 它的独立仓库：

```bash
git clone https://github.com/zhmge/skill-bilibili-subtitle-fetch.git
```

然后把整个目录放进你的 agent skills 目录即可，`SKILL.md` 是入口。

## 详情

每个 skill 的详细说明（用途、前置条件、用法、限制）放在 [`docs/`](docs/) 下：

- [bilibili-subtitle-fetch](docs/bilibili-subtitle-fetch.md)

## 为什么不用 submodule

本仓库**刻意不**通过 git submodule 串联各个 skill。submodule 会带来"一键拉取全部"
的效果，但代价是：clone 需要额外加 `--recurse-submodules`，在 GitHub 网页上点进
子目录只能看到一个 commit 指针、看不到实际文件。对一个以"按需取用"为目的的索引
仓库来说，直接给链接更简单也更透明。

## License

本索引仓库的文档与配置采用 [MIT](LICENSE) 许可。各 skill 仓库有各自的 LICENSE。
