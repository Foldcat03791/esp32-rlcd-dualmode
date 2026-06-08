# LIFE 模式 UI 优化方案 (v1.1 设计讨论)

> 屏幕：400×300 单色反射 RLCD  
> 目标：电池图形化 + WiFi/蓝牙状态 + 天气信息 + 保留核心时间/待办

---

## 1. 硬编码能力评估

### 1.1 ESPHome 原生支持的图形绘制

| 能力 | API | 能做吗？ |
|------|-----|----------|
| 矩形外框 | `it.rectangle(x, y, w, h, color)` | ✅ |
| 实心矩形 | `it.filled_rectangle(x, y, w, h, color)` | ✅ |
| 水平线 | `it.horizontal_line(x, y, w, color)` | ✅ |
| 垂直线 | `it.vertical_line(x, y, h, color)` | ✅ |
| 任意线 | `it.line(x1, y1, x2, y2, color)` | ✅ |
| 圆/弧 | `it.circle(x, y, r, color)` | ✅ |
| 实心圆 | `it.filled_circle(x, y, r, color)` | ✅ |

结论：**可以用基本几何图形组合出电池图标、WiFi 图标、天气图标等**，不需要额外字体或图片资源。

### 1.2 硬件/数据可行性

| 功能 | 需要什么？ | ESPHome 支持？ | 备注 |
|------|-----------|---------------|------|
| WiFi 状态 | `wifi_signal` sensor | ✅ 有 | 可获取 RSSI 值或连接状态 |
| 蓝牙状态 | `esp32_ble_tracker` | ✅ 有 | 或者简单显示"BT 扫描中/无设备" |
| 天气信息 | HTTP 请求 + JSON 解析 | ✅ `http_request` + `json` | 需要 Home Assistant 或自建天气 API |

---

## 2. 三种布局方案

### 方案 A：紧凑型（推荐，信息最丰富）

```
y=0    ┌────────────────────────────────────────────────┐
y=6    │  LIFE  📶 [100%]  🔋 [=====]  88%             │ ← 状态图标行（18px高）
y=24   ├────────────────────────────────────────────────┤
y=30   │              10:28                              │ ← 大字时间（52px）
y=82   │           2026-06-07  Sun                      │ ← 日期
y=102  ├────────────────────────────────────────────────┤
y=110  │  室内  28.3°C  55%  │  🌤 Sunny 26°C 52%      │ ← 室内/室外对比
y=126  │  (SHTCx 传感器)      │  风向: E  12km/h       │
y=140  ├────────────────────────────────────────────────┤
y=148  │  TASKS                                         │
y=166  │  [X] Buy milk                                  │ ← 选中反白
y=186  │  [ ] Review PR                                 │
y=206  │  [ ] Submit weekly report                      │
y=226  ├────────────────────────────────────────────────┤
y=234  │  Sensor OK  WiFi OK  BT OK                     │ ← 状态行
y=234  │                    BOOT:Mode  KEY:Select       │ ← 按键提示
y=254  │                    Long BOOT > WORK mode       │
y=274  └────────────────────────────────────────────────┘
       ↑ 400px 宽                                      ↑ y=300
```

**特点：**
- 顶部：纯图标 + 关键数值，一眼扫描
- 主体：时间仍占主导，室内室外并排对比
- 底部：状态信息 + 按键提示

---

### 方案 B：时间优先型（天气信息精简）

```
y=0    ┌────────────────────────────────────────────────┐
y=6    │  LIFE  📶  📡  🔋 [======] 88%                │ ← 状态图标
y=24   ├────────────────────────────────────────────────┤
y=30   │                     10:28                       │ ← 超级大字（60px+）
y=90   │                  2026-06-07 Sun                │
y=110  │                    18:24:03                     │ ← 精确到秒
y=128  ├────────────────────────────────────────────────┤
y=136  │  🌤 Sunny 26°C  (室内 28°C)                    │ ← 天气一行
y=156  │  湿度 52%  风速 12km/h                         │
y=176  ├────────────────────────────────────────────────┤
y=184  │  TASKS                                         │
y=202  │  [X] Buy milk                                  │
y=222  │  [ ] Review PR                                 │
y=242  │  [ ] Submit weekly report                      │
y=262  ├────────────────────────────────────────────────┤
y=270  │  Sensor OK              BOOT:Mode  KEY:Select  │
y=270  │                         Long BOOT > WORK mode  │
y=290  └────────────────────────────────────────────────┘
```

**特点：**
- 时间更大更醒目，适合作为"时钟屏"
- 天气只显示当前状况，不做对比

---

### 方案 C：功能优先型（减少信息密度）

