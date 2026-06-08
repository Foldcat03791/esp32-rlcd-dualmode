# LIFE 模式 UI 优化 v1.1 - 产品需求文档 (PRD)

## Overview
- **Summary**: 优化 LIFE 模式界面，实现分屏布局（左侧时间日期、右侧天气信息），使用标准图标库展示电池、WiFi、蓝牙状态，接入 OpenWeather API 获取天气数据。
- **Purpose**: 提升界面信息密度和视觉体验，让用户一眼获取时间、日期、室内外环境、网络状态等关键信息。
- **Target Users**: ESP32 + ST7305 RLCD 设备用户

## Goals
- 实现左右分屏布局：左侧显示时间日期，右侧显示天气信息
- 顶部状态栏简洁设计：WiFi 和电池图标靠右显示
- 接入 OpenWeather API，每小时刷新一次天气数据
- 蓝牙状态以简单模式显示（开启/关闭）
- 使用标准图标库（如 Material Icons）替代自定义绘制图标

## Non-Goals (Out of Scope)
- 不实现蓝牙设备扫描或连接功能
- 不实现多天天气预报
- 不修改 WORK 模式界面
- 不添加语音播报功能

## Background & Context
- 当前设备：ESP32-S3 DevKitC-1 + ST7305 RLCD 400×300 单色反射屏
- 当前界面已实现：时间、日期、室内温湿度、待办列表、电池电量（文字显示）
- 用户反馈：需要更直观的状态图标、天气信息、更好的布局组织

## Functional Requirements
- **FR-1**: 顶部状态栏显示 LIFE 标识、WiFi 状态图标、蓝牙状态图标、电池图标及电量百分比
- **FR-2**: 主体区域左侧显示大号时间和日期
- **FR-3**: 主体区域右侧显示天气图标、室外温度、天气描述文本
- **FR-4**: 接入 OpenWeather API 获取天气数据，每小时刷新一次
- **FR-5**: 蓝牙状态以简单模式显示（BT On/BT Off）
- **FR-6**: 使用 Material Icons 字体显示标准图标

## Non-Functional Requirements
- **NFR-1**: 界面刷新间隔保持 5 秒，天气数据刷新间隔 60 分钟
- **NFR-2**: 图标清晰可辨，单色显示效果良好
- **NFR-3**: 代码结构清晰，便于后续维护和扩展

## Constraints
- **Technical**: ESPHome 框架，C++ lambda 表达式限制，单色显示
- **Dependencies**: OpenWeather API Key（用户提供），WiFi 连接

## Assumptions
- 用户已配置 WiFi 连接
- 用户将提供 OpenWeather API Key 和城市 ID
- 设备能够访问互联网获取天气数据

## Acceptance Criteria

### AC-1: 顶部状态栏布局
- **Given**: LIFE 模式界面显示
- **When**: 查看顶部状态栏
- **Then**: 左侧显示 "LIFE" 文字，右侧依次显示 WiFi 图标、蓝牙图标、电池图标及电量百分比
- **Verification**: `human-judgment`
- **Notes**: 图标使用 Material Icons 字体

### AC-2: 左侧时间日期区域
- **Given**: LIFE 模式界面显示
- **When**: 查看主体区域左侧
- **Then**: 显示大号时间（如 10:28）和日期（如 2026-06-07 Sun）
- **Verification**: `human-judgment`

### AC-3: 右侧天气区域
- **Given**: LIFE 模式界面显示，WiFi 已连接
- **When**: 查看主体区域右侧
- **Then**: 显示天气图标、室外温度、天气描述文本
- **Verification**: `human-judgment`

### AC-4: WiFi 状态显示
- **Given**: LIFE 模式界面显示
- **When**: WiFi 连接状态变化
- **Then**: WiFi 图标显示连接状态（信号强度指示）
- **Verification**: `human-judgment`

### AC-5: 蓝牙状态显示
- **Given**: LIFE 模式界面显示
- **When**: 查看蓝牙状态图标
- **Then**: 显示 "BT" 文字，蓝牙启用时图标点亮，关闭时图标灰显
- **Verification**: `human-judgment`

### AC-6: 电池图标显示
- **Given**: LIFE 模式界面显示
- **When**: 查看电池状态
- **Then**: 显示电池图标及电量百分比（如 88%）
- **Verification**: `human-judgment`

### AC-7: 天气数据刷新
- **Given**: WiFi 已连接，OpenWeather API 配置正确
- **When**: 每小时到达刷新时间
- **Then**: 自动获取最新天气数据并更新界面
- **Verification**: `programmatic`（检查日志输出）

## Open Questions
- [ ] 用户需要提供 OpenWeather API Key 和城市 ID
- [ ] 蓝牙功能是否需要实际启用（当前仅显示状态）
