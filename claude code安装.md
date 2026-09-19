### 安装clauld code

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
