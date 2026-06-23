---
name: create-rules
description: >-
  分析老项目的技术栈、目录结构、代码风格、Lint 配置和团队约定，
  在用户选择 cursor、codex 或 claude 后，生成对应的 AI 编码规则文件：
  .cursor/rules/*.mdc、AGENTS.md 或 CLAUDE.md。
  Use when maintaining legacy projects, learning existing conventions,
  creating AI coding rules, or making AI code follow the project's original style.
license: MIT
metadata:
  author: zeroanonx
  version: "1.3.1"
---

# Create AI Coding Rules

帮助用户在维护老项目时，先理解项目已有规范，再根据用户选择生成可被 Cursor、Codex 或 Claude 长期遵守的 AI 编码规则。

核心原则：**项目事实优先，通用最佳实践其次**。不要为了“现代化”强行改变老项目既有风格。

## 工作流

1. 探测项目技术栈、包管理器、脚本、构建工具和 Lint 配置。
2. 抽样阅读真实代码，识别目录职责、命名方式、组件写法、状态管理、请求封装、样式组织和测试习惯。
3. 生成项目文件树，并为树中的目录和关键文件标注用途。
4. 准备就绪后，必须先让用户选择目标 AI：`cursor` / `codex` / `claude`。
5. 用户选择后，基于证据提炼并生成对应 AI 的 rules 内容。
6. 检查目标规则文件是否已存在；存在时先读取并给出合并建议，禁止覆盖用户已有规则。
7. 输出调查摘要：已识别规则、未确认项、已生成文件、后续使用方式。

## 读取项目信息

优先读取这些文件；缺失则跳过，并在最终摘要中注明：

| 文件 | 用途 |
| ---- | ---- |
| `package.json` | 依赖、脚本、包管理器、项目类型 |
| `README.md` | 运行方式、业务说明、维护约定 |
| `vite.config.*` / `webpack.config.*` / `next.config.*` / `nuxt.config.*` | 构建与框架配置 |
| `tsconfig.json` / `jsconfig.json` | TS/JS 路径别名、严格程度、编译目标 |
| `.eslintrc*` / `eslint.config.*` | JavaScript/TypeScript/Vue/React Lint 规则 |
| `.prettierrc*` / `prettier.config.*` | 格式化规则 |
| `stylelint.config.*` / `.stylelintrc*` | CSS/LESS/SCSS/PostCSS Lint 规则 |
| `tailwind.config.*` / `postcss.config.*` | 样式工具链 |
| `src/main.*` / `src/App.*` / `app/**` | 应用入口与根组件 |
| `src/router/**` / `src/routes/**` | 路由、权限入口 |
| `src/components/**` | 通用组件和组件命名方式 |
| `src/views/**` / `src/pages/**` | 页面模块和业务组织方式 |
| `src/api/**` / `src/services/**` | 请求封装和接口约定 |
| `src/store/**` / `src/stores/**` | Vuex/Pinia/Redux/Zustand 等状态管理 |
| `src/utils/**` / `src/hooks/**` / `src/composables/**` | 工具函数、Hooks、Composables |
| `tests/**` / `test/**` / `__tests__/**` | 测试框架和测试写法 |

## 规范加载

始终读取：[rules/standards.md](rules/standards.md)

按依赖读取同级 skills；缺失则跳过，并在最终摘要中注明：

| 条件 | 路径 |
| ---- | ---- |
| `vue` / `@vue/` / `nuxt` | `../vue-best-practices/SKILL.md` |
| `react` / `react-dom` / `next` | `../vercel-react-best-practices/SKILL.md` |
| `typescript` / `tsconfig.json` / `.ts` / `.tsx` | `../typescript-best-practices/SKILL.md` |

## 优先级

生成规则时按以下优先级判断：

1. 项目已有配置：ESLint、Prettier、Stylelint、tsconfig、构建配置。
2. 项目真实代码中重复出现的模式。
3. README、团队文档和项目内已有说明。
4. 本 skill 的 `rules/standards.md`。
5. 框架通用最佳实践。

如果通用最佳实践与项目现状冲突，默认遵守项目现状，并在摘要中说明风险。

## Stylelint 强制项

如果项目存在以下任意配置，生成的 CSS / LESS / SCSS / PostCSS 代码必须严格遵守项目 stylelint：

- `stylelint.config.*`
- `.stylelintrc`
- `.stylelintrc.*`
- `package.json` 中的 `stylelint` 字段
- `package.json` scripts 中包含 `stylelint`

发现 stylelint 后，必须继续读取并提取这些信息，但生成到 rules 时只保留必要结论：

- stylelint 配置文件内容。
- `package.json` 中可运行的 stylelint / lint:style / lint 脚本。
- `order/properties-order` 的属性顺序规则。
- `no-descending-specificity` 等选择器顺序规则。
- 1-2 个已通过 lint 的同类样式文件，作为排序参考。

Stylelint 规则要强，但产出要短。生成 rules 时不要写大段解释，只写这三类可执行内容：

1. 真实校验命令。
2. 必须修复到 `0 error` 才能交付。
3. 常见错误处理。

推荐写成：

```markdown
样式校验命令：pnpm lint:style

修改 CSS / LESS / SCSS 后必须运行样式校验，并修复到 0 error 后再交付。

常见错误处理：
- `order/properties-order`：按项目配置顺序重排属性。
- `no-descending-specificity`：调整选择器顺序，低特异性选择器放在高特异性选择器之前。
- 避免在文件末尾追加重复或倒序选择器，优先在原选择器块内修改。
```

如无法识别脚本，rules 中必须要求 agent 先询问用户或说明无法验证，不得假装已通过。

如 stylelint 与项目旧代码存在冲突：

- 新代码遵守 stylelint。
- 修改旧代码时只修正本次涉及范围。
- 不为了通过 stylelint 大规模重排无关样式文件。

不要把 stylelint 配置原文完整搬进 rules；只保留 AI 写代码时必须执行的校验命令和高频错误处理方式。

## 项目结构与文件用途树

必须分析项目结构，并在产出中列出文件树和用途说明。

要求：

- 覆盖所有非生成的源码文件，以及配置、路由、组件、页面、状态、请求、样式、测试等关键文件。
- 对树中的目录和文件使用注释说明用途。
- 忽略 `node_modules`、`dist`、`build`、`.git`、缓存目录、大型生成物。
- 对无法判断用途的文件标注“用途需确认”，不要编造。
- 如果项目很大，先输出关键目录树，并说明完整树因体量过大而省略的范围；不得省略核心业务目录。

示例格式：

```text
src/
├── main.ts                 # 应用入口，挂载根组件并注册全局插件
├── App.vue                 # 根组件，承载全局布局或 router-view
├── api/                    # 接口请求封装，页面不应直接调用第三方 SDK
│   └── user.ts             # 用户相关接口
├── components/             # 跨页面复用组件
├── views/                  # 页面级业务模块
├── router/                 # 路由定义与权限入口
├── stores/                 # 状态管理
├── utils/                  # 无状态工具函数
└── styles/                 # 全局样式、变量、mixin
```

## 目标 AI 选择门禁

一切扫描、分析和规则草稿准备就绪后，**必须先让用户选择目标 AI，再生成或写入最终 rules**。

优先使用选择框提问：

```text
请选择你后续主要使用的 AI 工具：
- cursor
- codex
- claude
```

如果可用，使用 `AskQuestion` 工具，选项固定为：

- `cursor`
- `codex`
- `claude`

如果 `AskQuestion` 不可用，则用对话明确询问。用户未选择前：

- 不生成最终规则文件。
- 不写入 `.cursor/rules`、`AGENTS.md`、`CLAUDE.md` 或其他目标规则文件。
- 可以先展示已完成的项目分析摘要和待生成规则大纲。

选择后的目标文件：

| 用户选择 | 生成目标 |
| ---- | ---- |
| `cursor` | `.cursor/rules/*.mdc` |
| `codex` | `AGENTS.md` |
| `claude` | `CLAUDE.md` |

选择 `cursor` 时，遵循下方 Cursor Rules 产出标准。

选择 `codex` 或 `claude` 时，应复用同一份项目事实和规则内容，但转换为对应工具常用的单文件规则格式；不要生成 `.cursor/rules`。

## Cursor Rules 产出标准

产出必须符合主流 Cursor Rules `.mdc` 格式：

- 文件放在 `.cursor/rules/`。
- 文件名使用 kebab-case，如 `project-overview.mdc`。
- 每个规则文件必须包含 YAML frontmatter。
- `description` 必须说明规则用途和适用场景。
- `globs` 尽量精准；只有全局规则才使用 `"**/*"`。
- `alwaysApply: true` 只用于所有任务都必须加载的规则。
- 内容应面向 AI 执行，使用明确的“必须 / 禁止 / 优先”规则。
- rules 优先短、准、可执行；不要写成完整规范文档。
- 少写背景解释，多写命令、约束、目录位置和禁止事项。

