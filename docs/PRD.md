# OpenDesign - AI-Native UI Prototyping Tool

## Product Requirements Document (PRD)

**Version**: 1.0.0  
**Date**: 2026-04-24  
**Status**: Draft  
**Author**: Hyper Vapor (with Claude / Hermes Agent)  

---

## 1. 项目概述

### 1.1 产品名称
**OpenDesign**（暂定，开源 AI 设计工具）

### 1.2 一句话定义
OpenDesign 是中国首个开源的、支持本地模型的、对话式 AI UI 原型工具。用户通过自然语言描述需求，AI 生成可交互的 HTML/React 原型，支持对话式迭代编辑、设计系统管理和一键导出生产级代码。

### 1.3 为什么要做这个项目

#### 市场空白
- Claude Design（2026.4 发布）：闭源、绑定 Claude Pro、国外产品
- Google Stitch：Google Labs 实验项目，不稳定
- v0.dev：Vercel 闭源产品，免费额度有限
- **中国厂商（DeepSeek/智谱/Kimi/MiniMax）：完全空白，没有任何 AI 设计工具**
- 开源方案（OpenUI）：功能单薄，缺乏设计系统、多屏原型、PPT 导出等核心能力

#### 技术条件成熟
- 开源多模态模型（DeepSeek V4、Qwen2.5-VL、GLM-4V）的视觉理解能力已足够
- 开源代码生成模型（DeepSeek V4、Qwen3、GLM-5.1）的前端代码质量优秀
- Tailwind CSS + shadcn/ui 成为 LLM 生成 UI 的事实标准
- Sandpack 等沙箱渲染方案成熟

#### 与现有赛道天然契合
- 用户在 AutoDL 部署 vLLM（DeepSeek/Qwen/GLM）
- 可主打"可私有化部署的 AI 设计工具"，满足企业安全合规需求
- 国产替代 + 本地化（中文字体、排版、间距）有巨大空间

### 1.4 目标用户

| 用户角色 | 需求 | 使用场景 |
|----------|------|----------|
| **独立开发者** | 快速出原型、验证想法 | 做 side project、创业 MVP |
| **产品经理** | 不用学 Figma 就能做高保真原型 | 写 PRD 时附交互 demo |
| **前端工程师** | 快速生成 UI 代码骨架 | 从设计稿起步开发 |
| **企业设计团队** | 私有化部署、数据不出域 | 内部设计系统管理 |
| **AI 爱好者** | 体验本地模型做设计 | 在 AutoDL 上跑自己的设计工具 |

### 1.5 核心卖点

1. **模型自由**：支持任意 LLM（OpenAI/Claude/DeepSeek/Qwen/GLM/本地 vLLM），不绑定任何厂商
2. **本地优先**：可私有化部署，满足企业安全合规需求
3. **对话式设计**：不用学 Figma，说话就能做设计
4. **中文优化**：原生支持中文字体、排版、间距规则
5. **开源开放**：Apache 2.0 协议，社区驱动演进

---

## 2. 产品定位

### 2.1 不是什么
- ❌ **不是 Figma 替代品**：不做专业设计师的精细画布工具
- ❌ **不是静态图片生成器**：不是 DALL-E/Midjourney 生设计图
- ❌ **不是低代码平台**：不拖拽组件，靠对话驱动

### 2.2 是什么
- ✅ **AI 原型加速器**：从想法到可点击原型，分钟级完成
- ✅ **代码生成器**：输出的是真实可执行的 HTML/React 代码
- ✅ **设计系统管理器**：管理 Design Tokens，保证跨页面一致性
- ✅ **Coding Agent 搭档**：生成 design.md，无缝对接 Claude Code / OpenCode

### 2.3 竞品定位图

```
                        专业设计能力
                              ↑
                              │
       Figma ─────────────────┼──────────────── Adobe XD
                              │
   低代码平台 ────────────────┼─────────────── v0.dev
       │                      │                         │
       │                      │                         │
       │               【OpenDesign】 ←── 目标定位      │
       │                      │                     Claude Design
       │                      │                     Google Stitch
       │                      │
  静态图片生成 ←──────────────┼─────────────── AI 对话生成
  (DALL-E/Midj)              │
                              ↓
                        快速原型能力
```