```
y=0    ┌────────────────────────────────────────────────┐
y=6    │  LIFE  📶 WiFi  🔋 88%                         │ ← 简洁顶部
y=24   ├────────────────────────────────────────────────┤
y=30   │          10:28  2026-06-07 Sun                 │
y=50   ├────────────────────────────────────────────────┤
y=58   │  🌤 Sunny  26°C / 20°C  H:52%  W:12km/h        │ ← 天气详情
y=78   │  今日预报: 多云转晴，20-28°C                   │ ← 文本摘要
y=98   ├────────────────────────────────────────────────┤
y=106  │  室内: 28.3°C / 55%  |  电池: 88% (4.06V)     │
y=126  ├────────────────────────────────────────────────┤
y=134  │  TASKS (3)                                     │
y=152  │  [X] Buy milk                                  │
y=172  │  [ ] Review PR                                 │
y=192  │  [ ] Submit weekly report                      │
y=212  ├────────────────────────────────────────────────┤
y=220  │  ⏰ 下次刷新 00:04:58                           │ ← 时间相关
y=240  ├────────────────────────────────────────────────┤
y=248  │  Sensor OK  WiFi OK  BT OK                     │
y=248  │                    BOOT:Mode  KEY:Select       │
y=268  │                    Long BOOT > WORK mode       │
y=288  └────────────────────────────────────────────────┘
```

**特点：**
- 时间缩小（和日期同行），腾出更多空间给天气摘要
- 增加"文本摘要"区域，天气信息更丰富

---

## 3. 图标设计规范（单色 RLCD 优化）

### 3.1 电池图标

```
方案 A（实心填充）：         方案 B（分段）：

┌──────────────┐             ┌──────────────┐
│█▌            │             │█████░░░░░░░░ │  ← 5/10 格 = 50%
│████████████  │             │███████████▌  │  ← 90%+
│██████████████│             │██████████████│  ← 100%
└──────────────┘             └──────────────┘
 88%                          88%

尺寸: 40px 宽 × 18px 高（顶部状态栏）
```

### 3.2 WiFi 图标

```
信号强:  ███████           信号中:  ░░██░
         █████░                      ░░██░
         ██░░░█                      ░░██░
         ██░░░█                      ██░░░█
         ██████                      ██████

连接中:  ⏳ (闪烁)           断开:  📶 ×
```

### 3.3 蓝牙图标

```
已连接:  ⟷  BT              扫描中:  ⋈ BT
未启用:  ○  BT
```

### 3.4 天气图标（用几何图形组合）

```
晴天 ☀:
   ███
  █████
  █████           ← 用 filled_circle + 线条光芒
   ███
  █   █

多云 ⛅:
  ░█████░
 █████████        ← 圆 + 弧线
  ░█████░
   ██░░           ← 下方画太阳的一半

阴天 ☁:
   ███████
  ██████████
 █████████████    ← 组合的 filled_circle 连成云朵
  █████████

小雨 🌧:
   █████
  ████████       ← 云朵 + 3个小雨点（short vertical line）
   █████
    ░   ░   ░   ← 雨滴（用 short vertical line）

雪天 ❄:
    █
 █  █  █
   ███           ← 复杂，建议简化为小圆点 + 线条
 █  █  █
    █
```

---

## 4. 技术实现要点

### 4.1 新增配置项

```yaml
# WiFi 状态
sensor:
  - platform: wifi_signal
    id: wifi_rssi
    update_interval: 30s

# 天气 (需要 HTTP 请求)
http_request:
  id: http_client
  useragent: esphome/rlcd

# 通过 text_sensor 存储天气信息
text_sensor:
  - platform: template
    id: weather_condition
    initial_value: "Sunny"

# 天气数值 sensor
  - platform: template
    id: outdoor_temp
    unit_of_measurement: "°C"
    initial_value: "26.0"
  - platform: template
    id: outdoor_hum
    unit_of_measurement: "%"
    initial_value: "52"
  - platform: template
    id: wind_speed
    unit_of_measurement: "km/h"
    initial_value: "12"

# 蓝牙状态
esp32_ble_tracker:
  scan_parameters:
    interval: 1100ms
    window: 1100ms
    active: true

binary_sensor:
  - platform: status
    id: system_status

# 蓝牙连接数
sensor:
  - platform: template
    id: ble_device_count
    initial_value: "0"
```

### 4.2 Display Lambda 重构思路

```cpp
// 1. 顶部状态栏 (绘制图标函数)
draw_wifi_icon(x, y, rssi_db);          // 根据 RSSI 值画 1-4 格
draw_bluetooth_icon(x, y, connected);   // 实心/空心 + 文本
draw_battery_icon(x, y, percentage);    // 填充矩形 + 顶部小凸起

// 2. 天气图标
draw_weather_icon(x, y, condition);     // 0=Sunny, 1=Cloudy, 2=Rainy, ...

// 3. 布局常量集中管理
const int TOP_H = 24;
const int BOTTOM_H = 36;
const int TIME_Y = 30;
const int DATE_Y = 96;
const int WEATHER_Y = 126;
const int TASKS_Y = 178;
```

