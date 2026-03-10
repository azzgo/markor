# Markor AI 改写/续写设计文档

本文档用于记录 Markor 编辑器新增 AI 改写/续写功能的需求、方案与实施任务清单，便于后续分阶段实现。

## 需求概述

- 通过可配置工具栏新增 AI 改写、AI 续写两个入口。
- 支持 OpenAI 兼容接口地址与 API KEY 的用户配置。
- system prompt 为全局配置，放在设置页面，不在按钮对话框中选择。
- 上下文注入不做外部文件选择；仅在当前文档内按“规则”取上下文。
- 同一文档内缓存“上一次的上下文规则”，每次运行时动态提取当前文本。
- AI 请求为阻塞式 UI，可取消（中断请求）。

## 现有可扩展点（基于代码结构）

- ActionButton 体系是编辑器工具栏扩展点：
  - `ActionButtonBase` 定义动作列表与渲染。
  - 各格式 ActionButtons（如 `MarkdownActionButtons`、`PlaintextActionButtons`）定义具体按钮。
  - `ActionButtonSettingsActivity` 负责按钮排序与启用。
- 结论：AI 改写/续写应作为新增 `ActionItem` 接入 ActionButton 体系，自动进入“可配置工具栏”。

## 功能方案

### 1) 工具栏入口

- 在目标格式的 ActionButtons 中新增两项 `ActionItem`：
  - AI 改写
  - AI 续写
- 点击动作进入统一对话框（或 DialogFactory 入口）。

### 2) 全局设置（system prompt 与 API 配置）

- 新增 AI 设置分组：
  - API Base URL
  - API KEY
  - System Prompt（单一全局）
- 在工具栏配置页面增加“AI 设置”入口，跳转设置页。
- 按钮对话框内不再选择 system prompt。

### 3) 上下文规则缓存（文档级内存）

- 缓存对象：`ContextRule`
  - 规则类型：选区 / 全文 / 光标段落 / 最近 N 段
  - N 值（若适用）
- 缓存位置：`DocumentActivity` 生命周期内存字段，不持久化。
- 执行时根据规则动态提取当前文本片段作为上下文。

### 4) 阻塞式 UI 与可取消请求

- 触发后显示阻塞式进度对话框。
- 提供“取消”按钮，调用 OkHttp `Call.cancel()`。
- 完成后关闭对话框，并应用结果。

### 5) 文本应用策略

- 改写：优先替换选区；无选区时提示是否改写全文。
- 续写：在光标处插入生成文本。

## 交互流程（摘要）

1. 用户点击 AI 改写/续写按钮。
2. 若无 ContextRule，弹出规则选择对话框并缓存。
3. 阻塞式进度对话框显示，可取消。
4. 请求成功后替换/插入文本；失败或取消则不改动。

## 技术设计

### 组件与职责

- `AiClient`
  - 负责 OpenAI 兼容 Chat Completions 请求。
  - 暴露取消能力。
- `AiRequestDialog`
  - 负责选择/确认上下文规则。
  - 调起 AI 请求并展示阻塞式进度。
- `ContextRuleManager`
  - 文档级缓存当前规则。
  - 根据规则动态提取上下文文本片段。

### OpenAI 兼容接口建议

- Endpoint: `POST {baseUrl}/v1/chat/completions`
- Headers: `Authorization: Bearer {apiKey}`
- Body:
  - system: 全局 system prompt
  - user: 目标文本 + 当前上下文片段

## 分阶段任务清单

### 阶段一：工具栏入口与设置

- 新增 AI 改写/续写 ActionItem（Markdown + Plaintext）。
- 新增 AI 设置分组与字段（API Base URL / API KEY / System Prompt）。
- 在工具栏配置页面添加“AI 设置”跳转入口。

### 阶段二：上下文规则与对话框

- 新增 ContextRule 结构与缓存逻辑（文档级内存）。
- 对话框支持选择规则并保存。
- 执行时按规则提取上下文。

### 阶段三：AI 请求与取消

- 实现 AiClient（OpenAI 兼容接口）。
- 阻塞式进度 UI + 取消请求。
- 错误处理与提示。

### 阶段四：文本应用与细节优化

- 改写/续写文本应用策略完善。
- 字符长度限制与提示。
- 体验优化与文案完善。

## 关键决策记录

- 工具栏入口采用 ActionButton 体系。
- system prompt 为全局配置（设置页），按钮对话框不再选择。
- 上下文为“规则缓存”，非外部文件注入。
- UI 阻塞式，提供取消。