---

## 3. 技术架构

### 3.1 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                      前端层 (Frontend)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  对话界面    │  │  画布渲染    │  │  设计面板    │  │
│  │  (Chat UI)   │  │  (Canvas)    │  │  (Inspector) │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  Next.js 15 + React 19 + Tailwind CSS + shadcn/ui      │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                      API 层 (Backend)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  会话管理    │  │  Prompt      │  │  代码处理    │  │
│  │  (Session)   │  │  Engine      │  │  (Parser)    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  FastAPI / Node.js + 多 Provider LLM Router             │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                      模型层 (LLM Providers)              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ OpenAI   │ │ Anthropic│ │ DeepSeek │ │  本地    │  │
│  │  GPT-x   │ │ Claude-x │ │   V4     │ │  vLLM    │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│  统一接口: /chat/completions (兼容 OpenAI SDK)          │
└─────────────────────────────────────────────────────────┘
```

### 3.2 核心模块

#### 3.2.1 输入解析模块 (Intent Parser)
- **输入类型**: 自然语言文本、图片上传、截图粘贴、设计文件导入（Figma JSON）
- **处理方式**:
  - 文本: 直接送入 LLM 做意图提取
  - 图片: 调用多模态模型（Claude/GPT-4V/Qwen-VL/DeepSeek V4）做视觉理解
  - 设计文件: 解析 Figma JSON/Sketch 文件提取设计系统
- **输出**: 结构化 Design Intent JSON

#### 3.2.2 设计系统模块 (Design System Engine)
- **Design Tokens 管理**:
  - Colors (primary, secondary, background, text, border)
  - Typography (font family, sizes, weights, line heights)
  - Spacing (scale, base unit)
  - Border Radius
  - Shadows
- **导入方式**:
  - 从品牌指南/截图自动提取
  - 从现有设计文件导入
  - 用户手动定义
- **输出**: `design.md` / `design.json`（AI 可读格式）

#### 3.2.3 代码生成模块 (Code Generation Engine)
- **Prompt 工程系统**:
  - System Prompt: 定义前端最佳实践、Tailwind 规范
  - Context Prompt: 注入 Design Tokens、已有组件
  - User Prompt: 用户需求 + 迭代指令
- **输出格式**:
  - 单文件 HTML (iframe 渲染) — 快速预览
  - React + Tailwind 组件 — 生产代码
  - Vue/Svelte (可选扩展)
- **沙箱渲染**: Sandpack / 自研 iframe 沙箱 + CSP 策略

#### 3.2.4 迭代编辑模块 (Iterative Editor)
- **对话式修改**:
  - 自然语言指令 → 代码 diff（如"按钮圆一点"→`rounded-md`→`rounded-xl`）
  - 内联评论: 点击画布元素添加反馈
  - 属性面板: 滑块调整间距、颜色、尺寸
- **版本管理**: 每次生成为一个 checkpoint，可 rewind

#### 3.2.5 导出模块 (Export Engine)
- **格式支持**:
  - 独立 HTML 文件（含 CDN 依赖）
  - React + Tailwind 项目（组件拆分）
  - PPTX / PDF（营销物料）
  - Figma 文件（JSON 格式）
  - design.md（给 Coding Agent 消费）

### 3.3 技术栈选型

| 层级 | 技术 | 理由 |
|------|------|------|
| **前端框架** | Next.js 15 (App Router) + React 19 | SSR/SSG、生态成熟、Vercel 部署方便 |
| **UI 组件** | shadcn/ui + Radix UI |  headless + 可定制，与 LLM 代码输出兼容 |
| **样式** | Tailwind CSS v4 | LLM 生成类名最稳定，社区标准 |
| **状态管理** | Zustand + React Query | 轻量、TypeScript 友好 |
| **实时渲染** | Sandpack (CodeSandbox) | 开箱即用 iframe 沙箱 |
| **后端** | FastAPI (Python) | AI 工程友好，async 支持好 |
| **数据库** | PostgreSQL + Redis | 会话存储 + 缓存 |
| **LLM 接口** | litellm / 自研 Router | 统一多 provider 接口 |
| **部署** | Docker + Coolify / Railway | 易于自托管 |

---

## 4. 功能需求

### 4.1 Phase 1: MVP（核心功能，2-3 个月）

#### FR-1.1 对话式 UI 生成
- **描述**: 用户在聊天框输入需求，AI 生成可交互 UI 原型
- **输入示例**: "帮我做一个 SaaS 仪表盘的首页，要有侧边栏、数据卡片、折线图，风格参考 Linear，深色模式"
- **输出**: 单文件 HTML，在右侧画布实时渲染
- **Acceptance Criteria**:
  - [ ] 支持多轮对话，每次输出覆盖或新增
  - [ ] 生成代码包含完整的 HTML/CSS/JS，可直接在 iframe 运行
  - [ ] 渲染结果可点击、可交互（按钮有 hover 效果等）

#### FR-1.2 多模型支持
- **描述**: 用户可切换不同的 LLM provider
- **支持列表**（MVP 阶段）:
  - [ ] OpenAI (GPT-4o, GPT-4.1)
  - [ ] Anthropic (Claude Sonnet/Opus)
  - [ ] DeepSeek (V4)
  - [ ] 自定义 API Endpoint（OpenAI-compatible）
- **Acceptance Criteria**:
  - [ ] 在设置面板配置 API Key 和 Base URL
  - [ ] 对话中可随时切换模型
  - [ ] 未配置模型的功能禁用并提示

#### FR-1.3 代码实时预览
- **描述**: 沙箱环境实时渲染生成的代码
- **技术实现**:
  - [ ] Sandpack 或 iframe sandbox 渲染
  - [ ] CSP 安全策略防止 XSS
  - [ ] 错误捕获和显示（代码有错时展示错误信息）
- **Acceptance Criteria**:
  - [ ] 代码生成后 3 秒内渲染完成
  - [ ] 代码错误时显示友好错误提示，不崩溃
  - [ ] 支持移动端/平板/桌面三种视口切换

#### FR-1.4 对话式迭代编辑
- **描述**: 用户通过自然语言修改已生成的设计
- **示例对话**:
  - 用户: "把卡片间距调大一点"
  - 用户: "图表改成柱状图"
  - 用户: "加个用户头像在右上角"
- **Acceptance Criteria**:
  - [ ] LLM 理解修改意图并生成代码 diff
  - [ ] 修改后保留之前的设计决策（不丢失已确认的部分）
  - [ ] 支持 "撤销" 到上一个版本

#### FR-1.5 代码导出
- **描述**: 将生成的原型导出为可用代码
- **导出格式**:
  - [ ] 独立 HTML 文件（含 Tailwind CDN）
  - [ ] React + Tailwind 组件文件
- **Acceptance Criteria**:
  - [ ] 导出的代码可直接复制到项目中运行
  - [ ] React 导出包含正确的组件结构和导入语句

---

### 4.2 Phase 2: 增强功能（3-4 个月）

#### FR-2.1 设计系统管理
- **描述**: 统一管理 Design Tokens，保证跨页面一致性
- **功能**:
  - [ ] 创建/编辑 Design Token（颜色、字体、间距等）
  - [ ] 从截图/品牌指南自动提取（调用 VLM）
  - [ ] 导出 design.md（给 Coding Agent 消费）
  - [ ] 不同项目使用不同的 Design System
- **Acceptance Criteria**:
  - [ ] 修改 token 后，所有相关组件自动更新
  - [ ] design.md 可被 Claude Code / OpenCode 正确解析

#### FR-2.2 多屏原型
- **描述**: 支持多个页面串联，模拟完整应用流程
- **功能**:
  - [ ] 创建多个 Screen（首页、详情页、设置页等）
  - [ ] 设置 Screen 之间的跳转链接
  - [ ] 点击预览模式下可交互跳转
- **Acceptance Criteria**:
  - [ ] 至少支持 10 个 Screen 管理
  - [ ] 跳转逻辑在预览模式下正常工作

#### FR-2.3 截图/图片输入
- **描述**: 上传截图或手绘草图，AI 还原为可编辑原型
- **技术**: 多模态 LLM 理解图片内容 → 生成代码
- **Acceptance Criteria**:
  - [ ] 支持 JPG/PNG/WEBP 上传
  - [ ] 上传后生成与原图布局相似的可交互原型
  - [ ] 可在生成基础上继续对话修改

#### FR-2.4 PPT/营销物料生成
- **描述**: 生成演示文稿和营销物料
- **模板**:
  - [ ] 投资人路演稿（12 页标准结构）
  - [ ] 产品介绍长图
  - [ ] 数据报告仪表盘
- **导出格式**:
  - [ ] PPTX
  - [ ] PDF
  - [ ] 长图 PNG
- **Acceptance Criteria**:
  - [ ] 生成内容结构合理（标题、要点、图表位置）
  - [ ] 导出 PPTX 可在 PowerPoint 中正常编辑

#### FR-2.5 中文排版优化
- **描述**: 针对中文内容做专门优化
- **功能**:
  - [ ] 中文字体栈（优先系统字体：PingFang SC、Microsoft YaHei）
  - [ ] 中文排版规则（段落缩进、标点挤压、行高适配）
  - [ ] 中文内容密度自适应（中文字符多时分栏策略）

---

### 4.3 Phase 3: 生态功能（6 个月+）

#### FR-3.1 Coding Agent Handoff
- **描述**: 一键将设计交付给 Coding Agent
- **集成**:
  - [ ] 生成 design.md + 组件代码
  - [ ] Claude Code MCP server 插件
  - [ ] OpenCode 集成
- **Acceptance Criteria**:
  - [ ] 导出的代码在 Claude Code 中可被正确理解和修改
  - [ ] 设计系统的 token 在代码中保持一致

#### FR-3.2 团队协作
- **描述**: 多用户协作编辑
- **功能**:
  - [ ] 项目共享（邀请链接）
  - [ ] 实时协作光标（类似 Figma）
  - [ ] 评论和批注
  - [ ] 版本历史

#### FR-3.3 插件系统
- **描述**: 支持第三方扩展
- **接口**:
  - [ ] MCP (Model Context Protocol) 支持
  - [ ] 自定义导出格式插件
  - [ ] 自定义组件库导入

#### FR-3.4 本地模型一键接入
- **描述**: 针对 AutoDL / 本地部署场景优化
- **功能**:
  - [ ] 预设 AutoDL 常见端口配置
  - [ ] vLLM / Xinference 接口模板
  - [ ] 本地模型性能检测（响应时间、token 速度）

---

## 5. 用户旅程 (User Journey)

### 5.1 独立开发者做 MVP 原型

```
1. 打开 OpenDesign（本地或在线）
   ↓
