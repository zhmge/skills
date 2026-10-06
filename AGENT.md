# AGENT.md

本文件面向在此仓库中协作的 AI agent，说明这个仓库的组织方式与协作约定。人类读者请优先看 [`README.md`](README.md)。

## 本仓库是什么

这是一个 **skill 索引仓库**（`zhmge/skills`），只收录索引文档，不存放任何 skill 实体。

- **每个 skill 独立成一个公开仓库**，命名为 `zhmge/skill-<skill-name>`。
- 仓库**刻意不使用 git submodule**：索引的定位是“按需取用”，不是“一键拉取全部”。取用某个 skill 时，直接 clone 它对应的独立仓库。
- 因此各 skill 子目录全部由 `.gitignore` 逐一忽略。**新增 skill 目录时必须同步补一条忽略规则**，否则 git 会把子目录记成空的 gitlink（embedded repository），别人 clone 本仓库只会看到一个空文件夹。
- `README.md` 是**纯索引表**：只保留索引表格与 clone 命令列表，不写各 skill 的内联详情小节，也不拆 `docs/` 详情页。某个 skill 的详情留在它自己的仓库里。

出处：`skill-github-publish/SKILL.md`。

## 目录布局

```text
skills/
|-- README.md            索引表：已发布 skill 一览 + clone 命令
|-- AGENT.md             本文件
|-- LICENSE              索引仓库自身的许可（MIT）
|-- .gitignore           逐一忽略各 skill 子目录
|-- .gitattributes       统一行尾为 LF
|-- .workbuddy/          本地工作区记忆，不随索引公开
`-- <skill-name>/        各 skill，均为独立仓库，被 .gitignore 忽略
```

`.workbuddy/memory/` 存放本仓库的工作日志与长期约定，属于项目自身数据，不对外公开，也不随索引仓库提交。

**skill 目录自身的结构**（`SKILL.md` 与可选的 `scripts/`、`references/`、`assets/` 如何组织）由 `skill-creator-generic` 定义，以该 skill 为准，本文件不复述。任何新建或改写 skill 的动作，都应先读它。

出处：`skill-creator-generic/SKILL.md`。

## SKILL.md 约定

每个 skill 的入口是它目录下的 `SKILL.md`。本仓库对入口的要求：

- frontmatter 至少包含 `name` 与 `description` 两个字段；`name` 与所在目录名一致。市场安装的 `find-skills__skillhub` 是已知例外（其 `name` 为 `find-skills`），不属于原创 skill，不参与本仓库约定。
- `description` 需说明「做什么」与「何时适用」，并给出**适用 / 不适用边界**，避免误触发。
- 正文语言可中可英，但同一篇内保持一致。

其余关于 frontmatter 字段、章节组织、渐进披露、命名规范的细则，一律以 `skill-creator-generic` 为准。

出处：`skill-creator-generic/SKILL.md`、各 skill 的 `SKILL.md`。

## 发布与 README 约定

把一个本地 skill 发布为独立 GitHub 仓库、并同步本索引仓库的完整流程，见 `skill-github-publish/SKILL.md`。协作时须注意的几条：

- **停机点不可越过。** 本地建仓与提交完成后必须停下，向用户展示仓库名与可见性、提交摘要、索引 `README.md` 的变更；**只有用户明确同意后才能推送**。含糊的「继续」不算同意。`gh repo create` 与 `git push` 不得写进同一条复合命令。
- **git 身份用仓库级配置**：`user.name=zhmge`、`user.email=148024522+zhmge@users.noreply.github.com`。**禁止使用 `--global`** —— 本机全局身份是另一个账号。
- **`.gitignore` 必须先于任何 `git add` 写入**，否则规则不生效。
- **不自动 commit。** 改动完成后只产出 commit message 供用户参考。
- 撰写或修改任何 skill 仓库的 `README.md` 前，先读 `skill-github-publish/references/readme-style.md`。其中的写作基线（无人称、中文弯引号、命名式标题、与 `SKILL.md` 不复述等）只约束各 skill 仓库自己的 `README.md`，不约束 `SKILL.md`。

出处：`skill-github-publish/SKILL.md`、`skill-github-publish/references/readme-style.md`。

## 环境创建规范

**skill 仓库里不放环境。** 仓库只保留文本与脚本，任何 Python 环境（venv、conda env、模型缓存）都在仓库之外。

- **重型环境由 conda 集中管理**，环境目录在 conda 根目录的 `envs/` 下，命名 `skill-<skill-name>`（与所服务的 skill 同名）。
  **创建或复用任何环境前，先读环境根目录的 `AGENT.md`**（即 `<conda根>/envs/AGENT.md`）——那里是环境清单、模型路径、环境变量约定与创建规则的唯一权威。新建或变更环境后要回去更新它。
- **skill 文档只声明「需要哪个环境」，不写本地路径。** 命令里的解释器统一用 `<python>` 占位符，环境清单与路径由环境根的 `AGENT.md` 承载。这样 skill 仓库可公开、可移植，换机器只需改环境根那份登记表。
- **脚本里不得写死本机路径。** 需要外部路径时从环境变量读（如 `FW_MODEL_DIR`），未设时**报错并给出指引**，不要静默回退到某个写死的默认位置。
- **`.gitignore` 必须覆盖环境目录**：`.venv*/`、`venv*/`、`envs/`、`.conda/`。漏配的后果不是「多提交几个文件」，而是把几百 MB 的二进制与**本机绝对路径**一起公开，并可能被记成空 gitlink。
- **不要用 `conda create --clone base`**：克隆时 conda 会把目标环境目录下的路径误判为 base 内文件去删除，触发安全机制拦截，**必然失败**并留下一个无 `python.exe`、无 `conda-meta` 的坏残壳。要复用 base 的包请显式安装。
- **不要往 base 里装重型依赖**，也**不要用 venv 装重型依赖**（venv 不可迁移，`Scripts/*.exe` 写死绝对路径）。

上述规则的执行保障在 `skill-github-publish`：`publish_check.py` 会把 `.venv*` / `venv*` / `envs` / `.conda` 排除出扫描，并在它们被 git 跟踪时判为失败。

出处：环境根目录的 `AGENT.md`、`skill-github-publish/scripts/publish_check.py`。

## 环境注意（Windows / 本机）

- 本机为 Windows + Git Bash，**bash 工作目录不可靠**：跨命令引用文件一律用绝对路径；路径含空格或中文必须加引号。
- `git init` 一律带 `-b main`（本机未设置 `init.defaultBranch`）。
- **GitHub 操作必须显式走代理**：`-c http.proxy=http://127.0.0.1:7897 -c https.proxy=http://127.0.0.1:7897`。报 `schannel: server closed abruptly` 时追加 `-c http.sslBackend=openssl`。GitHub 直连不通，**不要绕开代理重试**。该代理仅用于访问国外站点，其余网络访问走默认环境。
- 校验类脚本用托管 Python：`C:\Users\zhmge\.workbuddy\binaries\python\versions\3.13.12\python.exe`。若脚本需要 pyyaml，改用 `C:\Users\zhmge\anaconda3\python.exe`。

出处：`skill-github-publish/SKILL.md`、`.workbuddy/memory/MEMORY.md`。
