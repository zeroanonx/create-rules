# Frontend Code Review Standards

> 审查对照本文档；违规按 P0 / P1 / P2 标注。与 [SKILL.md](../SKILL.md) 配套。

---

## 零、通用强制项（优先检查）

| # | 规则 | CR 要点 |
|---|------|---------|
| 1 | 不要滥用 `async/await`，使用处必须 `try/catch` | 裸 await、无错误边界 → P0 |
| 2 | TS 项目禁止 `any`；API 入参/出参必须定义类型 | 缺失 interface/type → P0 |
| 3 | 禁止魔法值 | 未提取常量/枚举 → P1 |
| 4 | 遵守样式属性顺序 | 有 stylelint 的项目必须过；无校验老项目可忽略 |
| 5 | 第三方 API 必须二次封装 | 业务层直接调 SDK → P1 |
| 6 | Vue 3 使用 `<script setup>` 语法糖 | Options API 新代码 → P1 |
| 7 | 缩进：2 空格或 1 Tab（与项目一致） | 混用或错误缩进 → P2 |
| 8 | 生命周期钩子内不写业务逻辑 | 复杂逻辑应抽函数/composable → P1 |

---

## 一、编程规约

### （一）命名规范

#### 1.1.1 命名严谨性

- **禁止**拼音、中文命名
- 使用英文全拼或业界通用缩写（如 `DNA`、`CPU`、`URL`）
- **禁止**不规范缩写（如 `AbstractClass` → `AbsClass`）

#### 1.1.2 项目命名

- 全小写，中线分隔

```text
✅ ad-management-portal
❌ adManagementPortal
```

#### 1.1.3 JS/CSS 等非 Vue 组件目录

- 小驼峰 `lowerCamelCase`
- 复数概念用复数形式；缩写不加复数

```text
✅ src/utils/slsTracker.ts
❌ src/views/ad-manage/ad-plans/add/index.vue  （路径层级过深且命名混乱）
```

#### 1.1.4 常量命名

**1）全大写 + 下划线**

```typescript
const MAX_LENGTH = 200

export const COMMON_FLAG = {
  yes: 1,
  no: 2
}
```

**2）可见性 / 进行中状态：`is` + 动词/形容词**

```typescript
isShow, isVisible, isLoading, isConnecting, isValidating, isRunning, isListening
```

**3）属性状态类**

```typescript
disabled, editable, clearable, readonly, expandable, checked,
enumerable, iterable, clickable, draggable
```

#### 1.1.5 方法命名

- 小驼峰
- 多数情况：**动词 + 名词**

```typescript
function getUserDetail() {}
```

**常用动词**：`get/set`、`add/remove`、`create/destroy`、`start/stop`、`open/close`、`read/write`、`load/save`、`update/delete`、`find/search`、`submit/cancel` 等（完整列表见团队文档附录）。

**事件命名**：`on*` / `handle*`

```typescript
onSubmit, onKeydown, handleSizeChange, handlePageChange
```

---

### （二）注释规范

| 类型 | 要求 |
|------|------|
| 方法 | 必填 `@function`、`@param`；建议 `@description`、`@return`、`@example` |
| 常量 | `@constant` |
| 枚举 | `@enum` |
| 接口/API | `@api` |
| HTML | 区块注释 `<!-- S 模块名 -->` … `<!-- E 模块名 -->`（老项目可用 Start/End） |
| CSS | 单行 `/* */`；模块用分隔线；文件头可用 `@description` `@author` `@date` |

---

### （三）HTML 规范（Vue Template 同样适用）

#### 1.3.1 文档类型

- 推荐 HTML5：`<!DOCTYPE html>`

#### 1.3.2 标签

- 合法闭合、嵌套正确，标签名小写
- 语义化优先（`header`/`footer`/`nav` 等），避免无意义 `div` 堆砌
- 自定义属性以 `data-` 开头
- **禁止**随意用 `id` 做样式（除非功能/组件必需）

#### 1.3.3 引号

- Template 属性使用**双引号** `"`

```html
✅ <div class="box"></div>
❌ <div class='box'></div>
```

---

### （四）CSS 规范

#### 1.4.1 命名

- **禁止**语义不明的缩写类名（如 `.red`、`.fw-800` 单独表意）
- **推荐**语义化组合：`.heavy`、`.color-red`

#### 1.4.2 选择器

- 避免标签选择器、避免 `#id` 污染全局
- 优先直接子选择器 `>` 而非深层后代