2. 选择模型（DeepSeek V4 @ AutoDL）
   ↓
3. 输入: "帮我做一个冥想 App 的首页，深色模式，要有呼吸动画"
   ↓
4. AI 生成原型 → 右侧画布实时渲染
   ↓
5. 对话迭代:
   "把呼吸圈改成渐变色的"
   "底部加个导航栏，有首页/统计/设置"
   ↓
6. 满意后 → 导出 React 组件
   ↓
7. 复制到项目中继续开发
```

### 5.2 产品经理做 PRD 演示

```
1. 打开 OpenDesign
   ↓
2. 上传竞品 App 截图
   ↓
3. AI 还原为可编辑原型
   ↓
4. 修改文案、调整布局
   ↓
5. 生成多屏原型（首页 → 详情 → 支付）
   ↓
6. 导出 PPTX + 可点击原型链接
   ↓
7. 放入 PRD 给开发团队演示
```

### 5.3 企业私有化部署

```
1. IT 下载 Docker 镜像部署到内网服务器
   ↓
2. 接入公司内部的 vLLM（不连外网）
   ↓
3. 上传品牌 VI 手册 → 自动提取 Design System
   ↓
4. 设计团队使用对话做原型
   ↓
5. 导出 design.md + React 代码 → 交给开发
   ↓
