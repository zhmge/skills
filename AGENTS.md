# AGENTS.md

本文件面向在此仓库中协作的 AI agent，说明这个仓库的组织方式与协作约定。人类读者请优先看 [`README.md`](README.md)。

**本文件自足**：不依赖任何仓库之外的上下文即可执行。缺少仓库外的东西怎么办，见下面「外部依赖」一节。

## 一、仓库自身约束

### 本仓库是什么

这是一个 **skill 索引仓库**（`zhmge/skills`），只收录索引文档，不存放任何 skill 实体。

- **每个 skill 独立成一个公开仓库**，命名为 `zhmge/skill-<skill-name>`。GitHub 地址见 [`README.md`](README.md)。
- 仓库**刻意不使用 git submodule**：索引的定位是「按需取用」，不是「一键拉取全部」。取用某个 skill 时，直接 clone 它对应的独立仓库。
- 因此各 skill 子目录全部由 `.gitignore` 逐一忽略。**新增 skill 目录时必须同步补一条忽略规则**，否则 git 会把子目录记成空的 gitlink（embedded repository），别人 clone 本仓库只会看到一个空文件夹。
- `README.md` 是**纯索引表**：只保留索引表格与 clone 命令列表，不写各 skill 的内联详情小节，也不拆 `docs/` 详情页。某个 skill 的详情留在它自己的仓库里。

### 目录布局

