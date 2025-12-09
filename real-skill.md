---
name: gratitude-100days-real
description: Real-world constraints for Gratitude to Parents 100 Days project
---

# 现实约束 (Real)

<real>
- 产品必须在7天内完成MVP，功能设计必须极简，仅包含每日记录、进度展示等基础模块
- 技术方案限定为前端+localStorage本地存储，不能依赖复杂后端或数据库，不允许引入太多库
- 目标用户为25-45岁普通中国用户，无技术背景，必须做到零学习成本：无需注册登录、页面打开即可使用、小白级操作
- 开发者为单人，无设计师、无后端、无产品团队支持，预算为零，不调用付费API，界面和交互必须保持极简
</real>

<constraint type="time">
- 硬性期限：7天内完成可用MVP
- 功能范围：仅包含核心记录功能和进度展示
- 迭代策略：先上线最小可用版本，后续根据反馈迭代
</constraint>

<constraint type="tech">
- 前端技术栈：HTML + CSS + JavaScript（原生或轻量框架）
- 数据存储：浏览器localStorage
- 依赖限制：不引入复杂库，避免构建工程化
- 兼容性：支持主流浏览器（Chrome、Safari、Edge）
</constraint>

<constraint type="user">
- 年龄段：25-45岁成年人
- 技术水平：无技术背景，普通互联网用户
- 使用场景：每天1分钟快速记录
- 操作要求：打开即用，无需注册、登录、学习
- 心理预期：轻量、无负担、不打扰
</constraint>

<constraint type="resource">
- 团队规模：单人开发
- 预算：零预算
- 外部资源：无设计师、无后端工程师、无产品经理
- API限制：不调用任何付费API
- 设计要求：极简风格，依赖浏览器原生样式或简单CSS
</constraint>