6. 所有数据不出域，满足合规要求
```

---

## 6. 非功能需求

### 6.1 性能要求
| 指标 | 目标 | 说明 |
|------|------|------|
| 首屏加载 | < 3s | 前端 bundle 控制 |
| 代码生成响应 | < 30s | LLM 输出 + 渲染 |
| 沙箱渲染 | < 3s | 代码到 iframe 显示 |
| 并发用户数 | 100+ | 单实例 |

### 6.2 安全要求
- [ ] 用户 API Key 前端加密存储（AES + 用户密码派生密钥）
- [ ] iframe 沙箱隔离（sandbox="allow-scripts" + CSP）
- [ ] LLM 生成代码 XSS 过滤（基础标签白名单）
- [ ] 企业版支持完全离线部署

### 6.3 可维护性
- [ ] 前端组件 100% TypeScript
- [ ] API 接口 Swagger/OpenAPI 文档
- [ ] 单元测试覆盖率 > 60%
- [ ] 可 Docker 一键部署

### 6.4 开源合规
- [ ] 主项目 Apache 2.0 协议
- [ ] 依赖包许可证扫描
- [ ] 贡献者协议 (CLA)

---

## 7. 界面设计草案

### 7.1 主界面布局

```
┌─────────────────────────────────────────────────────────────┐
│  Logo    项目名称 ▼              模型: DeepSeek V4 ▼   设置 │
├─────────────┬───────────────────────┬───────────────────────┤
│             │                       │                       │
│  会话列表   │      画布区域          │      代码/属性        │
│  ─────────  │                       │                       │
│  Session 1  │   ┌─────────────┐     │  ┌─────────────────┐│
│  Session 2  │   │             │     │  │ Design Tokens   ││
│  + New      │   │   原型渲染   │     │  │ ─────────────── ││
│             │   │   iframe    │     │  │ Primary: #3b82f6││
│             │   │             │     │  │ Font: Inter     ││
│             │   │             │     │  │ ...             ││
│             │   └─────────────┘     │  └─────────────────┘│
│             │                       │                       │
│             │  视口切换: 📱 💻      │  ┌─────────────────┐│
│             │                       │  │ 导出:           ││
│             │                       │  │ [HTML] [React]  ││
│             │                       │  │ [PPTX] [PDF]    ││
│             │                       │  └─────────────────┘│
├─────────────┴───────────────────────┴───────────────────────┤
│  用户: 把按钮改成圆角的                                     │
│  AI: 已修改，button 的 rounded-md 改为 rounded-full        │
│  [输入框] [发送] [上传图片] [语音]                         │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 关键交互