```text
skills/
|-- README.md            索引表：已发布 skill 一览 + clone 命令
|-- AGENTS.md            本文件
|-- LICENSE              索引仓库自身的许可（MIT）
|-- .gitignore           逐一忽略各 skill 子目录
|-- .gitattributes       统一行尾为 LF
`-- .workbuddy/          本地工作区记忆，被忽略、不随索引公开
```

（各 skill 目录**不在本 clone 里**，见「外部依赖」第 3 条。）

skill 目录自身的结构（`SKILL.md` 与可选的 `scripts/`、`references/`、`assets/` 如何组织）不由本文件定义，以 `skill-creator-generic` 为准（见「外部依赖」第 1 条）。任何新建或改写 skill 的动作，都应以它为先。

### SKILL.md 约定

每个 skill 的入口是它目录下的 `SKILL.md`。本仓库对入口的要求：

- frontmatter 至少包含 `name` 与 `description` 两个字段；`name` 与所在目录名一致。市场安装的 `find-skills__skillhub` 是已知例外（其 `name` 为 `find-skills`），不属于原创 skill，不参与本仓库约定。
- `description` 需说明「做什么」与「何时适用」，并给出**适用 / 不适用边界**，避免误触发。
- 正文语言可中可英，但同一篇内保持一致。

其余关于 frontmatter 字段、章节组织、渐进披露、命名规范的细则，一律以 `skill-creator-generic` 为准。

### 发布与 README 约定

把一个本地 skill 发布为独立 GitHub 仓库、并同步本索引仓库的完整流程，由 `skill-github-publish` 定义（见「外部依赖」第 2 条）。协作时须注意的几条：

- **停机点不可越过。** 本地建仓与提交完成后必须停下，向用户展示仓库名与可见性、提交摘要、索引 `README.md` 的变更；**只有用户明确同意后才能推送**。含糊的「继续」不算同意。`gh repo create` 与 `git push` 不得写进同一条复合命令。
- **git 身份用仓库级配置**（`user.name`、`user.email` 由使用者自己提供）。**禁止使用 `--global`** —— 全局身份往往属于另一个账号。
- **`.gitignore` 必须先于任何 `git add` 写入**，否则规则不生效。
- **不自动 commit。** 改动完成后只产出 commit message 供用户参考。
- 撰写或修改任何 skill 仓库的 `README.md` 前，先读 `skill-github-publish` 的 `references/readme-style.md`。其中的写作基线（无人称、中文弯引号、命名式标题、与 `SKILL.md` 不复述等）只约束各 skill 仓库自己的 `README.md`，不约束 `SKILL.md`。

### 环境创建规范

**skill 仓库里不放环境。** 仓库只保留文本与脚本，任何 Python 环境（venv、conda env、模型缓存）都在仓库之外。

- **重型环境集中管理**，每个 skill 对应一个同名环境 `skill-<skill-name>`，位置由使用者的 Python 环境根决定。**创建或复用任何环境前，先读该环境根下的环境登记表**（详见「外部依赖」第 4 条）——那里是环境清单、模型路径、环境变量约定与创建规则的唯一权威。新建或变更环境后要回去更新它。
- **skill 文档只声明「需要哪个环境」，不写本地路径。** 命令里的解释器统一用 `<python>` 占位符，环境的真实位置由上面那张登记表承载。这样 skill 仓库可公开、可移植，换机器只需改登记表那一份。
- **脚本里不得写死本机路径。** 需要外部路径时从环境变量读（如 `FW_MODEL_DIR`），未设时**报错并给出指引**，不要静默回退到某个写死的默认位置。
- **`.gitignore` 必须覆盖环境目录**：`.venv*/`、`venv*/`、`envs/`、`.conda/`。漏配的后果不是「多提交几个文件」，而是把几百 MB 的二进制与**本机绝对路径**一起公开，并可能被记成空 gitlink。
- **不要用 `conda create --clone base`**：克隆时 conda 会把目标环境目录下的路径误判为 base 内文件去删除，触发安全机制拦截，**必然失败**并留下一个无 `python.exe`、无 `conda-meta` 的坏残壳。要复用 base 的包请显式安装。
- **不要往 base 里装重型依赖**，也**不要用 venv 装重型依赖**（venv 不可迁移，`Scripts/*.exe` 写死绝对路径）。

上述规则的检查脚本随 `skill-github-publish` 提供（`publish_check.py`）：它会把 `.venv*` / `venv*` / `envs` / `.conda` 排除出扫描，并在它们被 git 跟踪时判为失败。

## 二、外部依赖（本仓库不含以下内容）

本文件的若干约定与能力来自仓库之外。**只声明逻辑名与缺失后果，不声明它们的位置。**

| 依赖 | 它是什么 | 缺失时的后果 | 获取指引 |
| --- | --- | --- | --- |
| `skill-creator-generic` | skill 目录结构、`SKILL.md` frontmatter、渐进披露规范的权威出处 | 新建或改写 skill 时**无法核对规范**，只能照本文件的概要执行，容易不合规范 | 独立 skill 仓库；**尚未公开发布**，无公开地址。缺失时尚可工作，但应告知使用者「未经权威核对」 |
| `skill-github-publish`（含 `references/readme-style.md`） | 发布流程本身 + README 写作基线 | **发布动作无法完成**；写 skill 的 `README.md` 时无基线可依 | 独立 skill 仓库；**尚未公开发布**。缺失时**停止发布流程**并说明缺什么，不要即兴发挥发布步骤 |
| 各 skill 独立仓库 | skill 的实体内容（仅存在于索引之外） | clone 本仓库后看不到任何 `<skill-name>/` 目录 —— **这是设计如此，不是缺失** | 按 [`README.md`](README.md) 索引表里的 clone 命令单独取用 |
| Python 环境与其登记表 |  `<python>` 占位符所指的真实解释器，以及环境清单 / 模型路径 / 环境变量约定的登记表 | 不知道该创建或复用哪个环境；`<python>` 无法解析 | 由使用者在本机维护，**本仓库不提供也不写其位置**。skills 文档里只需声明「需要哪个环境」 |
| 重型模型缓存 | ASR / OCR 等推理权重，经环境变量（如 `FW_MODEL_DIR`）传入 | 相关 skill 跑不起来。**脚本必须报错并给指引，禁止静默回退到写死的默认路径** | 见各 skill 自己仓库的 `SKILL.md` |

## 三、单独 clone 时的行为

- 本仓库 clone 后只有 `README.md` / `LICENSE` / 本文件 / git 配置，**不含任何 skill 实体**。这是设计如此，**不得判定为损坏或缺失**。
- 缺少上表任一依赖**不得报错或阻塞**：核对类依赖缺失只影响「能否核对」，发布类依赖缺失则**停下并说明缺什么**，两种情况都不例外空转。
- 在此 clone 里工作时，**不得假定任何仓库外的私有上下文存在**（本机记忆、其他项目的控制面都不在本 clone 内）。需要的信息若不在本文件与 `README.md` 里，就**向使用者问**，不要猜、不要自行去找。
- 命令里出现 `<python>`、`<conda根>` 这类占位符时，先解析上表第 4 条；解析不了就**停止并询问**，不要用系统默认解释器硬上。

## 四、导航

- 找某个 skill → [`README.md`](README.md)：索引表 + 每个 skill 的 clone 命令
- 单个 skill 的用法 → 它的独立仓库，入口是 `SKILL.md`
- 许可 → `LICENSE`（本索引仓库 MIT）；各 skill 有自己的 LICENSE
- 新增 skill 目录 → 同步改 `.gitignore`（本仓库「本仓库是什么」一节）
- 行尾 → `.gitattributes`（统一 LF）
- 本文件中各条约定的出处 → 见「外部依赖」第 1、2 条
