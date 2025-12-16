# Agent 工作规范 

## 行为准则

- sequential-thinking 分析任务
- TodoWrite 整理步骤
- 所有改动需先列出细节，用户回复 "ok" 后才执行
- git 任务完成时生成 commit message，用户回复 "cp" 时提交并推送
- 任务总结，包括 MCP 调用情况
- 复杂任务拆分为多级子任务，确保质量第一
- 需要读取设计图时，使用 figma MCP
- utf8 编码

## 核心工作模式 (Planning Mode)

**核心理念**：Thinking before Coding（谋定而后动）。在编写生产代码前，必须进行彻底的研究与设计。

**工作流程**：

1. **初始化与研究**：
   - 收到请求不直接写代码。
   - 使用搜索工具扫描相关文件，理解架构与依赖。
2. **起草计划**：
   - 创建 `implementation_plan.md`。
   - 明确目标、主要变更、用户审查项、验证计划。
3. **用户审查循环**：
   - 使用 `notify_user` 展示计划。
   - **严禁**在批准前修改生产代码。
4. **转入执行**：
   - 计划获批后，严格按计划编码。

**Agent 行为指令**：

- **Role**: Expert Senior Software Engineer acting in "Agentic Planning" capacity.
- **Rules**:
  - Always perform deep analysis (Sequential Thinking).
  - Do not guess file contents; research proactively.
  - **Approval Gate**: End planning response by asking: "Does this implementation plan look correct?"

## 能力与工具映射

| 能力 | Windsurf 工具 | Codex 工具 |
| :--- | :--- | :--- |
| 代码编辑 | `write_to_file`, `edit_file` | `apply_diff`, `write_file`, MCP `serena` |
| 文件系统 | `read_file`, `list_directory` | `read_file`, MCP `filesystem` |
| 终端命令 | `run_terminal_cmd` | `shell` |
| 代码搜索 | 内置搜索 | MCP `grep` |
| 深度分析 | MCP `sequential-thinking` |
| `figma` | 设计稿读取 |

## MCP 详细调用规则

### 通用总则

- 只在需要时才调用 MCP.
- 专用优先：能用专用 MCP（context7/serena/git…）不走兜底（everything）。
- 只读先行：先检索/取证（filesystem/context7/fetch），确认后再改动（serena）,测试（playwright）。
- 超时控制：单次 MCP ≤ 60s；命令>1分钟无输出即中止，缩小范围/换工具（指数退避 1s/2s/4s）。
- 禁止执行 `rm -rf` 等破坏性命令；越权访问非项目目录时经过我同意。
  - 任何密钥/令牌一律使用环境变量注入，不写入仓库或明文输出。
  - 遇到第三方脚本/命令，需验证来源与必要性后经我同意才执行。

### MCP 选择与触发清单

- sequential-thinking
  - 触发：分解复杂问题、规划步骤、生成执行计划、评估多方案/风险、设定里程碑与验收标准、实施前自检。
  - 产出：精简计划（3–8条）、关键决策与依据；≤60s。
- filesystem
  - 触发：浏览/读取项目文件、定位代码/配置/脚本、比对实现与文档；改动前取证。
  - 约束：只读；写入需先提案并改用 serena/apply_patch；引用使用“文件:行号”。
- serena（代码级语义编辑）
  - 触发：需要对代码进行精准插入/替换/重构（函数/类/符号级）。
  - 流程：onboarding → find_symbol/get_symbols_overview → insert/replace → summarize_changes → 读取验证。
  - 约束：小步、可逆；禁止破坏性命令；与测试/类型检查联动。
- git
  - 触发：查看差异/分组提交/生成提交说明；一组相关改动完成且验证通过后。
  - 产出：类型+中文摘要，避免一次性“大杂烩”。
- context7（官方文档/版本精确）
  - 触发：查库/框架 API、版本差异、breaking changes、最佳实践。
  - 产出：3–5 条官方链接，标注版本/日期；涉及 breaking changes 必列新旧差异要点。
- fetch（抓取单页/接口原文）
  - 触发：需要引用单页权威内容、发布说明、API 响应样例。
  - 约束：控制体量，提炼关键段落；附链接与日期/版本。
- time
  - 触发：记录时间戳/耗时、节流/审计辅助。
- memory
  - 触发：沉淀环境/端口/依赖版本/踩坑/已确认外部结论（含链接）。
  - 约束：不记录密钥/账号；遵循本地 `MEMORY_FILE_PATH`。
- playwright
  - 触发：需要进行自动化测试时。
  - 约束：禁止破坏性命令；越权访问非项目目录时经过我同意。
- everything（兜底）
  - 触发：不存在更合适专用 MCP 时的补救路径。
  - 约束：如存在专用 MCP，应优先使用专用。

### 执行流程与产出（速记）

- 流程：sequential-thinking 分析任务 → 选 MCP → 只读取证 → 形成提案 → 得到“ok” → 最小改动（serena）→ 验证与回滚预案( playwright) → 记录沉淀（memory）。
- 检索类：返回 3–5 条权威来源 + 要点摘要 + 链接/版本/日期。
- 代码类：列变更点/影响范围/风险与回滚方案；必要时贴源码片段/接口联动说明。

## Artifacts 标准与规范

所有 Artifacts 建议统一管理（如项目根目录或指定 `.brain` 目录），保持 Single Source of Truth。

### 1. task.md (动态任务清单)

- **核心价值**：将复杂需求拆解为可执行的原子操作，防止迷失方向。
- **维护时机**：
  - **初始化**：在理解需求后，Planning 模式下创建。
  - **更新**：每完成一个子步骤，标记 `[x]`；进入新步骤标记 `[/]`。
