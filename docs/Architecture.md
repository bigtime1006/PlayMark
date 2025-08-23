## 架构概览（Architecture）

### 目标
- 支撑“运动中低时延打点 + 赛后高效复盘/导出”的端侧架构；保证离线可用、同步简单可靠、导出链路清晰。

### 工程与目标
- iOS App（主端）：`PlayMark/PlayMark`（SwiftUI）
- watchOS App（记录端）：`PlayMark/PlayMarkWatch Watch App`（SwiftUI）
- 测试：`PlayMarkTests`、`PlayMarkWatch Watch AppTests`

### 模块划分（iOS）
- Models：`Highlight`、`RecordSession`（含 title/updatedAt/deletedAt 等）
- Storage：本地 JSON 文件读写（`Storage.swift`）
- SessionStore：会话状态、增删改查、软/硬删除、回收站（`SessionStore.swift`）
- Connectivity：WatchConnectivity 收发与桥接（`Connectivity.swift`）
- TagProfiles：标签/颜色 Profile 管理与“立即同步到手表”（`TagProfiles.swift`）
- Views：主页、历史/回收站、详情/编辑、标签编辑、视频导入对齐（SwiftUI）
- Video：片段计算（`VideoPipeline.swift`）、导出器（`VideoExporter.swift`）、导入对齐 UI（`VideoImportView.swift`）

### 模块划分（watchOS）
- WatchSessionStore：手表会话状态、发送到 iPhone、最近活动编辑与撤销（`WatchSessionStore.swift`）
- WatchStorage：最近活动 JSON 持久化（`WatchStorage.swift`）
- Views：录制/打点入口（四标签网格）、最近活动列表/详情（`ContentView.swift`、`RecentViews.swift`）

### 数据流（核心时序）
1) 手表打点
   - 用户点击快捷标签 → `WatchSessionStore.addHighlight()` 入列
   - 结束 → 生成 `RecordSession`，经 `WatchConnectivity.send(session:)` 发送到 iPhone

2) iPhone 接收
   - `Connectivity.didReceiveMessageData` 解码 `RecordSession`
   - `SessionStorage` 追加到 `sessions.json`，通知 `SessionStore` 刷新 UI

3) 标签配置下发
   - iPhone `TagProfileStore.forceSyncToWatch()` → 通过 `tagsV2`（含 Profile 名称与前四个标签+颜色）下发到手表
   - 手表收到后落盘到 `UserDefaults`，离线可用

4) 视频导出
   - 选择视频与会话 → 设定对齐（会话开始/第一个标记 → 某视频时间）
   - `VideoPipeline.computeSegments()` 计算并合并片段区间
   - `VideoExporter.exportMergedMP4()` 合并导出（可选烧录字幕）；`generateSRT()` 生成字幕文件

### 存储与持久化
- iOS：`Application Support/PlayMark/sessions.json`（全量会话）；软删除保留 `deletedAt`
- watchOS：`Application Support/PlayMarkWatch/recent_sessions.json`（最近 20 条）
- 用户配置：手表端 `UserDefaults` 持久化标签与颜色、Profile 名称

### 错误处理与健壮性
- WatchConnectivity：若不可达则使用 `updateApplicationContext` 兜底；手表端保留最近标签配置以离线可用
- 导出：无视频轨/导出失败返回明确错误；文件存在先删除再写
- 软删除/回收站：避免误删；支持恢复与彻底删除

### 性能与时延
- 打点：手表端本地追加并触觉反馈；不等待 iPhone 回执
- 导出：合并区间减少剪切次数；字幕烧录使用 CoreAnimation 图层，默认 30fps

### 扩展点
- 多文件导出、过渡/转场、异步导出进度
- iCloud 同步（需加密与冲突合并）
- HealthKit（心率等）与时间点关联

### 关键文件一览
- iOS：`Models.swift`、`SessionStore.swift`、`Storage.swift`、`Connectivity.swift`、`VideoPipeline.swift`、`VideoExporter.swift`、`VideoImportView.swift`
- watchOS：`WatchSessionStore.swift`、`WatchStorage.swift`、`RecentViews.swift`