- **对话发送**: Enter 发送，Shift+Enter 换行
- **画布交互**: 点击元素 → 右侧属性面板高亮对应代码
- **内联评论**: 右键画布元素 → "添加评论" → AI 根据评论修改
- **版本切换**: 左侧时间轴查看历史版本

---

## 8. 风险与应对

| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 开源模型代码生成质量不稳定 | 中 | 高 | 做 Prompt 工程优化 + fallback 到好模型 |
| LLM API 成本过高 | 中 | 中 | 支持本地模型为主，API 为辅 |
| 用户期望过高（以为能替代设计师） | 高 | 中 | 产品定位明确为"原型工具"而非"设计软件" |
| 竞品（Figma AI）快速跟进 | 中 | 中 | 做差异化：开源 + 本地模型 + 中文优化 |
| 沙箱安全漏洞 | 低 | 高 | CSP + iframe 隔离 + 代码白名单过滤 |
| 社区贡献不足 | 中 | 高 | 初期核心功能自研，文档完善，积极推广 |

---

## 9. 里程碑计划

### Phase 1: MVP — "能跑起来"（2-3 个月）
**目标**: 文本 → UI 原型 → 对话迭代 → 导出代码

- [ ] Week 1-2: 项目脚手架搭建 + 基础 UI 布局
- [ ] Week 3-4: LLM 接口对接 + 基础 Prompt 工程
- [ ] Week 5-6: 沙箱渲染 + 代码预览
- [ ] Week 7-8: 对话迭代编辑 + 版本管理
- [ ] Week 9-10: 代码导出 + 模型切换
- [ ] Week 11-12: 测试 + Bug 修复 + 文档

