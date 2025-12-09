---
name: gratitude-100days-spec
description: Specification document for Gratitude to Parents 100 Days project
---

# 规约书 (Spec)

<spec>

<section name="产品概述">

产品名称：感恩父母100天（网页版记录工具）

产品定位：轻量级、纯前端的感恩记录工具，帮助用户连续100天记录对父母的感恩事件

核心价值：
- 建立持续表达感恩的习惯
- 促进更稳定、更柔软的亲子关系
- 形成属于用户自己的"家庭情感记录库"
- 通过极简界面降低使用门槛，每天1分钟完成记录

技术方案：纯前端 + localStorage本地存储

开发周期：7天MVP

</section>

<section name="数据结构">

<data-structure name="Entry">
记录实体（Entry）：
{
  id: string,              // 唯一标识，格式：entry_YYYYMMDD_timestamp
  date: string,            // 记录日期，格式：YYYY-MM-DD
  content: string,         // 记录内容，最大长度500字符
  emotion: string | null,  // 情绪标记，可选值：开心、感动、平静、反思、null
  timestamp: number,       // 创建时间戳（毫秒）
  createdAt: string        // 创建时间，格式：YYYY-MM-DD HH:mm:ss
}

字段约束：
- id：必填，唯一，自动生成
- date：必填，格式YYYY-MM-DD，每天只能有一条记录
- content：必填，长度1-500字符，不允许纯空格
- emotion：可选，枚举值之一或null
- timestamp：必填，自动生成
- createdAt：必填，自动生成
</data-structure>

<data-structure name="Progress">
进度实体（Progress）：
{
  totalDays: number,       // 总记录天数（0-100）
  streakDays: number,      // 连续记录天数
  percentage: number,      // 完成百分比（0-100）
  lastRecordDate: string,  // 最后一次记录日期，格式：YYYY-MM-DD
  startDate: string        // 首次记录日期，格式：YYYY-MM-DD
}

计算逻辑：
- totalDays = localStorage中Entry的总数量
- streakDays = 从最近一天往前推的连续记录天数
- percentage = (totalDays / 100) * 100
- lastRecordDate = 最新Entry的date字段
- startDate = 最早Entry的date字段
</data-structure>

<data-structure name="AppState">
应用状态（AppState）：
{
  entries: Entry[],        // 所有记录列表
  progress: Progress,      // 进度信息
  currentDate: string,     // 当前日期，格式：YYYY-MM-DD
  todayEntry: Entry | null // 今日记录（如果存在）
}

存储方式：
- localStorage key: "gratitude100_entries"
- 数据格式：JSON字符串
- 读取时机：页面加载时
- 写入时机：每次保存记录后
</data-structure>

</section>

<section name="功能规约">

<feature name="daily-record">
功能：每日记录

输入：
- 记录内容（content）：文本输入框，1-500字符
- 情绪标记（emotion）：可选，单选按钮组（开心、感动、平静、反思）

处理逻辑：
1. 检查今天是否已有记录
   - 如果已有记录，显示"今日已记录"提示，禁用输入框
   - 如果未记录，显示输入界面
2. 验证输入内容
   - 内容不能为空或纯空格
   - 内容长度不超过500字符
3. 生成Entry对象
   - id = "entry_" + YYYYMMDD + "_" + timestamp
   - date = 当前日期（YYYY-MM-DD）
   - content = 用户输入内容（去除首尾空格）
   - emotion = 用户选择的情绪或null
   - timestamp = Date.now()
   - createdAt = 当前时间（YYYY-MM-DD HH:mm:ss）
4. 保存到localStorage
   - 读取现有entries数组
   - 添加新Entry到数组
   - 按date倒序排序
   - 写入localStorage
5. 更新进度显示
6. 显示成功提示

输出：
- 成功提示："记录成功！已坚持X天"
- 更新后的进度条
- 禁用今日输入界面

边界条件：
- 每天只能记录一次，不允许修改或删除
- 如果localStorage已满，提示用户导出备份后清理
- 如果日期跨天，自动刷新页面状态
</feature>

<feature name="progress-display">
功能：进度展示

显示内容：
1. 进度条
   - 视觉：横向进度条，填充百分比
   - 颜色：0-30%（浅蓝）、31-70%（蓝色）、71-100%（深蓝）
   - 文字：X/100天
2. 连续天数
   - 文字："已连续坚持X天"
   - 如果中断，显示："上次中断于YYYY-MM-DD"
3. 鼓励文案
   - 1-7天："刚刚开始，继续加油！"
   - 8-30天："习惯正在养成，坚持下去！"
   - 31-70天："你已经坚持了这么久，真棒！"
   - 71-99天："即将完成100天，最后冲刺！"
   - 100天："恭喜你完成100天感恩记录！"

计算逻辑：
1. 读取localStorage中的所有Entry
2. 计算totalDays = entries.length
3. 计算streakDays：
   - 从今天开始往前推
   - 如果前一天有记录，streak+1
   - 如果前一天无记录，停止计算
