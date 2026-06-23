# Create Cursor Rules

一个用于维护老项目的 Agent Skill。它会先阅读项目真实代码和配置，提炼项目已有的工程规范，然后生成可被 AI 编码工具长期遵守的规则文件。

它不是“通用最佳实践模板”。它的目标是把老项目里已经存在但没有写成文档的约定提取出来，让后续 AI 改代码时更像项目原来的维护者。

## 它会生成什么

这个 skill 最终会生成一份“项目 AI 编码规则”。规则内容来自目标项目本身，包括：

- 项目技术栈、包管理器、构建工具和运行方式。
- 项目文件树，以及每个目录和关键文件的用途说明。
- 命名规范、目录约定、组件写法、状态管理方式、接口封装方式。
- ESLint、Prettier、Stylelint、TypeScript 等工具约束。
- 样式代码规则，尤其是项目存在 stylelint 时的强制要求。
- AI 后续修改代码时必须遵守的工作方式。
- 未确认的项目约定和需要人工确认的风险点。

根据用户最后选择的 AI 工具，生成位置不同：

```text
cursor  -> .cursor/rules/*.mdc
codex   -> AGENTS.md
claude  -> CLAUDE.md
```

选择 `cursor` 时，通常会生成这样的文件：

```text
.cursor/rules/
├── project-overview.mdc      # 技术栈、目录树、项目结构、运行方式
├── coding-conventions.mdc    # JS/TS/Vue/React 编码规范
├── style-conventions.mdc     # CSS/LESS/SCSS/stylelint 规范
└── ai-workflow.mdc           # AI 修改代码时必须遵守的工作方式
```

小项目也可能合并为一个文件：

```text
.cursor/rules/project-conventions.mdc
```

选择 `codex` 或 `claude` 时，会把同一份项目规则转换成对应工具常用的单文件格式：

```text
AGENTS.md
CLAUDE.md
```

## 生成的规则长什么样

规则不是简单列几条建议，而是一份可以直接指导 AI 编码的项目说明。示例结构如下：

```markdown
# Project Conventions

## Project Overview

- Framework: Vue 3 + Vite
- Language: TypeScript
- Package manager: pnpm
- Style: LESS + stylelint

## File Tree

src/
├── main.ts                 # 应用入口，注册全局插件并挂载 App
├── App.vue                 # 根组件，承载全局布局和 router-view
├── api/                    # 接口请求封装，页面禁止直接调用后端 SDK
├── components/             # 跨页面复用组件
├── views/                  # 页面级业务模块
├── router/                 # 路由定义和权限入口
├── stores/                 # 状态管理
├── utils/                  # 无状态工具函数
└── styles/                 # 全局样式、变量、mixin

## Coding Rules

- 新增页面必须放在 `src/views/` 对应业务目录下。
- API 请求必须通过 `src/api/` 或 `src/services/` 封装。
- 修改代码前先观察同目录已有写法，优先沿用项目现有模式。

## Style Rules

- 如果项目存在 stylelint，所有新增或修改的 CSS、LESS、SCSS 都必须通过 stylelint。
- 不为了格式化大规模重排无关样式文件。

## AI Workflow

- 不直接引入新的框架、状态库、请求库或样式方案，除非用户明确要求。
- 不为了现代化重构无关老代码。
- 优先复用项目已有组件、工具函数、请求封装和类型定义。
```

## 安装

使用 `skills` CLI 从 GitHub 安装：

```bash
npx skills add https://github.com/zeroanonx/create-rules --skill create-cursor-rules
```

也可以使用 GitHub 简写：

```bash
npx skills add zeroanonx/create-rules --skill create-cursor-rules
```

如果只想确认仓库里有哪些 skill：

```bash
npx skills add https://github.com/zeroanonx/create-rules --list
```

安装后建议重新开启一次 Agent 会话，让工具重新加载 skill。

## 使用方式

在需要生成规则的老项目中，对 Agent 说：

```text
使用 create-cursor-rules 分析这个项目，并生成 AI 编码规则。
```

Agent 会先分析项目，不会立刻写入最终规则。一切准备完成后，它必须先让你选择目标 AI：

```text
请选择你后续主要使用的 AI 工具：
- cursor
- codex
- claude
```

你选择后，它才会生成并写入对应规则文件。

## 它会分析哪些内容

skill 会优先读取这些项目证据：

- `package.json`
- `README.md`
- `vite.config.*` / `webpack.config.*` / `next.config.*` / `nuxt.config.*`
- `tsconfig.json` / `jsconfig.json`
- `.eslintrc*` / `eslint.config.*`
- `.prettierrc*` / `prettier.config.*`
- `stylelint.config.*` / `.stylelintrc*`
- `src/main.*` / `src/App.*` / `app/**`
- `src/router/**` / `src/routes/**`
- `src/components/**`
- `src/views/**` / `src/pages/**`
- `src/api/**` / `src/services/**`
- `src/store/**` / `src/stores/**`
- `src/utils/**` / `src/hooks/**` / `src/composables/**`
- `tests/**` / `test/**` / `__tests__/**`

缺失的文件会跳过，并在最终摘要中说明。

## Stylelint 强制规则

如果项目存在 stylelint，生成的规则会明确要求：

```markdown
任何新增或修改的样式代码都必须通过项目 stylelint。样式规则以项目 stylelint 配置为最高优先级，不得生成与 stylelint 冲突的 CSS、LESS、SCSS 或 PostCSS 写法。
```

如果旧代码和 stylelint 存在冲突，规则会要求：

- 新代码遵守 stylelint。
- 修改旧代码时只处理本次涉及范围。
- 不为了 stylelint 大规模重排无关样式文件。

## 规则优先级

生成规则时按以下优先级判断：

1. 项目已有配置：ESLint、Prettier、Stylelint、tsconfig、构建配置。
2. 项目真实代码中重复出现的模式。
3. README、团队文档和项目内已有说明。
4. 本 skill 内置的 `rules/standards.md`。
5. 框架通用最佳实践。

如果通用最佳实践和项目现状冲突，默认遵守项目现状，并在摘要中说明风险。

## 适用场景

- 接手老项目，需要快速熟悉原有规范。
- 项目没有完善文档，但代码里已经形成了约定。
- 团队希望 AI 后续改代码时遵守项目既有风格。
- 想为项目生成 Cursor Rules、Codex `AGENTS.md` 或 Claude `CLAUDE.md`。
- 前端项目需要把 ESLint、Prettier、Stylelint、TypeScript 等规则同步给 AI。

## 仓库结构

```text
create-rules/
├── README.md
└── create-rules/
    ├── SKILL.md            # Skill 主说明，定义分析流程和生成规则
    └── rules/
        └── standards.md    # 通用前端规范，作为项目事实之后的补充参考
```

## 质量标准

生成的规则必须具体、可执行、基于项目证据。

避免：

- 泛泛而谈的“保持代码整洁”。
- 与项目 Lint 或真实代码风格冲突的建议。
- 没有项目证据支撑的强制规则。
- 把外部最佳实践当成项目既有规范。
- 未经用户选择目标 AI 就直接写入规则文件。
