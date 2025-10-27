# Spec Kit 项目架构说明

**版本**: 0.0.17  
**文档创建日期**: 2025年  
**项目**: GitHub Spec Kit - 规范驱动开发工具包

---

## 目录

1. [项目概述](#项目概述)
2. [核心理念](#核心理念)
3. [整体架构](#整体架构)
4. [核心组件](#核心组件)
5. [工作流程](#工作流程)
6. [目录结构](#目录结构)
7. [技术栈](#技术栈)
8. [扩展性设计](#扩展性设计)
9. [数据流](#数据流)

---

## 项目概述

### 什么是 Spec Kit？

**Spec Kit** 是一个全面的工具包，用于实现规范驱动开发（Spec-Driven Development, SDD）方法论。这种方法论强调在实现之前创建清晰的规范文档，将规范作为软件开发的核心驱动力。

### 什么是 Specify CLI？

**Specify CLI** 是命令行接口工具，用于使用 Spec Kit 框架引导项目。它设置必要的目录结构、模板和 AI 代理集成，以支持规范驱动开发工作流程。

### 核心价值

- **规范优先**: 将规范文档作为首要产物，代码是规范的表达
- **AI 增强**: 支持多个 AI 编码助手，保持一致的项目结构和开发实践
- **模板驱动**: 通过结构化模板约束 AI 输出，确保质量
- **自动化工作流**: 从规范 → 计划 → 任务的自动化流程

---

## 核心理念

### 规范驱动开发（SDD）

SDD 颠覆了传统的软件开发流程。几十年来，代码一直是核心，规范只是我们构建和丢弃的脚手架。SDD 改变了这一点：**规范变成可执行的**，直接生成工作实现，而不仅仅是指导它们。

#### 关键原则

1. **规范作为通用语言**: 规范成为主要产物，代码成为其在特定语言和框架中的表达
2. **可执行规范**: 规范必须足够精确、完整和明确，以生成可工作的系统
3. **持续改进**: 一致性验证持续进行，而不是一次性检查
4. **研究驱动的上下文**: 研究代理在整个规范过程中收集关键上下文
5. **双向反馈**: 生产现实影响规范演进
6. **探索性分支**: 从同一规范生成多个实现方法

---

## 整体架构

Spec Kit 采用模块化、可扩展的架构，主要由以下几层组成：

```
┌─────────────────────────────────────────────────────────────┐
│                        用户界面层                              │
│  (命令行界面 - Specify CLI)                                   │
│  Commands: init, check                                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      核心引擎层                                │
│  - 项目初始化引擎                                              │
│  - 模板处理引擎                                                │
│  - AI 代理集成引擎                                             │
│  - 工具检查引擎                                                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      资源层                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   模板系统    │  │   脚本系统    │  │  配置文件     │      │
│  │              │  │              │  │              │      │
│  │ - spec       │  │ - bash       │  │ - memory/    │      │
│  │ - plan       │  │ - powershell │  │ - agents/    │      │
│  │ - tasks      │  │              │  │              │      │
│  │ - commands   │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    AI 代理集成层                               │
│  Claude | Gemini | Copilot | Cursor | Qwen | OpenCode      │
│  Windsurf | Codex | KiloCode | Auggie | Roo | Amazon Q     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    版本控制层                                  │
│  Git 仓库 + 规范文档 + 生成的代码                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心组件

### 1. Specify CLI (`src/specify_cli/__init__.py`)

#### 主要职责

- 项目初始化和配置
- AI 代理选择和集成
- 工具依赖检查
- 模板下载和解压
- Git 仓库初始化

#### 核心类和函数

##### StepTracker 类

```python
class StepTracker:
    """跟踪和渲染分层步骤，支持实时刷新"""
    
    def __init__(self, title: str)
    def add(self, key: str, label: str)
    def start(self, key: str, detail: str = "")
    def complete(self, key: str, detail: str = "")
    def error(self, key: str, detail: str = "")
    def skip(self, key: str, detail: str = "")
    def render(self)
```

**功能**: 提供类似 Claude Code 树形输出的进度跟踪，使用圆圈和颜色编码显示状态：
- ● 绿色：已完成
- ○ 灰色：待处理
- ○ 青色：运行中
- ● 红色：错误
- ○ 黄色：跳过

##### 主要命令

**init 命令**

```python
@app.command()
def init(
    project: str = typer.Argument(...),
    ai: Optional[str] = typer.Option(None),
    script: Optional[str] = typer.Option(None),
    here: bool = typer.Option(False),
    no_git: bool = typer.Option(False),
    force: bool = typer.Option(False),
    ignore_agent_tools: bool = typer.Option(False),
    github_token: Optional[str] = typer.Option(None)
)
```

**功能**:
1. 选择 AI 代理（交互式或通过参数）
2. 选择脚本类型（bash/PowerShell）
3. 验证工具依赖
4. 从 GitHub 获取最新发布版本
5. 下载并解压模板
6. 确保脚本可执行
7. 初始化 Git 仓库（可选）
8. 显示安全提示

**check 命令**

```python
@app.command()
def check()
```

**功能**:
- 检查所有支持的 AI 代理工具是否已安装
- 验证 Git 和 UV 工具的可用性
- 提供安装链接和指导

#### AI 代理支持

当前支持的 AI 代理（通过 `AI_CHOICES` 字典定义）:

| 代理 | 目录 | 格式 | CLI 工具 | 描述 |
|------|------|------|----------|------|
| Claude Code | `.claude/commands/` | Markdown | `claude` | Anthropic 的 Claude Code CLI |
| Gemini CLI | `.gemini/commands/` | TOML | `gemini` | Google 的 Gemini CLI |
| GitHub Copilot | `.github/prompts/` | Markdown | N/A (IDE) | VS Code 中的 GitHub Copilot |
| Cursor | `.cursor/commands/` | Markdown | `cursor-agent` | Cursor CLI |
| Qwen Code | `.qwen/commands/` | TOML | `qwen` | 阿里巴巴的 Qwen Code CLI |
| opencode | `.opencode/command/` | Markdown | `opencode` | opencode CLI |
| Windsurf | `.windsurf/workflows/` | Markdown | N/A (IDE) | Windsurf IDE 工作流 |
| Amazon Q Developer | `.amazonq/prompts/` | Markdown | `q` | Amazon Q Developer CLI |
| Codex | `.codex/commands/` | Markdown | `codex` | Codex CLI |
| KiloCode | `.kilocode/commands/` | Markdown | `kilocode` | KiloCode CLI |
| Auggie | `.augment/commands/` | Markdown | `auggie` | Auggie CLI |
| Roo Code | `.roo/commands/` | Markdown | N/A | Roo Code |

### 2. 模板系统 (`templates/`)

#### 模板类型

##### 核心模板

**spec-template.md** - 功能规范模板
```
结构:
- 执行流程 (Execution Flow)
- 快速指南 (Quick Guidelines)
- 用户场景和测试 (User Scenarios & Testing) *强制*
- 功能需求 (Functional Requirements) *强制*
- 关键实体 (Key Entities)
- 审核和验收检查清单 (Review & Acceptance Checklist)
- 执行状态 (Execution Status)
```

**plan-template.md** - 实现计划模板
```
结构:
- 执行流程 (/plan 命令范围)
- 技术上下文 (Technical Context)
- 宪法检查 (Constitution Check)
- 项目结构 (Project Structure)
- 阶段 0: 大纲和研究 (Outline & Research)
- 阶段 1: 设计和合约 (Design & Contracts)
- 阶段 2: 任务规划方法 (Task Planning Approach)
- 复杂度跟踪 (Complexity Tracking)
- 进度跟踪 (Progress Tracking)
```

**tasks-template.md** - 任务清单模板
```
结构:
- 执行流程 (Execution Flow)
- 格式约定: [ID] [P?] Description
- 阶段 3.1: 设置 (Setup)
- 阶段 3.2: 测试优先 (Tests First - TDD)
- 阶段 3.3: 核心实现 (Core Implementation)
- 阶段 3.4: 集成 (Integration)
- 阶段 3.5: 完善 (Polish)
```

**agent-file-template.md** - 代理文件模板
```
用于生成特定 AI 代理的上下文文件，包含:
- 项目宪法
- 技术栈信息
- 项目特定指南
```

**constitution.md** - 宪法模板
```
定义项目的核心原则和开发规范:
- 核心原则 (Library-First, CLI Interface, Test-First, etc.)
- 额外约束
- 开发工作流程
- 治理规则
```

##### 命令模板 (`templates/commands/`)

所有命令模板都遵循一致的结构，包含：
- 描述 (description)
- 脚本路径 (scripts)
- 执行流程 (Execution Flow)
- 命令特定内容

支持的命令：
1. **specify.md** - 创建功能规范
2. **plan.md** - 生成实现计划
3. **tasks.md** - 生成任务清单
4. **constitution.md** - 创建项目宪法
5. **clarify.md** - 澄清规范中的不确定性
6. **analyze.md** - 分析现有代码
7. **implement.md** - 执行实现

### 3. 脚本系统 (`scripts/`)

#### Bash 脚本 (`scripts/bash/`)

**common.sh** - 通用函数库
```bash
功能:
- 颜色输出函数 (print_success, print_error, print_info)
- 仓库根目录检测
- 错误处理
```

**check-prerequisites.sh** - 先决条件检查
```bash
检查项:
- git 是否安装
- 项目是否在 git 仓库中
- 规范目录结构是否存在
```

**create-new-feature.sh** - 创建新功能分支
```bash
功能:
1. 查找下一个可用的功能编号
2. 创建功能分支和目录
3. 复制 spec 模板
4. 初始化 git 分支
```

**setup-plan.sh** - 设置实现计划
```bash
功能:
- 将 plan 模板复制到功能目录
- 准备开始规划阶段
```

**update-agent-context.sh** - 更新 AI 代理上下文
```bash
功能:
1. 读取项目宪法
2. 扫描技术栈信息
3. 更新特定代理的上下文文件
支持的代理文件:
- CLAUDE.md
- .github/copilot-instructions.md
- GEMINI.md
- QWEN.md
- OPENCODE.md
- .cursor/rules/specify-rules.md
- .codex/rules/specify-rules.md
- .windsurf/rules/specify-rules.md
- .kilocode/rules/specify-rules.md
- .augment/rules/specify-rules.md
- .roo/rules/specify-rules.md
- AGENTS.md (通用)
```

#### PowerShell 脚本 (`scripts/powershell/`)

与 Bash 脚本对应，提供 Windows 平台支持：
- **common.ps1**
- **check-prerequisites.ps1**
- **create-new-feature.ps1**
- **setup-plan.ps1**
- **update-agent-context.ps1**

### 4. 配置和文档 (`memory/`, `docs/`)

#### memory/constitution.md

存储项目的核心原则和开发规范，作为所有开发活动的指导文件。

#### docs/ 目录

包含项目文档：
- **index.md** - 文档主页
- **installation.md** - 安装指南
- **quickstart.md** - 快速开始
- **local-development.md** - 本地开发指南
- **README.md** - 文档概述

---

## 工作流程

### 完整的 SDD 工作流

```
1. 项目初始化
   ↓
   specify init <project-name> --ai <agent>
   ↓
   [创建项目结构、模板、脚本]
   
2. 建立项目原则
   ↓
   /constitution Create principles...
   ↓
   [生成 constitution.md]
   
3. 创建功能规范
   ↓
   /specify Build an application...
   ↓
   [生成 specs/###-feature/spec.md]
   [创建功能分支]
   
4. 澄清不确定性（如需要）
   ↓
   /clarify [问题]
   ↓
   [更新 spec.md，移除 [NEEDS CLARIFICATION] 标记]
   
5. 生成技术计划
   ↓
   /plan The application uses...
   ↓
   [生成 plan.md, research.md, data-model.md, contracts/, quickstart.md]
   [更新代理上下文文件]
   
6. 生成任务清单
   ↓
   /tasks
   ↓
   [生成 tasks.md，包含 TDD 任务序列]
   
7. 执行实现
   ↓
   按照 tasks.md 执行
   ↓
   [测试驱动开发流程]
```

### 项目初始化流程详解

```
用户执行: specify init my-project --ai claude

1. 显示欢迎横幅
   ↓
2. AI 代理选择
   - 如果指定 --ai: 使用指定的代理
   - 否则: 显示交互式选择菜单
   ↓
3. 脚本类型选择
   - 如果指定 --script: 使用指定的类型
   - 否则: 自动检测（Windows=ps, 其他=sh）或交互式选择
   ↓
4. 工具依赖验证（如果未使用 --ignore-agent-tools）
   - 检查选定的 AI 代理 CLI 是否安装
   - 检查 Git 是否安装
   ↓
5. 获取最新发布版本
   - 通过 GitHub API 获取
   - 确定正确的包名（基于 AI 代理和脚本类型）
   ↓
6. 下载模板
   - 从 GitHub Releases 下载 ZIP
   - 显示下载进度
   ↓
7. 解压模板
   - 解压到项目目录
   - 验证文件完整性
   ↓
8. 确保脚本可执行（仅限 Unix）
   - chmod +x scripts/**/*.sh
   ↓
9. Git 初始化（除非 --no-git）
   - git init
   - 创建 .gitignore
   - 初始提交
   ↓
10. 显示安全提示
    - 提醒将代理文件夹添加到 .gitignore
    ↓
11. 显示下一步指导
    - 显示可用的斜杠命令
    - 提供使用示例
```

### 命令执行流程（以 /specify 为例）

```
AI 代理接收命令: /specify Build an application...

1. 加载 spec-template.md
   ↓
2. 执行 Execution Flow:
   a. 解析用户描述
   b. 提取关键概念
   c. 标记不确定的方面 [NEEDS CLARIFICATION]
   d. 填充用户场景和测试部分
   e. 生成功能需求
   f. 识别关键实体
   g. 运行审核检查清单
   ↓
3. 调用 create-new-feature.sh 脚本
   - 查找下一个功能编号
   - 创建 specs/###-feature/ 目录
   - 创建 git 分支
   ↓
4. 生成 spec.md 文件
   - 使用模板结构
   - 填充用户提供的内容
   - 保留 [NEEDS CLARIFICATION] 标记
   ↓
5. 返回规范给用户审核
```

---

## 目录结构

### 源代码仓库结构

```
spec-kit/
├── .github/                    # GitHub 配置和工作流
│   ├── workflows/              # CI/CD 工作流
│   └── agents/                 # GitHub 代理指令
│
├── docs/                       # 项目文档
│   ├── index.md
│   ├── installation.md
│   ├── quickstart.md
│   └── local-development.md
│
├── media/                      # 媒体资源（图片、logo）
│
├── memory/                     # 项目记忆和上下文
│   └── constitution.md         # 项目宪法模板
│
├── scripts/                    # 自动化脚本
│   ├── bash/                   # Bash 脚本（Linux/Mac）
│   │   ├── common.sh
│   │   ├── check-prerequisites.sh
│   │   ├── create-new-feature.sh
│   │   ├── setup-plan.sh
│   │   └── update-agent-context.sh
│   │
│   └── powershell/             # PowerShell 脚本（Windows）
│       ├── common.ps1
│       ├── check-prerequisites.ps1
│       ├── create-new-feature.ps1
│       ├── setup-plan.ps1
│       └── update-agent-context.ps1
│
├── src/                        # 源代码
│   └── specify_cli/            # Specify CLI 包
│       └── __init__.py         # 主 CLI 实现
│
├── templates/                  # 模板文件
│   ├── commands/               # 斜杠命令模板
│   │   ├── specify.md          # /specify 命令
│   │   ├── plan.md             # /plan 命令
│   │   ├── tasks.md            # /tasks 命令
│   │   ├── constitution.md     # /constitution 命令
│   │   ├── clarify.md          # /clarify 命令
│   │   ├── analyze.md          # /analyze 命令
│   │   └── implement.md        # /implement 命令
│   │
│   ├── spec-template.md        # 功能规范模板
│   ├── plan-template.md        # 实现计划模板
│   ├── tasks-template.md       # 任务清单模板
│   └── agent-file-template.md  # 代理文件模板
│
├── AGENTS.md                   # 代理集成指南
├── ARCHITECTURE.md             # 本文档（架构说明）
├── CHANGELOG.md                # 变更日志
├── CODE_OF_CONDUCT.md          # 行为准则
├── CONTRIBUTING.md             # 贡献指南
├── LICENSE                     # 许可证
├── README.md                   # 项目自述文件
├── SECURITY.md                 # 安全政策
├── SUPPORT.md                  # 支持信息
├── pyproject.toml              # Python 项目配置
└── spec-driven.md              # SDD 方法论详解
```

### 初始化后的项目结构

使用 `specify init` 初始化项目后的目录结构：

```
my-project/
├── .git/                       # Git 仓库
│
├── .<agent>/                   # AI 代理特定目录
│   ├── commands/               # 或 prompts/, workflows/ 等
│   │   ├── specify.md
│   │   ├── plan.md
│   │   ├── tasks.md
│   │   ├── constitution.md
│   │   ├── clarify.md
│   │   ├── analyze.md
│   │   └── implement.md
│   │
│   └── rules/                  # 代理规则（部分代理）
│       └── specify-rules.md
│
├── memory/                     # 项目上下文
│   └── constitution.md         # 项目宪法
│
├── scripts/                    # 自动化脚本
│   └── [bash|powershell]/      # 根据选择
│       ├── common.[sh|ps1]
│       ├── check-prerequisites.[sh|ps1]
│       ├── create-new-feature.[sh|ps1]
│       ├── setup-plan.[sh|ps1]
│       └── update-agent-context.[sh|ps1]
│
├── specs/                      # 功能规范（运行命令后生成）
│   └── 001-feature-name/
│       ├── spec.md             # 功能规范
│       ├── plan.md             # 实现计划
│       ├── research.md         # 技术研究
│       ├── data-model.md       # 数据模型
│       ├── quickstart.md       # 快速验证
│       ├── contracts/          # API 合约
│       │   ├── api-spec.json
│       │   └── ...
│       └── tasks.md            # 任务清单
│
├── templates/                  # 模板文件
│   ├── spec-template.md
│   ├── plan-template.md
│   ├── tasks-template.md
│   └── agent-file-template.md
│
└── [AGENT].md                  # 代理上下文文件
    # 或 copilot-instructions.md, CLAUDE.md, GEMINI.md 等
```

---

## 技术栈

### 核心技术

#### Python 3.11+
项目使用 Python 3.11 或更高版本，利用现代 Python 特性。

#### 主要依赖

```toml
[project.dependencies]
typer       # CLI 框架，提供命令行参数解析
rich        # 富文本和终端格式化
httpx       # HTTP 客户端，用于 GitHub API
platformdirs # 跨平台目录路径
readchar    # 跨平台键盘输入
truststore  # SSL 证书管理
```

#### 构建系统

- **构建后端**: Hatchling
- **包管理**: UV (推荐) 或 pip
- **分发**: PyPI 和 GitHub Releases

### CLI 框架 (Typer)

使用 Typer 构建命令行界面，提供：
- 自动类型验证
- 帮助文档生成
- 子命令支持
- 选项和参数解析

### UI 库 (Rich)

使用 Rich 提供丰富的终端输出：
- 彩色文本和样式
- 进度条和加载动画
- 表格和树形结构
- 面板和边框
- 实时更新（Live）

### HTTP 客户端 (HTTPX)

用于与 GitHub API 交互：
- 获取最新发布版本
- 下载模板包
- 支持认证（GitHub Token）
- SSL 证书验证

---

## 扩展性设计

### 1. AI 代理扩展

#### 添加新代理的步骤

1. **更新 AI_CHOICES**
   ```python
   AI_CHOICES = {
       # ... 现有代理 ...
       "new-agent": "New Agent Name",
   }
   ```

2. **更新 agent_folder_map**
   ```python
   agent_folder_map = {
       # ... 现有映射 ...
       "new-agent": ".newagent/",
   }
   ```

3. **创建命令文件生成器**
   - 确定命令文件格式（Markdown 或 TOML）
   - 确定目录结构
   - 确定参数占位符约定

4. **更新发布脚本**
   - `.github/workflows/scripts/create-release-packages.sh`
   - 添加新代理的包生成逻辑

5. **更新上下文更新脚本**
   - `scripts/bash/update-agent-context.sh`
   - `scripts/powershell/update-agent-context.ps1`
   - 添加新代理的上下文文件支持

6. **更新文档**
   - README.md
   - AGENTS.md

#### 代理类别

**CLI 基础代理** - 需要命令行工具：
- Claude Code, Gemini CLI, Cursor, Qwen Code, opencode, Codex, Amazon Q

**IDE 基础代理** - 在 IDE 内工作：
- GitHub Copilot, Windsurf

### 2. 命令文件格式

#### Markdown 格式
用于：Claude, Cursor, opencode, Windsurf, Amazon Q, Copilot, Codex

```markdown
---
description: "命令描述"
scripts:
  sh: scripts/bash/script-name.sh
  ps: scripts/powershell/script-name.ps1
---

命令内容，包含 {SCRIPT} 和 $ARGUMENTS 占位符。
```

#### TOML 格式
用于：Gemini, Qwen

```toml
description = "命令描述"

[scripts]
sh = "scripts/bash/script-name.sh"
ps = "scripts/powershell/script-name.ps1"

prompt = """
命令内容，包含 {SCRIPT} 和 {{args}} 占位符。
"""
```

### 3. 参数模式

不同代理使用不同的参数占位符：

- **Markdown/prompt-based**: `$ARGUMENTS`
- **TOML-based**: `{{args}}`
- **脚本占位符**: `{SCRIPT}` (被实际脚本路径替换)
- **代理占位符**: `__AGENT__` (被代理名称替换)

### 4. 模板扩展

#### 添加新模板的步骤

1. 在 `templates/` 中创建新模板文件
2. 定义模板结构和必填字段
3. 创建相应的命令模板（在 `templates/commands/`）
4. 更新文档说明模板用途

#### 模板设计原则

- **结构化约束**: 强制执行特定的部分和格式
- **执行流程**: 包含明确的处理步骤
- **验证检查**: 内置质量门控
- **元数据**: 版本、日期、状态跟踪

---

## 数据流

### 项目初始化数据流

```
用户输入
   ↓
[项目名称, AI 代理, 脚本类型]
   ↓
Specify CLI (init 命令)
   ↓
┌─────────────────────────────┐
│ 1. 验证输入                  │
│ 2. 检查工具依赖              │
└─────────────────────────────┘
   ↓
GitHub API
   ↓
[获取最新发布版本信息]
   ↓
┌─────────────────────────────┐
│ 下载模板包                   │
│ (spec-kit-template-{ai}-{script}.zip) │
└─────────────────────────────┘
   ↓
本地文件系统
   ↓
┌─────────────────────────────┐
│ 1. 解压到项目目录            │
│ 2. 设置脚本权限              │
│ 3. 初始化 Git 仓库           │
└─────────────────────────────┘
   ↓
项目就绪
   ↓
[显示下一步指导]
```

### 规范生成数据流

```
用户描述
   ↓
/specify 命令
   ↓
AI 代理
   ↓
加载 spec-template.md
   ↓
┌─────────────────────────────┐
│ 执行流程:                    │
│ 1. 解析描述                  │
│ 2. 提取概念                  │
│ 3. 标记不确定性              │
│ 4. 生成场景                  │
│ 5. 生成需求                  │
│ 6. 识别实体                  │
│ 7. 审核检查                  │
└─────────────────────────────┘
   ↓
调用 create-new-feature.sh
   ↓
┌─────────────────────────────┐
│ 1. 查找功能编号              │
│ 2. 创建目录结构              │
│ 3. 创建 git 分支             │
└─────────────────────────────┘
   ↓
生成 spec.md
   ↓
┌─────────────────────────────┐
│ 包含:                        │
│ - 功能名称和元数据           │
│ - 用户场景                   │
│ - 功能需求                   │
│ - 关键实体                   │
│ - [NEEDS CLARIFICATION]     │
└─────────────────────────────┘
   ↓
等待用户审核
```

### 计划生成数据流

```
用户技术偏好
   ↓
/plan 命令
   ↓
AI 代理
   ↓
加载 plan-template.md + spec.md
   ↓
┌─────────────────────────────┐
│ 执行流程:                    │
│ 1. 加载规范                  │
│ 2. 填充技术上下文            │
│ 3. 宪法检查                  │
│ 4. 阶段 0: 研究              │
│ 5. 阶段 1: 设计和合约        │
│ 6. 计划阶段 2               │
└─────────────────────────────┘
   ↓
生成多个文档
   ↓
┌─────────────────────────────┐
│ - plan.md (主计划)           │
│ - research.md (技术研究)     │
│ - data-model.md (数据模型)   │
│ - quickstart.md (快速验证)   │
│ - contracts/ (API 合约)      │
└─────────────────────────────┘
   ↓
调用 update-agent-context.sh
   ↓
更新代理上下文文件
   ↓
┌─────────────────────────────┐
│ 从以下来源收集:              │
│ - memory/constitution.md    │
│ - 检测到的技术栈             │
│ - 项目结构                   │
└─────────────────────────────┘
   ↓
[AGENT].md / copilot-instructions.md / 等
```

### 任务生成数据流

```
/tasks 命令
   ↓
AI 代理
   ↓
加载 tasks-template.md + plan.md + 设计文档
   ↓
┌─────────────────────────────┐
│ 执行流程:                    │
│ 1. 加载计划                  │
│ 2. 加载设计文档              │
│ 3. 生成任务分类              │
│ 4. 应用任务规则              │
│ 5. 编号任务                  │
│ 6. 生成依赖图                │
└─────────────────────────────┘
   ↓
生成 tasks.md
   ↓
┌─────────────────────────────┐
│ 包含:                        │
│ - 设置任务                   │
│ - 测试任务（TDD）            │
│ - 实现任务                   │
│ - 集成任务                   │
│ - 完善任务                   │
│ - [P] 标记（并行）           │
└─────────────────────────────┘
   ↓
准备执行
```

### 代理上下文更新流程

```
触发器
   ↓
[/plan 命令完成 OR 手动调用脚本]
   ↓
update-agent-context.sh/ps1
   ↓
┌─────────────────────────────┐
│ 1. 检测代理类型              │
│ 2. 读取 constitution.md     │
│ 3. 扫描项目结构              │
│ 4. 收集技术栈信息            │
└─────────────────────────────┘
   ↓
生成上下文内容
   ↓
┌─────────────────────────────┐
│ 包含:                        │
│ - 项目宪法                   │
│ - 检测到的技术               │
│ - 目录结构                   │
│ - SDD 工作流指导             │
└─────────────────────────────┘
   ↓
更新代理文件
   ↓
特定代理的上下文文件:
- CLAUDE.md
- .github/copilot-instructions.md
- GEMINI.md
- QWEN.md
- OPENCODE.md
- .cursor/rules/specify-rules.md
- .codex/rules/specify-rules.md
- .windsurf/rules/specify-rules.md
- .kilocode/rules/specify-rules.md
- .augment/rules/specify-rules.md
- .roo/rules/specify-rules.md
- AGENTS.md (通用)
   ↓
AI 代理加载更新的上下文
```

---

## 关键设计决策

### 1. 模板驱动的质量控制

**问题**: AI 生成的内容可能不一致且缺乏结构。

**解决方案**: 强制使用结构化模板，包含：
- 明确的执行流程
- 必填和可选部分
- 验证检查清单
- 标记不确定性的机制

**效果**: 
- 约束 AI 输出到预期格式
- 确保关键信息不被遗漏
- 提高规范的可测试性

### 2. 多代理支持

**问题**: 不同的 AI 代理有不同的命令格式和工作方式。

**解决方案**: 
- 抽象的命令接口
- 特定代理的文件格式生成器
- 统一的模板内容，不同的包装格式

**效果**:
- 用户可以选择自己喜欢的 AI 工具
- 相同的 SDD 工作流程适用于所有代理
- 易于添加新代理支持

### 3. 脚本自动化

**问题**: 手动创建目录、分支和文件容易出错。

**解决方案**: 
- 提供 Bash 和 PowerShell 脚本
- 集成到命令模板中
- 自动化常见任务（创建功能、更新上下文）

**效果**:
- 减少手动错误
- 加快工作流程
- 跨平台支持（Linux, Mac, Windows）

### 4. 宪法驱动的开发

**问题**: 项目原则和约束在开发过程中容易被忽略。

**解决方案**:
- 在规划阶段强制执行宪法检查
- 将宪法内容注入 AI 代理上下文
- 在模板中包含宪法验证步骤

**效果**:
- 确保架构一致性
- 自动执行最佳实践
- 防止违反核心原则

### 5. 测试驱动开发（TDD）集成

**问题**: TDD 容易被跳过或不正确执行。

**解决方案**:
- 任务模板强制执行测试优先顺序
- 明确标记测试任务必须在实现前完成
- 生成合约测试和集成测试脚手架

**效果**:
- 强制执行 TDD 纪律
- 提高代码质量
- 更好的测试覆盖率

### 6. 版本控制集成

**问题**: 规范和代码的版本控制容易脱节。

**解决方案**:
- 每个功能都有独立的分支
- 规范文档与代码在同一仓库
- 自动创建功能分支

**效果**:
- 规范和代码同步演进
- 更好的协作和审核
- 清晰的历史记录

---

## 安全考虑

### 1. 代理文件夹

某些 AI 代理可能在项目文件夹中存储凭据、认证令牌或其他敏感信息。

**建议**: 将代理文件夹添加到 `.gitignore`：
```
.claude/
.gemini/
.cursor/
.qwen/
.opencode/
.codex/
.windsurf/
.kilocode/
.augment/
.roo/
.github/copilot/
.amazonq/
```

### 2. GitHub Token

使用 GitHub API 时，可以提供个人访问令牌以避免速率限制。

**最佳实践**:
- 使用环境变量 `GH_TOKEN` 或 `GITHUB_TOKEN`
- 不要在代码中硬编码令牌
- 使用最小权限范围（public_repo 读取）

### 3. 脚本执行

脚本自动化功能强大，但也需要谨慎。

**安全措施**:
- 脚本只在项目目录内操作
- 在执行前验证路径
- 不执行未知来源的脚本

---

## 性能考虑

### 1. 模板下载

**优化**:
- 使用 GitHub Releases 托管预打包模板
- 分代理和脚本类型打包，减小下载大小
- 支持缓存（未来改进）

### 2. 文件系统操作

**优化**:
- 批量创建目录和文件
- 使用流式解压
- 避免不必要的文件读写

### 3. Git 操作

**优化**:
- 只在必要时初始化仓库
- 批量提交相关更改
- 提供 `--no-git` 选项跳过 Git 操作

---

## 未来改进方向

### 1. 本地模板缓存

缓存下载的模板，避免重复下载。

### 2. 交互式配置向导

更友好的初始化向导，指导新用户完成设置。

### 3. 模板版本管理

支持指定模板版本，允许项目使用特定版本的模板。

### 4. 插件系统

允许第三方扩展添加自定义命令和模板。

### 5. Web UI

提供 Web 界面，用于可视化管理规范和计划。

### 6. 协作功能

支持团队协作，包括规范审核、评论和批准流程。

### 7. 指标和分析

收集项目指标，分析规范质量和开发效率。

### 8. IDE 集成

为流行的 IDE 提供插件（VS Code, IntelliJ, etc.）。

---

## 故障排除

### 常见问题

#### 1. "No module named 'specify_cli'"

**原因**: 包未正确安装。

**解决方案**:
```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

#### 2. "Agent CLI not found"

**原因**: 选定的 AI 代理 CLI 工具未安装。

**解决方案**:
- 使用 `specify check` 查看安装状态
- 按照提示安装缺失的工具
- 或使用 `--ignore-agent-tools` 跳过检查

#### 3. "Permission denied" 执行脚本

**原因**: 脚本文件没有执行权限（Unix）。

**解决方案**:
```bash
chmod +x scripts/**/*.sh
```

#### 4. 下载失败

**原因**: 网络问题或 GitHub API 速率限制。

**解决方案**:
- 检查网络连接
- 提供 GitHub Token: `--github-token YOUR_TOKEN`
- 稍后重试

#### 5. Git 初始化失败

**原因**: Git 未安装或目录已是 Git 仓库。

**解决方案**:
- 安装 Git
- 使用 `--no-git` 跳过 Git 初始化
- 手动初始化 Git

---

## 总结

Spec Kit 是一个精心设计的工具包，旨在通过规范驱动开发（SDD）方法论改变软件开发流程。其核心架构包括：

1. **Specify CLI**: 强大的命令行工具，用于项目初始化和配置
2. **模板系统**: 结构化模板，确保规范和计划的一致性和质量
3. **脚本基础设施**: 跨平台自动化脚本，加速开发工作流
4. **AI 代理集成**: 支持多个 AI 助手，保持灵活性和选择性
5. **SDD 工作流**: 从规范到计划再到任务的完整流程

通过将规范作为核心产物并使用 AI 生成代码，Spec Kit 使开发团队能够：
- 更快地构建高质量软件
- 保持规范和实现的同步
- 强制执行最佳实践和架构原则
- 支持测试驱动开发
- 促进团队协作和沟通

这个架构设计考虑了可扩展性、可维护性和用户体验，为现代软件开发提供了一个强大而灵活的框架。

---

**文档版本**: 1.0  
**最后更新**: 2025年  
**维护者**: Spec Kit 团队