```markdown
---
description: 项目编码规范，适用于该项目的所有代码生成和修改
globs:
  - "**/*"
alwaysApply: true
---

# Project Conventions

## Project Overview

## File Tree

## Coding Rules

## Style Rules

## AI Workflow
```

推荐拆分为多个规则文件：

```text
.cursor/rules/
├── project-overview.mdc      # 技术栈、目录树、项目结构、运行方式
├── coding-conventions.mdc    # JS/TS/Vue/React 编码规范
├── style-conventions.mdc     # CSS/LESS/SCSS/stylelint 规范
└── ai-workflow.mdc           # AI 修改代码时必须遵守的工作方式
```

拆分文件时使用更精准的 globs：

```markdown
---
description: 样式代码规范，适用于 CSS、LESS、SCSS、Vue style block 和 PostCSS 文件
globs:
  - "**/*.css"
  - "**/*.less"
  - "**/*.scss"
  - "**/*.vue"
alwaysApply: false
---
```

小项目可以合并为单文件：

```text
.cursor/rules/project-conventions.mdc
```

## 必须写入的 AI 工作方式

生成的 rules 中必须包含这些约束：

- 修改代码前先观察同目录已有写法，优先沿用项目现有模式。
- 优先复用项目已有组件、工具函数、请求封装、状态模块和类型定义。
- 不直接引入新的框架、状态库、请求库、样式方案，除非用户明确要求。
- 不绕过项目已有 ESLint、Prettier、Stylelint、TypeScript 配置。
- 不为了现代化重构无关老代码。
- 新增代码应放在项目已有目录职责对应的位置。
- 如果项目已有测试模式，新增或修改核心逻辑时应补充同风格测试。