```css
✅ .content > .title { font-size: 2rem; }
❌ .content .title { font-size: 2rem; }  /* 过宽 */
❌ span { color: red; }
❌ #header { margin: 0; }
```

#### 1.4.3 精简写法

- `0` 不带单位；颜色能缩写则缩写；可合并的属性合并
- `rgba` 逗号后无空格（按项目 Prettier/stylelint 为准）

```css
✅ margin: 0; padding: 1em 2em; background: #fff; color: rgba(255,255,255,.5);
❌ margin: 0em; padding-top/right/bottom/left 分拆冗余
```

---

### （五）LESS / SCSS 规范

#### 1.5.1 代码组织

- 公共样式放 `assets/styles`（如 `variables.less`、`common.less`）
- 文件内顺序：`@import` → 变量 → 样式规则

#### 1.5.2 嵌套

- 嵌套层级不宜过深（建议 ≤ 3 层）

```less
✅ .main-title { .name { color: #fff; } }
❌ .main { .title { .name { .text { ... } } } }
```

---

### （六）JavaScript 规范（StandardJS 风格）

#### 1.6.1 字符串

- JS 代码中统一**单引号** `'`
- Template 中仍用双引号（见 HTML 规范）
- 长字符串用 `+` 拼接，**禁止**行尾 `\` 续行
- 程序化拼接优先**模板字符串**

```javascript
✅ const str = `ab${test}`
❌ const str = 'a' + 'b' + test  // 多段静态拼接
❌ let str = "foo"
```

#### 1.6.2 对象

- 字面值 `{}` 创建，禁止 `new Object()`
- 属性/方法简写；简写与非简写分组

#### 1.6.3 ES6+

- 使用 `const`/`let`，**禁止** `var`

#### 1.6.4 括号

- 控制语句必须使用 `{}`，即使单行

```javascript
✅ if (condition) { doSomething() }
❌ if (condition) doSomething()
```

#### 1.6.5 undefined 判断

```javascript
✅ if (typeof person === 'undefined') { ... }
❌ if (person === undefined) { ... }
```

#### 1.6.6 条件与循环

- 条件/循环嵌套**最多三层**；优先早 return、逻辑合并

#### 1.6.7 解构

- 多属性读取优先解构

#### 1.6.8 函数

- 禁止 `arguments`，使用 rest `...args`
- 禁止修改参数；默认值用参数默认值 `opts = {}`

#### 1.6.9 变量声明

- 一行一个 `let`/`const`

#### 1.6.10 遍历

- 数组：推荐普通 `for`（性能敏感场景）
- `for...in` 遍历对象必须 `hasOwnProperty`

---

### （七）图片规范

- PNG/JPG：小写 + 下划线；多倍图 `@2x` / `@3x`

```text
step_cut.png
step_cut@2x.png
step_cut@3x.png
```

- PNG 须压缩（如 tinypng.com）

---

## 二、Vue 3 项目规范

### （一）编码基础

#### 2.1.1 组件规范

**1）命名**

- 多单词（≥2），大驼峰 PascalCase
- 基础组件：`Base` 或项目约定前缀（如 `Fs`）开头
- 紧密耦合子组件：父名前缀，如 `TodoList` + `TodoListItem`
- 单文件：`TodoItem.vue`；拆分样式时目录 `TodoList/index.vue` + `index.scss`
- 详情等子页面放在父页面目录下

```text
✅ components/TodoList/index.vue
✅ components/TodoListItem.vue
✅ views/UserList/UserDetail.vue
❌ components/TodoList.vue + TodoItem.vue（子组件未加前缀）
❌ UProfOpts.vue（缩写）
```

**2）JSX/TSX vs Template**

- JSX/TSX：`<MyComponent />` PascalCase + 自闭合
- Template：`<my-component />` kebab-case

**3）Props**

- camelCase；必须声明 `type`
- 必须注释含义；`required` 或 `default` 二选一
- 有枚举/范围时加 `validator`

**4）样式作用域**

- 组件样式必须 `scoped`（或 CSS Modules 等等价方案）

**5）属性换行**

- 属性较多时多行排列，避免单行过长

#### 2.1.2 模板表达式

- 模板只放简单表达式；复杂逻辑 → `computed` / `methods`

#### 2.1.3 指令缩写

- `:`、`@`、`#` 代替 `v-bind:`、`v-on:`、`v-slot:`

#### 2.1.4 单文件组件标签顺序

