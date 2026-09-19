## 安装clauld code

#### 方式一：使用 Windows 包管理器 WinGet（推荐）

```powershell
winget install Anthropic.ClaudeCode
```

#### 方式二：手动下载并安装

https://downloads.claude.ai/claude-code-releases/2.1.268/win32-x64/claude.exe

### 方式三：npm 安装（已废弃，不推荐）

需 Node.js 18+，官方已停止维护此方式：

```bash
npm install -g @anthropic-ai/claude-code
```

### 验证安装

```bash
claude --version    # 输出版本号即成功
claude doctor       # 诊断配置问题
```

### 跳过登录验证

编辑 `~/.claude.json`，设置：

```json
{ "hasCompletedOnboarding": true }
```

### 第三方/国内模型接入

#### 方式一：

在 `~/.claude/settings.json`（Windows: `C:\Users\<用户名>\.claude\settings.json`）中配置：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "你的API密钥",
    "ANTHROPIC_BASE_URL": "兼容接口地址",
    "ANTHROPIC_MODEL": "模型名称"
  }
}
```

#### 方式二：

使用cc switch

下载地址：

- https://github.com/farion1231/cc-switch/releases

- https://www.ccswitch.io/zh/download

---

----

## 如何用CLAUDE.md(类似codex的rules)

#### 加载顺序

Claude Code 启动时会自下而上合并所有找到的规则文件，**越靠近项目、越具体的层级优先级越高**：

```textile
组织级 /etc/claude-code/CLAUDE.md        ← 公司统一发，个人改不了（最高）
项目本地 ./CLAUDE.local.md               ← 你自己的覆盖，必须 .gitignore
项目级 ./CLAUDE.md                       ← 团队共享，提交 Git
用户级 ~/.claude/CLAUDE.md               ← 你的全局偏好（最低）
```

同一层内还有 `.claude/rules/*.md`，带 `paths` 元数据的文件只在匹配到对应文件时才注入上下文。

所以：**全局习惯写 `~/.claude/CLAUDE.md`，项目规范写 `./CLAUDE.md`，按文件类型分的细则丢进 `.claude/rules/`。**

#### 一、全局规则：`~/.claude/CLAUDE.md`

这是对应 Cursor User Rules 的那一层。先建目录再建文件：

```bash
mkdir -p ~/.claude
touch ~/.claude/CLAUDE.md
```

内容只放**跨项目通用**的东西，建议控制在 80 行以内（超过 200 行后半段容易被忽略）。一个可直接抄的模板：

markdown

编辑

```markdown
# 全局偏好

## 关于我
后端开发，主要用 Claude Code 写 Go/Python 服务和脚本。

## 沟通方式
- 默认中文回复，代码、命令、变量名用英文
- 结论先行，不要铺垫"当然可以""这是个好问题"
- 需求模糊时先给最合理的方案，再问要不要调整

## 红线（即使 auto-accept 也必须先问我）
- 删除文件/目录、改写 git 历史
- 修改 .env、密钥、CI/CD 配置
- 数据库 schema 变更、数据迁移
- git push --force、rebase、reset --hard
- npm/pip 全局安装、修改系统配置

## 工程纪律
- 改动后主动跑验证命令，不要只改不验
- 不要为了跑通而注释掉报错或加 TODO 绕过
- 密钥不进代码、不进 commit、不进日志
- 大改动先进 Plan Mode 出方案，我确认后再动手
```

> 注意：CLAUDE.md 是**软约束**，模型不一定 100% 遵守。像"禁止 rm -rf""禁止读 ~/.ssh"这类硬性安全要求，应该写在 `~/.claude/settings.json `的` permissions` 里配 deny 规则，或者用 Hooks，别指望写在 md 里就万无一失。

### 二、项目规则：`./CLAUDE.md`

最快的方式是让 AI 自己写初稿：

```bash
cd /path/to/your/projectclaude> /init
```

`/init` 会扫描 package.json、Makefile、README 等，生成包含构建命令、测试命令、目录结构的初稿，你再手动删改。**一定要做减法**——把 AI 能从代码里自己推断出来的东西全删掉（比如"用 Java 17"这种 pom.xml 里有的），只留它猜不到的：

- 非标准的构建/测试命令和参数
- 和语言默认规范不一样的约定
- 踩过的坑（"search_code 是 RAG 辅助，优先用 grep"）
- 改了某个斜杠命令要同步改哪几个文件

如果内容超过 200 行，拆到 `.claude/rules/` 里，主文件用 `@` 引用：

```markdown
# my-project

编码规范见 @.claude/rules/code-style.md
测试规范见 @.claude/rules/testing.md
```

---

### 三、路径级规则：`.claude/rules/*.md`（最像 Cursor 的 `globs`）

在 `.cursor/rules/*.mdc` 里用 `globs` + `alwaysApply` 做的事，这里用 `paths`：

```bash
mkdir -p .claude/rules
```

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.spec.ts"
  - "tests/**"
---
# 测试规范
- 框架用 vitest，禁用 jest
- 每个文件必须有 describe，名称与被测模块一致
- mock 统一用 vi.mock()，禁止手写 mock
- 异步测试一律 async/await，禁止 done 回调
```

```markdown
---
paths:
  - "prisma/**"
  - "src/repositories/**"
---
# 数据库规范
- 已提交 main 的迁移文件禁止修改，只能新增
- service 层禁止直接调 Prisma client，必须走 repository
- 单次写入超 100 条必须分批 + 事务
```

两个容易踩的坑：

1. **不带 `paths` 的规则文件 = 无条件加载**，和写进 CLAUDE.md 消耗的 token 完全一样，拆出来只有组织结构上的好处，不省上下文。
2. **`paths` 控制的是"何时加载"，不是"何时生效"**。一旦因为匹配到文件被加载进来，它在本次会话里就不会卸载。所以别指望靠它实现"离开这个目录就失效"的隔离。

---

### 四、怎么确认它真的生效了

- 启动时终端会打印 `Detected CLAUDE.md in project root` / `Reading CLAUDE.md...`，有这行就是加载了。
- 会话里输入 `/memory` 可以查看当前加载了哪些记忆与规则文件。
- 最直接的办法：问它一句"我现在的规则里关于测试框架的要求是什么？"，能复述出来就说明生效了。
- 改完文件**不需要重启**，保存后自动检测；执行 `/compact` 也会从磁盘重新加载。

---

### 五、和 Cursor 双开的最佳姿势

Cursor 已经原生认项目根目录的 `CLAUDE.md`，所以**以 `CLAUDE.md` 为唯一真实源**最省事：

- 全局层：`~/.claude/CLAUDE.md`（Claude Code）+ Cursor Settings → Rules → Rules for AI（Cursor 没有全局 md 文件，这块得两边各写一份，或用软链接指向同一个文件）。
- 项目层：一份 `CLAUDE.md` 提交进 Git，两边都认。
- 如果你偏好用 `.claude/rules/` 当唯一源，就在 Cursor 里打开 "Include third-party Plugins, Skills, and other configs"，再把 `.cursor/rules/` 软链接过去：

```bash
ln -s ../.claude/rules .cursor/rules
```

---

----

## 一、 存放位置（决定了作用范围）

Skill 可以放在两个地方，优先级是 **项目级 > 个人级**：

1. **个人全局（所有项目通用）**：  
   放在 `~/.claude/skills/<skill-name>/SKILL.md`。适合你个人的通用习惯，比如“全局 Python 命名规范”。
2. **项目级（仅当前项目生效）**：  
   放在项目根目录的 `.claude/skills/<skill-name>/SKILL.md`。适合团队共享，可以直接提交到 Git。

### 二、 如何安装/创建 Skill

有四种常见方式：

1. **手动创建（最灵活）**：  
   自己建一个文件夹，里面写一个 `SKILL.md`。
2. **自然语言安装（最省事）**：  
   直接在对话框里对 Claude Code 说：“帮我安装这个 skill：[GitHub仓库链接]”，它会自动帮你下载并放到对应目录。
3. **CLI 工具安装**：  
   在终端运行 `npx skills add <GitHub仓库URL>`。
4. **插件市场安装**：  
   在对话框输入 `/plugin`，选择官方或社区市场一键安装。

### 三、 如何调用 Skill

Skill 的触发非常智能，有两种方式：

1. **自动触发（最常用）**：  
   你不需要记任何命令。只要你的提问匹配了 Skill 里的 `description`（描述），Claude Code 就会自动加载它。比如你写了一个叫 `python-code-review` 的 skill，你直接说“帮我审查一下这段代码”，它就会自动应用。
2. **手动斜杠命令触发**：  
   在对话框输入 `/skill-name`（比如 `/python-code-review`），强制 AI 使用这个技能。

### 四、 核心：SKILL.md 怎么写？

这是 Skill 的大脑。它由 YAML 元数据（Frontmatter）和 Markdown 指令组成。

**一个标准的 SKILL.md 示例：**

```markdown
---
# 1. 技能名称（必须小写，用连字符，和文件夹名一致）
name: python-code-review 

# 2. 描述（极其重要！AI 靠这句话来判断什么时候自动触发）
description: 对 Python 代码进行 PEP8 规范审查和可读性优化。当用户要求审查、重构或检查 Python 代码时使用。

# 3. 可选：限制该技能允许使用的工具
allowed-tools: Read, Grep
---

# 具体的执行指令（Markdown 格式）

当执行 Python 代码审查时，请严格按照以下步骤：
1. **规范检查**：检查变量命名是否符合 snake_case，类名是否符合 PascalCase。
2. **类型提示**：所有函数必须有 Type Hints。
3. **文档字符串**：公开函数必须有 docstring。
4. **输出格式**：最后给出一个总结表格，列出问题文件和修改建议。
```

### 五、 怎么确认它生效了？

装完 Skill 后，别急着用，先验证一下：

1. 在对话框输入 `/skills`，看看列表里有没有你刚装的技能。
2. 或者直接问它：“你现在有哪些可用的 skills？”
3. 发一个能匹配 `description` 的需求，看它是不是按你设定的流程走的。

**总结**：如果你有一套重复做的工作流，或者想让 AI 严格遵循某种输出格式，把它写成 Skill 放在 `.claude/skills/` 里，就能一劳永逸地解决“每次都要反复交代”的痛点。