### 4.3 图标绘制函数

```cpp
// 电池图标 (36px宽 × 14px高)
void draw_battery(DisplayBuffer& it, int x, int y, int pct) {
  it.rectangle(x, y, 32, 14, fg);           // 主体
  it.filled_rectangle(x + 32, y + 4, 2, 6, fg); // 正极小凸起
  int fill = (pct * 28) / 100;              // 内部 28px 可用
  it.filled_rectangle(x + 2, y + 2, fill, 10, fg);
}

// WiFi 图标 (16px宽 × 14px高)
void draw_wifi(DisplayBuffer& it, int x, int y, int bars) {
  // bars = 1-4, 用弧线表示信号强度
  // 底部小圆点表示设备
}

// 天气图标 (32px × 32px)
void draw_weather(DisplayBuffer& it, int x, int y, int condition) {
  switch(condition) {
    case 0: // Sunny - 圆形光芒
      it.filled_circle(x + 16, y + 16, 8, fg);
      for (int i = 0; i < 8; i++) {
        float angle = i * M_PI / 4;
        int lx = x + 16 + cos(angle) * 12;
        int ly = y + 16 + sin(angle) * 12;
        it.line(x + 16 + cos(angle) * 9, y + 16 + sin(angle) * 9,
                lx, ly, fg);
      }
      break;
    case 1: // Cloudy - 三个重叠圆
      it.filled_circle(x + 10, y + 18, 8, fg);
      it.filled_circle(x + 18, y + 16, 10, fg);
      it.filled_circle(x + 26, y + 18, 8, fg);
      break;
    case 2: // Rainy - 云 + 雨滴
      ...
      break;
  }
}
```

---

## 5. 实施步骤

### 阶段 1：图标系统（先做视觉基础）
1.1 提取图标绘制函数，统一放置在 lambda 前部
1.2 画电池图标（替代文字 BAT）
1.3 画 WiFi 图标（静态，先不接入真实 RSSI）
1.4 画蓝牙图标（静态，先不接入 BLE）
1.5 画天气图标（固定 Sunny，先不接入 API）
1.6 上传测试 → 确认视觉效果

### 阶段 2：数据接入（让图标动起来）
2.1 添加 `wifi_signal` sensor → WiFi 图标显示真实信号
2.2 添加 `esp32_ble_tracker` → 蓝牙图标显示状态
2.3 添加天气 HTTP 请求（需讨论数据源）
2.4 上传测试 → 确认数据正确

### 阶段 3：整体布局优化
3.1 确认最终布局方案（方案 A/B/C）
3.2 调整坐标和间距
3.3 确保所有信息在 400×300 内完整显示
3.4 上传测试 → 最终版本

---

## 6. 待讨论的问题

### 6.1 天气数据源
ESPHome 没有内建天气平台，需要您选择：

| 方案 | 需要什么 | 优点 | 缺点 |
|------|---------|------|------|
| **A. Home Assistant** | 已有 HA 实例 | ✅ ESPHome 原生支持 | 需要 HA 一直在线 |
| **B. OpenWeather API** | 免费 API Key | ✅ 独立工作，无需 HA | 需要 HTTP + JSON 解析 |
| **C. 其他天气 API** | 自建服务 | ✅ 完全可控 | 工作量大 |
| **D. 固定演示数据** | 无需 API | ✅ 零依赖 | 不是真正的天气 |

建议先用 **D**（演示数据）完成视觉效果，再讨论接入真实数据。

### 6.2 蓝牙显示什么？
- **简单模式**：只显示 "BT On" / "BT Off"
- **设备计数**：显示附近蓝牙设备数量（需开启 BLE 扫描，耗电）
- **连接状态**：显示是否有设备已连接

建议先用 **简单模式**。

### 6.3 布局方案选择
您倾向于哪个方案？（A 紧凑 / B 时间优先 / C 功能优先）

### 6.4 室内温湿度 vs 天气温湿度
当前显示的是 **SHTCx 传感器的室内数据**。  
新增天气后需要明确"哪些是室内，哪些是室外"，避免混淆。

---

## 7. 风险与注意事项

| 风险 | 影响 | 对策 |
|------|------|------|
| HTTP 天气请求耗电 | 🔋 电池续航下降 | 拉长更新间隔（15-30min） |
| BLE 扫描耗电 | 🔋 电池续航下降 | 仅扫描 30s，平时关闭 |
| display lambda 代码过长 | 📝 维护困难 | 拆分函数，集中布局常量 |
| 400×300 空间不足 | 🚫 信息重叠 | 严格测试坐标，可能需要缩减待办数 |
| 单色天气图标辨识度 | 👁 看不清 | 用粗线条、大图标，避免复杂细节 |
