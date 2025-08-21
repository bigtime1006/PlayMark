## 项目概览

- **名称**: 秒记 / Highlight Marker
- **平台**: iOS（主管理端）+ watchOS（记录端）
- **核心价值**: 运动中一键标记“精彩瞬间”，赛后导出时间点清单用于剪辑/分析

## 功能已实现

- **会话管理（iPhone）**: 开始/标记/结束；进行中支持快速标签；结束后保存历史
- **导出（iPhone）**: TXT/CSV/JSON，均包含相对秒数与实际时间（ISO8601）
- **UI（iPhone）**: 全中文、样式化配色/按钮、列表显示“相对秒数 + 实际时间”
- **watchOS**: 开始/四个快捷标签/结束；触觉反馈；结束后自动同步到 iPhone
- **同步**: Watch 通过 `WCSession` 发送完整会话；iPhone 后台接收、落盘并刷新列表

## 目录与关键文件

- **iOS App（主端）**
  - `PlayMark/PlayMark/Models.swift`（`Highlight`、`RecordSession`）
  - `PlayMark/PlayMark/Storage.swift`（本地 JSON 持久化）
  - `PlayMark/PlayMark/SessionStore.swift`（会话状态管理）
  - `PlayMark/PlayMark/Exporters.swift`（TXT/CSV/JSON 导出）
  - `PlayMark/PlayMark/Connectivity.swift`（iPhone 端 `WCSession` 接收 Watch 会话）
  - `PlayMark/PlayMark/ContentView.swift`（中文 UI，开始/结束/标记/导出）
  - `PlayMark/PlayMark/Colors.swift`（配色）
- **watchOS App（记录端）**
  - `PlayMark/PlayMarkWatch Watch App/SharedModels.swift`（Watch 端共享模型）
  - `PlayMark/PlayMarkWatch Watch App/WatchSessionStore.swift`（手表端会话管理/发送）
  - `PlayMark/PlayMarkWatch Watch App/ContentView.swift`（四标签+结束）

## 数据模型

- **Highlight**: `id`, `timestamp`(相对秒), `createdAt`(实际时间), `tag`
- **RecordSession**: `id`, `startedAt`, `endedAt?`, `highlights[]`, `note?`

## 本地存储

- **文件位置**: `Application Support/PlayMark/sessions.json`
- **逻辑**: iPhone 端结束会话或接收到 Watch 会话后即时写入；首页自动刷新

## 导出

- **TXT**: 含头部会话起止时间；行格式：`seconds    actual_time    tag`
- **CSV**: 表头 `timestamp_seconds,actual_time_iso,tag`（ISO8601，逗号分隔，含引号转义）
- **JSON**: 完整 `RecordSession` 对象数组
- **分享**: 系统分享面板，文件名根据格式自动生成

## 使用说明

- **iPhone**
  - 开始记录: 点“开始”
  - 标记瞬间: 点“标记瞬间”，可选择快速标签（或仅标记）
  - 结束记录: 进行中按钮变为“结束”，点击即停止并保存至历史
  - 查看与导出: 历史列表进入详情 → 选择导出 → TXT/CSV/JSON
- **Watch**
  - 开始记录: “开始”
  - 标记瞬间: 进行中显示四个按钮（默认：进球/助攻/失误/冲刺），点击即打点并触觉反馈
  - 结束记录: “结束”（长震反馈）；会话自动发送到 iPhone 并保存

## iPhone ↔ Watch 同步

- **结束后同步**: Watch 将 `RecordSession` 通过 `WCSession` 发送；iPhone 端 `Connectivity.swift` 接收与落盘，并发通知刷新首页
- **运行建议**: 首次可先启动 iPhone App，再在 Watch 上开始/结束，确保会话能到达并展示

## 构建与运行

- **iPhone（模拟器/真机）**: 方案选择 “PlayMark”，任意 iPhone 运行
- **Watch（模拟器/真机）**: 方案选择 “PlayMarkWatch Watch App”，选择任意 Apple Watch 模拟器运行
- **如遇安装问题**: 卸载残留、抹除模拟器、清空 DerivedData、重新构建安装（曾出现的 watch-only 报错已通过此流程修复）

## 权限与隐私

- 当前未接入 HealthKit；后续若记录心率等需在 iOS 与 watchOS 目标分别开启 HealthKit，并请求授权
- 数据仅本地 JSON 存储；如需云同步，需增加加密与隐私策略

## 后续路线图（建议）

- 表盘 Complication：一键开始/标记
- 标签管理：自定义常用标签、排序与颜色
- Siri Shortcuts：语音“开始/标记/结束”
- HealthKit：心率/卡路里采集并与瞬间关联
- 视频对齐助手：开场同步动作与参考声/震动
- 离线/重试：Watch 离线缓存，iPhone 恢复联机后重传
- 导出增强：FCPX XML/EDL；批量导出多会话

## 待确认