4. 计算percentage = (totalDays / 100) * 100

更新时机：
- 页面加载时
- 保存新记录后
- 用户手动刷新时
</feature>

<feature name="history-view">
功能：历史记录查看

显示内容：
1. 记录列表
   - 按日期倒序排列（最新的在最上面）
   - 每条记录显示：日期、内容摘要（前50字）、情绪标记
2. 记录详情
   - 点击某条记录，展开显示完整内容
   - 显示：日期、完整内容、情绪标记、创建时间

交互逻辑：
1. 默认显示最近10条记录
2. 点击"加载更多"显示更多记录
3. 点击某条记录，展开/收起详情
4. 支持按情绪筛选（全部、开心、感动、平静、反思）

边界条件：
- 如果没有记录，显示"暂无记录，开始你的第一天吧！"
- 如果记录超过100条，只显示最近100条
</feature>

<feature name="data-export">
功能：数据导出

导出格式：
1. 文本格式（.txt）
   - 格式：
     ```
     感恩父母100天记录
     导出时间：YYYY-MM-DD HH:mm:ss
     总记录天数：X天

     ========================================

     日期：YYYY-MM-DD
     情绪：开心
     内容：今天和妈妈通了电话...

     ========================================

     日期：YYYY-MM-DD
     情绪：感动
     内容：...
     ```

2. JSON格式（.json）
   - 格式：
     ```json
     {
       "exportTime": "YYYY-MM-DD HH:mm:ss",
       "totalDays": 30,
       "entries": [
         {
           "id": "entry_20250101_1234567890",
           "date": "2025-01-01",
           "content": "...",
           "emotion": "开心",
           "timestamp": 1234567890,
           "createdAt": "2025-01-01 12:00:00"
         }
       ]
     }
     ```

处理逻辑：
1. 读取localStorage中的所有Entry
2. 根据用户选择的格式生成文件内容
3. 创建Blob对象
4. 触发浏览器下载
5. 文件名：gratitude100_YYYYMMDD.txt 或 gratitude100_YYYYMMDD.json

边界条件：
- 如果没有记录，提示"暂无记录可导出"
- 导出成功后显示提示："导出成功！"
</feature>

<feature name="data-import">
功能：数据导入（可选，MVP可不实现）

支持格式：JSON格式

处理逻辑：
1. 用户选择JSON文件
2. 读取文件内容
3. 验证JSON格式
4. 验证Entry数据结构
5. 合并到现有数据（去重，按date去重）
6. 写入localStorage
7. 刷新页面显示

边界条件：
- 如果JSON格式错误，提示"文件格式错误"
- 如果数据结构不匹配，提示"数据格式不正确"
- 如果导入后总记录超过100条，提示"已达到100天上限"
</feature>

</section>

<section name="界面规约">

<ui-layout>
页面布局：单页应用（SPA）

主要区域：
1. 顶部区域（Header）
   - 产品标题："感恩父母100天"
   - 副标题："记录每一天的感恩时刻"

2. 进度区域（Progress）
   - 进度条
   - 统计信息（总天数、连续天数）
   - 鼓励文案

3. 记录区域（Record）
   - 今日记录输入框（如果未记录）
   - 今日记录显示（如果已记录）
   - 情绪选择按钮组
   - 保存按钮

4. 历史区域（History）
   - 历史记录列表
   - 筛选按钮
   - 加载更多按钮

5. 底部区域（Footer）
   - 导出按钮
   - 关于说明
</ui-layout>

<ui-style>
设计风格：极简、温暖、柔和

颜色方案：
- 主色：#4A90E2（柔和蓝色）
- 辅助色：#F5F5F5（浅灰背景）
- 文字色：#333333（深灰）
- 强调色：#FF6B6B（温暖红色，用于情绪标记）

字体：
- 标题：18-24px，加粗
- 正文：14-16px，常规
- 辅助文字：12px，浅色

间距：
- 区域间距：24px
- 元素间距：16px
- 内边距：12px

圆角：
- 按钮：8px
- 卡片：12px
- 输入框：6px
</ui-style>

<ui-interaction>
交互反馈：
1. 按钮点击：颜色变深 + 轻微缩放
2. 输入框聚焦：边框高亮
3. 保存成功：顶部显示成功提示（3秒后自动消失）
4. 加载状态：显示加载动画
5. 错误提示：红色文字 + 抖动动画

响应式设计：
- 桌面端（>768px）：双栏布局，左侧记录，右侧历史
- 移动端（≤768px）：单栏布局，上下排列
</ui-interaction>

</section>

<section name="业务逻辑">

<logic name="streak-calculation">
连续天数计算逻辑：

输入：所有Entry的date数组

处理步骤：
1. 获取今天的日期（YYYY-MM-DD）
2. 检查今天是否有记录
   - 如果没有，从昨天开始计算
   - 如果有，从今天开始计算