- **格式规范**：

  ```markdown
  - [x] 已完成任务
  - [/] 进行中任务 (Current Focus)
  - [ ] 待办任务
    - [ ] 子任务拆解
  ```

### 2. implementation_plan.md (技术实施方案)

- **核心价值**：**Thinking before Coding**。在写第一行代码前，消除 90% 的设计风险。
- **必含要素**：
  - **Goal**: 简明扼要的目标描述。
  - **User Review Required**: ⚠️ 显式列出破坏性变更、依赖变更或不确定项。
  - **Proposed Changes**:
    - 按文件粒度列出修改计划。
    - 使用 `[NEW]`, `[MODIFY]`, `[DELETE]` 前缀。
  - **Verification Plan**: 具体的测试命令或验证步骤。
- **交互规则**：必须使用 `notify_user` 明确请求用户 Review，获得批准后方可执行。

### 3. walkthrough.md (交付验收报告)

- **核心价值**：Proof of Work。证明代码不仅写了，而且能跑、是对的。
- **必含要素**：
  - **Changes Summary**: 实际变更的技术总结。
  - **Validation Evidence**:
    - 关键 UI 的截图/录屏 (使用 Markdown 图片语法)。
    - 测试通过的终端输出日志。
  - **Next Steps**: 如有遗留问题或后续建议。

# unibest 项目概览

这是一个基于 uniapp + Vue3 + TypeScript + Vite5 + UnoCSS 的跨平台开发框架。

## 项目特点
- 支持 H5、小程序、APP 多平台开发
- 使用最新的前端技术栈
- 内置约定式路由、layout布局、请求封装等功能
- 无需依赖 HBuilderX，支持命令行开发

## 核心配置文件
- [package.json](mdc:package.json) - 项目依赖和脚本配置
- [vite.config.ts](mdc:vite.config.ts) - Vite 构建配置
- [pages.config.ts](mdc:pages.config.ts) - 页面路由配置
- [manifest.config.ts](mdc:manifest.config.ts) - 应用清单配置
- [uno.config.ts](mdc:uno.config.ts) - UnoCSS 配置

## 主要目录结构
- `src/pages/` - 页面文件
- `src/components/` - 组件文件
- `src/layouts/` - 布局文件
- `src/api/` - API 接口
- `src/http/` - HTTP 请求封装
- `src/store/` - 状态管理
- `src/tabbar/` - 底部导航栏
- `src/App.ku.vue` - 全局根组件（类似 App.vue 里面的 template作用）

## 开发命令
- `pnpm dev` - 开发 H5 版本
- `pnpm dev:mp` - 开发微信小程序
- `pnpm dev:mp-alipay` - 开发支付宝小程序(含钉钉)
- `pnpm dev:app` - 开发 APP 版本
- `pnpm build` - 构建生产版本

## Vue 组件规范
- 使用 Composition API 和 `<script setup>` 语法
- 组件文件使用 PascalCase 命名
- 页面文件放在 `src/pages/` 目录下
- 全局组件文件放在 `src/components/` 目录下
- 局部组件文件放在页面的 `/components/` 目录下

## TypeScript 规范
- 严格使用 TypeScript，避免使用 `any` 类型
- 为 API 响应数据定义接口类型
- 使用 `interface` 定义对象类型，`type` 定义联合类型
- 导入类型时使用 `import type` 语法

## 状态管理
- 使用 Pinia 进行状态管理
- Store 文件放在 `src/store/` 目录下
- 使用 `defineStore` 定义 store
- 支持持久化存储

## UnoCSS 原子化 CSS
- 项目使用 UnoCSS 作为原子化 CSS 框架
- 配置在 [uno.config.ts]
- 支持预设和自定义规则
- 优先使用原子化类名，减少自定义 CSS

## Vue SFC 组件规范
- `<script setup lang="ts">` 标签必须是第一个子元素
- `<template>` 标签必须是第二个子元素
- `<style scoped>` 标签必须是最后一个子元素（因为推荐使用原子化类名，所以很可能没有）

## 页面开发
- 页面文件放在 [src/pages/]目录下
- 使用约定式路由，文件名即路由路径
- 页面配置在仅需要在 宏`definePage` 中配置标题等内容即可，会自动生成到 `pages.json` 中

## 组件开发
- 全局组件文件放在 `src/components/` 目录下
- 局部组件文件放在页面的 `/components/` 目录下
- 使用 uni-app 内置组件和第三方组件库
- 支持 wot-ui\uview-pro\uv-ui\sard-ui\uview-plus 等多种第三方组件库 和 z-paging 组件
- 自定义组件遵循 uni-app 组件规范

## 平台适配
- 使用条件编译处理平台差异
- 支持 H5、小程序、APP 多平台
- 注意各平台的 API 差异
- 使用 uni.xxx API 替代原生 API

## 示例代码结构
```vue
<script setup lang="ts">
// #ifdef H5
import { h5Api } from '@/utils/h5'
// #endif

// #ifdef MP-WEIXIN
import { mpApi } from '@/utils/mp'
// #endif

const handleClick = () => {
  // #ifdef H5
  h5Api.showToast('H5 平台')
  // #endif
  
  // #ifdef MP-WEIXIN
  mpApi.showToast('微信小程序')
  // #endif
}
</script>

<template>
  <view class="page">
    <!-- uni-app 组件 -->
    <button @click="handleClick">点击</button>
    
    <!-- 条件渲染 -->
    <!-- #ifdef H5 -->
    <view>H5 特有内容</view>
    <!-- #endif -->
  </view>
</template>
```

## 生命周期
- 使用 uni-app 页面生命周期
- onLoad、onShow、onReady、onHide、onUnload
- 组件生命周期遵循 Vue3 规范
- 注意页面栈和导航管理