- 快速标签默认四项与顺序（手表端按钮）
- 导出默认格式（当前预设 CSV）
- 列表时间格式是否统一显示 ISO8601（iOS 当前已改为 ISO8601）
- 是否需要在 iPhone 端也提供“正在进行”状态下的悬浮快速标签

## 变更一览（本次迭代）

- 新增与更新 `Models/Storage/SessionStore/Exporters/Connectivity/Colors/ContentView`（iOS）
- 新增 `PlayMarkWatch Watch App` 下 `SharedModels/WatchSessionStore/ContentView`（watchOS）
- iPhone/Watch 全中文界面文案；列表显示“相对秒 + 实际时间(ISO8601)”；导出含两种时间；Watch 端四按钮快速标记与触觉反馈




## 产品决策与现状（截至 2025-08-21）

- 不做心率/HealthKit（暂不纳入 MVP）
- 已实现标签配置（含颜色）与多运动类型（默认：足球运动；新增：篮球运动）
- iPhone ↔ Watch 标签同步：含 Profile 名称与前 4 个标签及颜色；手表离线时使用上次落盘配置
- iPhone 顶部加入“立即同步到手表”按钮；配置页也支持手动同步
- Watch 端顶部显示当前 Profile 名称，四个按钮按颜色显示；点击即低时延打点


## 视频导入与对齐剪辑流水线（规划）

### 目标
- 从相册/文件导入运动视频，按会话打点生成剪辑片段，用于快速复盘/发布
- 支持“对齐锚点”，将视频时间轴与会话时间线对齐（开场拍手/口令/哨声等）
- 一次性导出片段 MP4（逐段或合并），可选生成字幕文件（SRT/VTT）

### MVP 范围（v1）
- 导入单个视频（本地相册/文件）
- 选择一个会话作为“时间线”
- 指定对齐锚点：
  - 方式 A：用户输入“第一个标记对应的视频时间”
  - 方式 B：在视频预览中点击某一帧作为起点（与会话 start 或第一个标记对齐）
- 片段规则：每个标记导出 [preRoll, postRoll]（默认 3s 前 ~ 5s 后，可调），重叠自动合并
- 片段上叠加可选字幕：标签名称（例如“进球/犯规/搞笑/配合”）
- 导出：
  - 合并为一个 MP4（片段顺序拼接，片间可选 0.1s 过渡黑场）
  - 或导出独立多个 MP4（每个片段一个文件）
  - 可选生成 SRT/VTT（片段开始/结束 + 文本 = 标签）

### 技术路线
- 媒体：AVFoundation（`AVAsset`, `AVMutableComposition`, `AVAssetExportSession`）
- 对齐：将 `RecordSession.startedAt/Highlight.createdAt` 转为绝对时间轴；结合用户指定的视频锚点（video time ↔ session time）计算映射 `sessionTime -> videoTime`
- 片段：根据每个 highlight 的相对时间（或绝对时间）与映射计算出视频内 `CMTimeRange`
- 合并重叠：按时间排序，相交区间合并以减少切割次数
- 字幕：SRT/VTT 文本生成（无需烧录到视频流，降低导出耗时；后续支持 CoreAnimation 叠加）
- 导出：H.264/AAC，MP4 容器；码率/分辨率初期使用“与源一致”与“中等码率”预设

### 界面与流程（iOS）
- “视频”页新增入口
  1) 选择会话（默认最近）
  2) 选择视频（PhotoPicker/DocumentPicker）
  3) 对齐设置：
     - 方式 A：输入“第一个标记对应的视频时间（mm:ss.s）”
     - 方式 B：预览中定位一帧 → 设为会话开始/第一个标记的对齐点
  4) 片段参数：preRoll/postRoll、最小片段长度、是否合并重叠、导出方式（合并/多文件）、是否生成 SRT/VTT、是否叠加字幕
  5) 导出 → 渲染进度页（可取消/后台）→ 系统分享

### 里程碑
- M1 导入+锚点+生成片段列表（不导出，仅预览范围）
- M2 片段合并/导出单个 MP4；可选导出 SRT/VTT
- M3 多文件导出；字幕叠加（烧录）
- M4 批量处理多个会话；导出 FCPXML（与 NLE 对接）

### 当前进度（本次提交）
- 需求确认：已确认先做视频流水线，心率不纳入
- 代码准备：
  - 会话与打点数据已具备（含绝对时间 `createdAt`）
  - iPhone UI/数据流可复用；后续新增 Video 模块（`VideoPipeline/VideoEditor.swift`、`VideoViews/VideoImportView.swift` 等）
- 待实现（下一步）：
  1) 创建 `VideoPipeline` 模块与占位实现（时间映射、range 计算、SRT 生成）
  2) 新增“视频”页与导入/对齐 UI，接入会话选择器
  3) 生成片段预览并导出合并 MP4 + SRT（MVP）

### 风险与性能
- 长视频内多片段截取的 IO 压力与内存占用
- 不同帧率/变帧视频的对齐精度（将使用 `CMTime` 严格运算）
- 部分源视频编码/封装不标准导致导出失败（提供降级：转码后再拼接）

# PlayMark
