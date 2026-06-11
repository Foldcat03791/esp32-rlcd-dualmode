# ESP32 RLCD TODO同步功能 - 实现计划

## [ ] Task 1: 修改全局变量支持动态TODO列表
- **Priority**: P0
- **Depends On**: None
- **Description**: 
  - 将硬编码的3个TODO项改为动态数组（支持10个）
  - 每个TODO项包含：标题、完成状态
  - 使用ESPhome的globals存储TODO列表
- **Acceptance Criteria Addressed**: AC-1, AC-2, AC-3, AC-4
- **Test Requirements**:
  - `programmatic` TR-1.1: 全局变量支持至少10个TODO项
  - `programmatic` TR-1.2: 每个TODO项包含标题字符串和布尔完成状态
- **Notes**: 使用std::vector或数组存储TODO项

## [ ] Task 2: 添加ESPHome API服务调用
- **Priority**: P0
- **Depends On**: Task 1
- **Description**: 
  - 添加API服务用于添加TODO项
  - 添加API服务用于编辑TODO项
  - 添加API服务用于删除TODO项
  - 添加API服务用于查询TODO列表
- **Acceptance Criteria Addressed**: AC-1, AC-2, AC-3, AC-4, AC-5
- **Test Requirements**:
  - `programmatic` TR-2.1: API调用`add_todo`能添加新任务
  - `programmatic` TR-2.2: API调用`edit_todo`能修改任务内容
  - `programmatic` TR-2.3: API调用`delete_todo`能删除任务
  - `programmatic` TR-2.4: API调用`get_todos`能返回完整列表
- **Notes**: 使用ESPHome的api服务组件

## [ ] Task 3: 修改显示逻辑支持动态列表
- **Priority**: P0
- **Depends On**: Task 1
- **Description**: 
  - 修改display lambda函数支持动态数量的TODO项
  - 实现滚动显示（当超过屏幕显示数量时）
  - 保持赛博朋克风格的UI设计
- **Acceptance Criteria Addressed**: AC-1, AC-2, AC-3, AC-6
- **Test Requirements**:
  - `programmatic` TR-3.1: 显示逻辑能正确渲染动态数量的TODO项
  - `human-judgment` TR-3.2: 显示效果保持原有的赛博朋克风格
- **Notes**: 需要考虑屏幕空间限制（400x300分辨率）

## [ ] Task 4: 添加数据持久化存储
- **Priority**: P1
- **Depends On**: Task 1
- **Description**: 
  - 使用ESPHome的preferences组件存储TODO列表
  - 实现TODO列表的保存和加载
  - 确保重启后数据不丢失
- **Acceptance Criteria Addressed**: NFR-3
- **Test Requirements**:
  - `programmatic` TR-4.1: 修改TODO后保存到preferences
  - `programmatic` TR-4.2: 设备重启后能正确恢复TODO列表
- **Notes**: 使用ESPHome的preferences组件实现持久化

## [ ] Task 5: 添加远程修改后的自动刷新机制
- **Priority**: P1
- **Depends On**: Task 2, Task 3
- **Description**: 
  - 在API服务调用后触发显示刷新
  - 实现5秒内自动更新显示
  - 避免频繁刷新影响性能
- **Acceptance Criteria Addressed**: AC-6
- **Test Requirements**:
  - `human-judgment` TR-5.1: 远程修改后5秒内屏幕自动更新
  - `human-judgment` TR-5.2: 刷新不影响正常使用体验
- **Notes**: 使用component.update触发显示刷新

## [ ] Task 6: 测试和验证
- **Priority**: P1
- **Depends On**: Task 1-5
- **Description**: 
  - 编译测试：确保代码能正常编译
  - 功能测试：验证所有API调用正常工作
  - 集成测试：验证硬件上的完整功能
- **Acceptance Criteria Addressed**: 所有AC
- **Test Requirements**:
  - `programmatic` TR-6.1: 代码编译通过无错误
  - `human-judgment` TR-6.2: 硬件上所有功能正常工作
- **Notes**: 需要实际硬件测试
