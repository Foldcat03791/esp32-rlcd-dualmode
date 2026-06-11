# ESP32 RLCD TODO同步功能 - 产品需求文档

## Overview
- **Summary**: 完善TODO列表功能，支持通过手机或其他方式远程添加、编辑TODO项，并能将硬件上标记的完成状态同步回远程设备。
- **Purpose**: 解决当前TODO列表只能在硬件上查看和操作的限制，实现跨设备同步。
- **Target Users**: ESP32 RLCD设备用户，希望通过手机App或Web界面管理TODO列表。

## Goals
- 支持远程添加、编辑TODO项
- 支持远程删除TODO项
- 硬件上标记的完成状态能同步到远程
- 远程修改的TODO列表能实时同步到硬件显示

## Non-Goals (Out of Scope)
- 复杂的任务优先级管理
- 任务分类/标签功能
- 任务提醒功能
- 多用户协作功能

## Background & Context
- 当前TODO功能仅支持3个硬编码任务
- 当前完成状态仅保存在设备本地
- 需要通过网络实现跨设备同步
- 设备已支持WiFi连接和ESPHome API

## Functional Requirements
- **FR-1**: 支持通过ESPHome API远程添加TODO项
- **FR-2**: 支持通过ESPHome API远程编辑TODO项
- **FR-3**: 支持通过ESPHome API远程删除TODO项
- **FR-4**: 支持通过ESPHome API查询TODO列表
- **FR-5**: 硬件上标记完成的任务状态能通过API同步
- **FR-6**: 远程修改后设备能自动刷新显示

## Non-Functional Requirements
- **NFR-1**: 同步延迟不超过5秒
- **NFR-2**: 支持至少10个TODO项（当前为3个）
- **NFR-3**: 数据持久化存储，重启后不丢失

## Constraints
- **Technical**: ESPHome框架、ESP32-S3硬件、RLCD显示屏(400x300)
- **Dependencies**: ESPHome API、WiFi连接

## Assumptions
- 用户设备已连接WiFi
- 用户能访问ESPHome Dashboard或类似工具
- ESPHome API已启用

## Acceptance Criteria

### AC-1: 远程添加TODO项
- **Given**: 设备已连接WiFi并在线
- **When**: 通过ESPHome API发送添加TODO请求
- **Then**: 新TODO项出现在设备屏幕上
- **Verification**: `programmatic`

### AC-2: 远程编辑TODO项
- **Given**: 设备上已有TODO项
- **When**: 通过ESPHome API发送编辑请求
- **Then**: 设备屏幕上显示更新后的内容
- **Verification**: `programmatic`

### AC-3: 远程删除TODO项
- **Given**: 设备上已有TODO项
- **When**: 通过ESPHome API发送删除请求
- **Then**: 该TODO项从设备屏幕上消失
- **Verification**: `programmatic`

### AC-4: 查询TODO列表
- **Given**: 设备已连接WiFi并在线
- **When**: 通过ESPHome API查询TODO列表
- **Then**: 返回包含所有TODO项及其状态的列表
- **Verification**: `programmatic`

### AC-5: 完成状态同步
- **Given**: 设备上有未完成的TODO项
- **When**: 在硬件上标记任务完成
- **Then**: 通过API查询时该任务显示为已完成
- **Verification**: `programmatic`

### AC-6: 实时刷新显示
- **Given**: 设备正在显示TODO列表
- **When**: 通过API修改TODO列表
- **Then**: 设备屏幕在5秒内自动更新显示
- **Verification**: `human-judgment`

## Open Questions
- [ ] 是否需要支持TODO项的拖拽排序？
- [ ] 是否需要支持任务截止日期？