### Phase 2: 增强 — "能用起来"（3-4 个月）
**目标**: 设计系统 + 多屏 + 截图输入 + PPT 导出

- [ ] Month 4: 设计系统管理模块
- [ ] Month 5: 多屏原型 + 截图输入
- [ ] Month 6: PPT/PDF 导出 + 中文排版优化
- [ ] Month 7: 集成测试 + 性能优化

### Phase 3: 生态 — "能玩起来"（6 个月+）
**目标**: Coding Agent 集成 + 协作 + 插件

- [ ] Month 8-9: Claude Code / OpenCode Handoff
- [ ] Month 10-11: 团队协作功能
- [ ] Month 12+: MCP 插件系统 + 社区运营

---

## 10. 成功指标

### 10.1 技术指标
- [ ] GitHub Stars > 5K (6 个月) / > 15K (12 个月)
- [ ] 支持 5+ LLM Provider
- [ ] 单元测试覆盖率 > 60%

### 10.2 用户指标
- [ ] 周活用户 > 1K (6 个月)
- [ ] 生成的原型数量 > 100K
- [ ] 导出到生产的项目 > 100

### 10.3 社区指标
- [ ] 外部贡献者 > 20
- [ ] 文档完善度 > 80%
- [ ] Discord/微信群成员 > 500

---

## 11. 附录

### 11.1 参考项目
- [OpenUI](https://github.com/wandb/openui) — wandb 开源 UI 生成器
- [screenshot-to-code](https://github.com/abi/screenshot-to-code) — 截图转代码
- [v0.dev](https://v0.dev) — Vercel AI UI 生成器（闭源）
- [Claude Design](https://www.anthropic.com/news/claude-design-anthropic-labs) — Claude 设计工具（闭源）
- [Google Stitch](https://stitch.withgoogle.com/) — Google AI UI 工具（实验性）

### 11.2 关键 Prompt 模板（初稿）

**System Prompt — UI 生成器**:
```
你是一个专业的前端开发工程师和 UI 设计师。你的任务是根据用户需求生成高质量、可执行的 HTML 代码。

规则：
1. 使用 Tailwind CSS 进行样式设计
2. 使用现代、简洁的设计风格
3. 确保响应式布局
4. 代码必须是完整的、可独立运行的 HTML 文件
5. 包含所有必要的 CDN 链接
6. 使用语义化 HTML 标签
7. 添加适当的过渡动画和交互效果
8. 优先使用 Tailwind 的内置类，避免 arbitrary values

Design System（当前项目）:
- Primary Color: {primary}
- Font: {font}
- Base Spacing: {spacing}
- Border Radius: {radius}
```

**迭代编辑 Prompt**:
```
基于之前的设计，用户要求修改：{user_request}

当前代码：
{current_code}

请只修改必要的部分，保留其他设计决策。输出修改后的完整代码。
```

### 11.3 Design Token JSON Schema

```json
{
  "name": "My Project Design System",
  "version": "1.0.0",
  "colors": {
    "primary": {"value": "#3b82f6", "usage": "主品牌色"},
    "secondary": {"value": "#8b5cf6", "usage": "次要强调色"},
    "background": {"value": "#ffffff", "dark": "#0f172a"},
    "surface": {"value": "#f8fafc", "dark": "#1e293b"},
    "text": {"value": "#1e293b", "dark": "#f1f5f9"},
    "muted": {"value": "#64748b"}
  },
  "typography": {
    "fontFamily": "Inter, system-ui, -apple-system, sans-serif",
    "fontFamilyChinese": "Inter, PingFang SC, Microsoft YaHei, sans-serif",
    "scale": [12, 14, 16, 18, 20, 24, 30, 36, 48, 60],
    "lineHeight": {"tight": 1.25, "normal": 1.5, "relaxed": 1.75}
  },
  "spacing": {
    "base": 4,
    "scale": [0, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96]
  },
  "borderRadius": {
    "sm": "4px",
    "md": "8px",
    "lg": "12px",
    "xl": "16px",
    "full": "9999px"
  }
}
```

---

*此文档基于 2026-04-24 的市场调研和技术分析编写，后续随项目演进更新。*
