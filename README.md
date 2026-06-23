# Create AI Coding Rules Skill

这是一个用于维护老项目的 Cursor Skill。它会分析项目真实代码、目录结构、Lint 配置和团队约定，再根据用户选择生成对应 AI 工具可使用的编码规则文件。

支持的目标 AI：

- `cursor`：生成 `.cursor/rules/*.mdc`
- `codex`：生成 `AGENTS.md`
- `claude`：生成 `CLAUDE.md`

核心原则：**项目事实优先，通用最佳实践其次**。生成规则时优先遵守老项目已经形成的写法，不为了“现代化”强行改造项目风格。

## 适用场景

- 接手老项目，需要快速熟悉原有规范。
- 希望 AI 后续改代码时遵守项目既有风格。
- 需要为项目生成 Cursor Rules、Codex `AGENTS.md` 或 Claude `CLAUDE.md`。
- 项目缺少规范文档，但代码里已经形成了约定。
- 维护前端项目时，需要把 ESLint、Prettier、Stylelint、TypeScript 等规则同步给 AI。

## 目录结构

```text
create-rules/
├── SKILL.md            # Skill 主说明，定义分析流程和生成规则
└── rules/
    └── standards.md    # 通用前端规范，作为项目事实之后的补充参考
```

## 工作方式

这个 skill 会按以下顺序工作：

1. 探测项目技术栈、包管理器、构建工具和 Lint 配置。
2. 读取项目关键文件，如 `package.json`、`README.md`、构建配置、Lint 配置和入口文件。
3. 抽样分析真实代码，识别目录职责、命名方式、组件写法、状态管理、请求封装、样式组织和测试习惯。
4. 输出项目文件树，并为目录和关键文件标注用途。
5. 一切准备就绪后，提示用户选择目标 AI：`cursor` / `codex` / `claude`。
6. 用户选择后，生成对应规则文件。
7. 如果目标规则文件已存在，先读取并给出合并建议，不直接覆盖用户已有规则。

## 目标 AI 选择

生成最终规则前，必须先让用户选择目标 AI。

```text
请选择你后续主要使用的 AI 工具：
- cursor
- codex
- claude
```

选择不同目标时，产物不同：

| 选择     | 生成目标              | 说明                               |
| -------- | --------------------- | ---------------------------------- |
| `cursor` | `.cursor/rules/*.mdc` | 使用 Cursor Rules 主流 `.mdc` 格式 |
| `codex`  | `AGENTS.md`           | 使用 Codex 常见项目规则文件        |
| `claude` | `CLAUDE.md`           | 使用 Claude 常见项目规则文件       |

用户未选择前，skill 不应生成或写入最终规则文件。

## Cursor Rules 输出

选择 `cursor` 时，推荐生成多个 `.mdc` 文件：

```text
.cursor/rules/
├── project-overview.mdc      # 技术栈、目录树、项目结构、运行方式
├── coding-conventions.mdc    # JS/TS/Vue/React 编码规范
├── style-conventions.mdc     # CSS/LESS/SCSS/stylelint 规范
└── ai-workflow.mdc           # AI 修改代码时必须遵守的工作方式
```

小项目可以合并为：

```text
.cursor/rules/project-conventions.mdc
```

每个 `.mdc` 文件应包含 YAML frontmatter，例如：

```markdown
---
description: 项目编码规范，适用于该项目的所有代码生成和修改
globs:
  - "**/*"
alwaysApply: true
---
```

## Stylelint 规则

如果项目存在以下任意配置，生成的样式代码必须严格遵守项目 stylelint：

- `stylelint.config.*`
- `.stylelintrc`
- `.stylelintrc.*`
- `package.json` 中的 `stylelint` 字段
- `package.json` scripts 中包含 `stylelint`

生成的规则中必须明确：

```markdown
任何新增或修改的样式代码都必须通过项目 stylelint。样式规则以项目 stylelint 配置为最高优先级，不得生成与 stylelint 冲突的 CSS、LESS、SCSS 或 PostCSS 写法。
```

## 项目结构树

产出规则时必须包含项目结构树和用途说明。示例：

```text
src/
├── main.ts                 # 应用入口，挂载根组件并注册全局插件
├── App.vue                 # 根组件，承载全局布局或 router-view
├── api/                    # 接口请求封装，页面不应直接调用第三方 SDK
├── components/             # 跨页面复用组件
├── views/                  # 页面级业务模块
├── router/                 # 路由定义与权限入口
├── stores/                 # 状态管理
├── utils/                  # 无状态工具函数
└── styles/                 # 全局样式、变量、mixin
```

要求：

- 覆盖所有非生成的源码文件和关键配置文件。
- 忽略 `node_modules`、`dist`、`build`、`.git`、缓存目录和大型生成物。
- 能判断用途的目录和文件必须写明职责。
- 无法判断用途时标注“用途需确认”，不要编造。

## 规则优先级

生成规则时按以下优先级判断：

1. 项目已有配置：ESLint、Prettier、Stylelint、tsconfig、构建配置。
2. 项目真实代码中重复出现的模式。
3. README、团队文档和项目内已有说明。
4. `create-rules/rules/standards.md`。
5. 框架通用最佳实践。

如果通用最佳实践和项目现状冲突，默认遵守项目现状，并在摘要中说明风险。

## 使用方式

将 `create-rules/` 作为 skill 放入你的 Cursor skills 目录后，在目标项目中请求：

```text
请使用 create-cursor-rules 分析这个老项目，并生成 AI 编码规则。
```

推荐让 agent 先完成项目分析，再根据提示选择目标 AI。

## 质量标准

生成的规则应该具体、可执行、基于项目证据。

避免：

- 泛泛而谈的“保持代码整洁”。
- 与项目 Lint 或真实代码风格冲突的建议。
- 没有项目证据支撑的强制规则。
- 把外部最佳实践当成项目既有规范。
- 未经用户选择就直接写入规则文件。