3. 初始化streak = 0
4. 从起始日期开始往前推：
   - 检查前一天是否有记录
   - 如果有，streak++，继续往前推
   - 如果没有，停止计算
5. 返回streak

边界条件：
- 如果只有今天一条记录，streak = 1
- 如果今天没记录，昨天有记录，streak = 昨天的连续天数
- 如果连续多天没记录，streak = 0
</logic>

<logic name="date-validation">
日期验证逻辑：

输入：date字符串（YYYY-MM-DD）

验证规则：
1. 格式验证：必须是YYYY-MM-DD格式
2. 日期有效性：必须是有效日期（如2月30日无效）
3. 时间范围：不能是未来日期
4. 唯一性：该日期不能已有记录

返回：
- 验证通过：true
- 验证失败：false + 错误信息
</logic>

<logic name="content-validation">
内容验证逻辑：

输入：content字符串

验证规则：
1. 非空验证：去除首尾空格后不能为空
2. 长度验证：1-500字符
3. 特殊字符：允许所有Unicode字符（包括中文、标点、emoji）

返回：
- 验证通过：true
- 验证失败：false + 错误信息
</logic>

<logic name="storage-management">
存储管理逻辑：

localStorage容量检查：
1. 在保存前检查localStorage剩余空间
2. 如果空间不足，提示用户导出备份
3. 提供清理选项（删除最早的记录）

数据备份策略：
1. 每次保存后，自动备份到localStorage的备份key
2. 备份key："gratitude100_backup"
3. 如果主数据损坏，尝试从备份恢复

数据迁移：
1. 检查localStorage中的数据版本
2. 如果版本不匹配，执行数据迁移
3. 迁移完成后更新版本号
</logic>

</section>

<section name="边界条件">

<boundary name="time-boundary">
时间边界：
1. 100天上限：
   - 当totalDays达到100时，显示完成页面
   - 不再允许新增记录
   - 提供导出和分享功能
2. 日期跨天：
   - 每次页面加载时检查当前日期
   - 如果日期变化，刷新todayEntry状态
3. 时区处理：
   - 使用本地时区
   - 日期以本地时间为准
</boundary>

<boundary name="data-boundary">
数据边界：
1. 记录数量：最多100条
2. 单条内容长度：最多500字符
3. localStorage容量：约5MB（浏览器限制）
4. 情绪选项：固定4种（开心、感动、平静、反思）
</boundary>

<boundary name="user-boundary">
用户边界：
1. 单浏览器单用户：
   - 数据存储在浏览器localStorage
   - 不同浏览器数据不互通
   - 清除浏览器数据会丢失记录
2. 无账号系统：
   - 无需注册登录
   - 无法跨设备同步
   - 数据完全本地化
</boundary>

<boundary name="error-boundary">
错误边界：
1. localStorage不可用：
   - 检测localStorage是否可用
   - 如果不可用，显示错误提示
   - 建议用户检查浏览器设置
2. 数据损坏：
   - 尝试从备份恢复
   - 如果备份也损坏，提示用户重新开始
3. 浏览器兼容性：
   - 检测浏览器版本
   - 如果不支持localStorage，显示升级提示
</boundary>

</section>

<section name="验收标准">

<acceptance name="mvp-checklist">
MVP验收清单：

核心功能：
- [ ] 用户可以每天记录一条感恩内容
- [ ] 用户可以选择情绪标记（可选）
- [ ] 系统显示当前进度（总天数、连续天数、百分比）
- [ ] 用户可以查看历史记录列表
- [ ] 用户可以导出记录（文本格式）

数据持久化：
- [ ] 记录保存到localStorage
- [ ] 页面刷新后数据不丢失
- [ ] 关闭浏览器后数据保留

界面体验：
- [ ] 界面简洁清晰，无多余元素
- [ ] 操作流程顺畅，无需学习
- [ ] 移动端和桌面端都能正常使用
- [ ] 按钮和输入框有明确的视觉反馈

边界处理：
- [ ] 每天只能记录一次
- [ ] 内容长度限制在500字符
- [ ] 达到100天后显示完成页面
- [ ] localStorage不可用时显示错误提示

性能要求：
- [ ] 页面加载时间 < 2秒
- [ ] 保存记录响应时间 < 500ms
- [ ] 历史记录列表渲染流畅（无卡顿）
</acceptance>

<acceptance name="quality-checklist">
质量验收清单：

代码质量：
- [ ] 代码结构清晰，易于维护
- [ ] 关键函数有注释说明
- [ ] 无明显的代码冗余

浏览器兼容性：
- [ ] Chrome（最新版）
- [ ] Safari（最新版）
- [ ] Edge（最新版）
- [ ] Firefox（最新版）

响应式设计：
- [ ] 桌面端（1920x1080）
- [ ] 平板端（768x1024）
- [ ] 移动端（375x667）

用户体验：
- [ ] 首次使用有简单引导
- [ ] 操作反馈及时明确
- [ ] 错误提示友好易懂
- [ ] 完成100天有仪式感
</acceptance>

</section>

</spec>