```text
✅ template → script → style
❌ template → style → script
```

#### 2.1.5–2.1.6 列表与显隐

- `v-for` 必须 `:key`
- 频繁切换 → `v-show`；少变条件 → `v-if`

#### 2.1.7 Vue Router

- 页面间传参优先 **query/params**，不依赖 vuex 持久化（刷新丢失）
- 路由组件 **懒加载** `() => import('...')`
- `path`、`name` 小驼峰 `lowerCamelCase`

---

### （二）目录规范

#### 2.2.1 前后端命名统一

- 业务域单词与后端一致（如 `activity` → router/store/api 均用 `activity`）

#### 2.2.2 推荐目录结构

```text
src/
├── api/           # 接口
├── assets/        # 静态资源
├── components/    # 公共组件
├── config/
├── constants/     # 枚举、全局常量
├── hooks/
├── plugins/
├── router/
├── stores/        # pinia/vuex
├── types/
├── layout/
├── views/
└── utils/
```

#### 2.2.3 其他

- 尽量避免手动操作 DOM（数据驱动为主）
- 及时删除无用代码、`console` 调试语句

#### 2.2.4 `<script setup>` 内代码顺序

```typescript
// 1. import
// 2. defineOptions / defineProps / defineEmits
// 3. 变量声明
// 4. hooks
// 5. computed、watch
// 6. 方法
// 7. onMounted 等生命周期
// 8. defineExpose
```

---

## 三、接口相关

### 3.1.1 常用请求字段

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| page | int | 1 | 当前页码 |
| pageSize | int | 10 | 每页条数；移动端上拉可省略 |
| totalCount | int | 0 | 总条数；PC 分页必填 |
| list | Array | [] | 列表统一字段名；多列表时前缀如 `storeList` |
| username / password / oldPassword / newPassword | string | "" | 账号相关 |
| verificationCode | string | "" | 验证码 |
| phone / email / headImage | string | "" | 用户资料 |
| address / province / city / district | string | "" | 地址 |
| hasPermission / isAdmin | int | 1 | 1=是 2=否 |
| time | string | yyyy-MM-dd HH:mm:ss | 单时间点 |
| startTime / endTime | string | | 时间段 |
| status | int | 1 | 状态；**0 禁止有业务含义** |
| searchContent | string | "" | 列表搜索 |
| version | string | 1.0.0 | 版本号 |
| appinfo | string/object | | App 专用，不参与验签 |
| sign | string | | MD5 签名串 |

### 3.1.2 统一响应结构

| 字段 | 类型 | 说明 |
|------|------|------|
| errorCode | int | 200 为成功；业务错误据此分支 |
| errorMsg | string | 错误文案，可直接展示 |
| data | Object | 成功时必须为对象；列表/实体均在 data 内 |
| success | boolean | 业务是否成功 |

### 3.1.3 列表响应

| 字段 | 必填 | 说明 |
|------|------|------|
| list | 是 | 列表数据 |
| totalCount | PC 必填 | 分页；移动端上拉加载可省略 |

---

## 四、CR 快速检查清单

审查时可按模块勾选：

- [ ] 无 `any`、API 有完整类型
- [ ] 无魔法值、常量/枚举合理
- [ ] async/await 有 try/catch
- [ ] 第三方 SDK 已封装
- [ ] Vue3 setup + 标签顺序 + scoped
- [ ] 生命周期无大块业务逻辑
- [ ] 命名/目录/组件符合上文规范
- [ ] 接口字段命名与响应结构符合第三节
- [ ] 样式：选择器、嵌套、顺序（若项目有 lint）
- [ ] 无调试代码、无无用注释块

---

## 五、正例 / 反例索引

| 场景 | ✅ | ❌ |
|------|----|----|
| 项目名 | `user-center-admin` | `userCenterAdmin` |
| 工具文件 | `dateFormatter.ts` | `date_formatter.ts` |
| JS 字符串 | `'hello'` | `"hello"` |
| Template 属性 | `class="box"` | `class='box'` |
| 条件语句 | `if (x) { fn() }` | `if (x) fn()` |
| 组件样式 | `<style scoped>` | 全局污染 |
| 列表字段 | `data.list` | `data.items` / `data.rows` 混用 |
| 状态 0 | 不使用 | `status: 0` 表示业务态 |

---

*文档版本：与 code-review SKILL 配套使用。审查时结合项目实际 lint/tsconfig 配置，老项目无 stylelint 时可忽略样式顺序项。*