## 写入策略

写入前必须先完成“目标 AI 选择门禁”。根据用户选择检查对应目标：

- `cursor`：检查 `.cursor/rules/` 下已有 `.mdc` / `.md` 文件。
- `codex`：检查仓库根目录是否已有 `AGENTS.md`。
- `claude`：检查仓库根目录是否已有 `CLAUDE.md`。

写入规则：

- 如果目标不存在，创建目录或文件并写入推荐规则。
- 如果目标存在，先读取已有内容。
- 不覆盖用户已有规则。
- 若可以安全补充，则新建缺失规则文件。
- 若存在冲突，向用户展示建议内容和冲突点，由用户决定是否合并。
- 用户未选择目标 AI 前，禁止写入任何规则文件。

## 最终摘要

完成后向用户输出：

- 已识别技术栈和包管理器。
- 已分析的关键文件和目录。
- 项目文件树与用途说明。
- 用户选择的目标 AI：`cursor` / `codex` / `claude`。
- 已生成或建议生成的规则文件。
- Stylelint / ESLint / Prettier / TypeScript 的约束摘要。
- 未确认项和需要用户确认的项目约定。

## 质量标准

规则必须具体、可执行、基于项目证据，并保持足够精简。

生成 rules 时优先写：

- AI 修改代码前必须看的项目事实。
- AI 生成代码时必须遵守的约束。
- 可直接运行的校验命令。
- 常见错误的修复方式。

生成 rules 时不要写：

- 大段背景解释。
- 完整规范手册。
- 和代码生成无关的项目介绍。
- 已经能从配置文件自动推断的细节全文。

避免：

- 泛泛而谈的“保持代码整洁”。
- 与项目 Lint 或真实代码风格冲突的建议。
- 没有证据支撑的强制规则。
- 过长的大而全规范。
- 把外部最佳实践当成项目既有规范。
