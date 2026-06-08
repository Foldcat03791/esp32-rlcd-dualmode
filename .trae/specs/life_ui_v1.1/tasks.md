# LIFE 模式 UI 优化 v1.1 - 实现计划

## [x] Task 1: 添加 Material Icons 字体配置
- **Priority**: P0
- **Depends On**: None
- **Description**: 
  - 在 font 配置中添加 Material Icons 字体
  - 配置三种尺寸：small（14px）、medium（18px）、large（24px）
- **Acceptance Criteria Addressed**: AC-1, AC-4, AC-5, AC-6
- **Test Requirements**:
  - `programmatic` TR-1.1: 编译成功，字体配置无错误
  - `human-judgment` TR-1.2: 图标显示清晰可辨
- **Notes**: 使用 Google Fonts 提供的 Material Icons

## [x] Task 2: 添加 WiFi 信号传感器
- **Priority**: P0
- **Depends On**: None
- **Description**: 
  - 添加 `wifi_signal` 传感器获取 RSSI 值
  - 用于判断 WiFi 信号强度并显示对应图标
- **Acceptance Criteria Addressed**: AC-1, AC-4
- **Test Requirements**:
  - `programmatic` TR-2.1: WiFi 连接时能获取 RSSI 值
  - `human-judgment` TR-2.2: WiFi 图标随信号强度变化
- **Notes**: RSSI 范围通常为 -100dB 到 -30dB，数值越大信号越强

## [x] Task 3: 添加蓝牙状态显示（简单模式）
- **Priority**: P0
- **Depends On**: None
- **Description**: 
  - 配置 ESP32 BLE 组件（启用但不扫描）
  - 显示 "BT" 文字和状态图标
- **Acceptance Criteria Addressed**: AC-1, AC-5
- **Test Requirements**:
  - `programmatic` TR-3.1: BLE 组件配置成功
  - `human-judgment` TR-3.2: 蓝牙状态正确显示（开启/关闭）
- **Notes**: 简单模式仅显示开启状态，不进行设备扫描

## [x] Task 4: 添加 OpenWeather API 集成
- **Priority**: P0
- **Depends On**: None
- **Description**: 
  - 添加 HTTP 请求组件
  - 创建模板传感器存储天气数据（温度、描述、图标代码）
  - 配置每小时刷新一次
- **Acceptance Criteria Addressed**: AC-3, AC-7
- **Test Requirements**:
  - `programmatic` TR-4.1: HTTP 请求能成功获取天气数据
  - `programmatic` TR-4.2: 日志显示每小时刷新一次
  - `human-judgment` TR-4.3: 天气信息正确显示在界面上
- **Notes**: 需要用户提供 API Key 和城市 ID

## [x] Task 5: 实现分屏布局 UI
- **Priority**: P0
- **Depends On**: Task 1, Task 2, Task 3, Task 4
- **Description**: 
  - 修改 display lambda 实现新布局
  - 顶部状态栏：LIFE 标识 + WiFi + BT + 电池图标
  - 左侧：大号时间 + 日期
  - 右侧：天气图标 + 温度 + 描述
  - 底部：待办列表 + 操作提示
- **Acceptance Criteria Addressed**: AC-1, AC-2, AC-3, AC-4, AC-5, AC-6
- **Test Requirements**:
  - `human-judgment` TR-5.1: 布局符合设计要求
  - `human-judgment` TR-5.2: 所有元素显示完整，无重叠
  - `human-judgment` TR-5.3: 图标清晰，文字可读
- **Notes**: 400×300 屏幕，左右分屏各约 200px 宽

## [x] Task 6: 调试与验证
- **Priority**: P1
- **Depends On**: Task 5
- **Description**: 
  - 编译并上传固件
  - 验证所有功能正常工作
  - 记录调试日志
- **Acceptance Criteria Addressed**: 所有 AC
- **Test Requirements**:
  - `programmatic` TR-6.1: 编译成功无错误
  - `human-judgment` TR-6.2: 界面显示正确
  - `human-judgment` TR-6.3: 按键操作正常
- **Notes**: 需要用户协助测试实际设备

## [ ] Task 7: 代码优化与文档更新
- **Priority**: P2
- **Depends On**: Task 6
- **Description**: 
  - 优化代码结构，提取重复逻辑
  - 更新 PIN_MAPPING.md 等文档
  - 添加必要注释
- **Acceptance Criteria Addressed**: NFR-3
- **Test Requirements**:
  - `human-judgment` TR-7.1: 代码结构清晰
  - `human-judgment` TR-7.2: 文档更新及时
- **Notes**: 可选任务，根据实际情况决定是否执行
