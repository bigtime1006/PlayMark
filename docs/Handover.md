## 工作交接计划（工程/运维/测试）

### 一、仓库结构
- iOS 主端：`PlayMark/PlayMark/`
- watchOS：`PlayMark/PlayMarkWatch Watch App/`
- 文档：`docs/`（本文件、设计文档、需求变更记录）

### 二、构建与运行
- iOS：方案 `PlayMark`，iPhone 模拟器/真机
- watchOS：方案 `PlayMarkWatch Watch App`，Watch 模拟器/真机
- 常见问题：
  - 若遇到“Watch-only app cannot be contained...”安装失败：卸载残留 → 抹除模拟器 → 清空 DerivedData → 清理构建 → 重装
  - `WCSession is not paired`：请在 Xcode/Simulator 配对 iPhone 与 Watch 模拟器

### 三、功能模块与负责人交接
- 会话与存储：`Models/Storage/SessionStore`（iOS）、`WatchSessionStore/WatchStorage`（watchOS）
- 同步：`Connectivity.swift`（iOS）、`WatchSessionStore.swift` 内 send/receive（watchOS）
- 标签配置：`TagProfiles.swift`、`TagEditorView.swift`、`ProfileListView.swift`
- 视频流水线：`VideoPipeline.swift`、`VideoExporter.swift`、`VideoImportView.swift`
- 历史与回收站：`HistoryView.swift`、`RecycleBinView.swift`

### 四、上线与证书
- 当前使用本地签名 `Sign to Run Locally`
- 上线需接入团队证书与包标识，配置 Capabilities（如 HealthKit 未来加入）

### 五、测试用例建议
- 单元：
  - 片段计算（边界：开头/结尾、重叠合并、最小时长过滤）
  - 导出 SRT 时间格式与排序
  - 会话/高光的软删/硬删/恢复
- UI：
  - iPhone 历史列表选择/批量删除/回收站
  - 视频导入→对齐→预览→导出（含烧录字幕开关）
  - Watch 最近活动编辑与撤销

### 六、后续工作队列
- 视频导出：多文件导出、转场/过渡、异步导出进度
- 历史详情：批量恢复/撤销、筛选预设
- Watch：批量选择删除、搜索/排序
- 兼容：导出器使用新的 async API（iOS 18+）

### 七、运维与数据
- 存储为本地 JSON；如需 iCloud 同步需设计加密与冲突合并策略
- 日志：目前以调试日志为主，无远程日志/崩溃收集；可接入统一平台



