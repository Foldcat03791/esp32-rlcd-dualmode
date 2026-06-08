# UI Design Specification

## 概述

本规范基于 Waveshare ESP32-S3-RLCD-4.2 (ST7305) 400x300 显示屏的 ESPHome 配置。

---

## 1. 硬件参数

| 参数 | 值 |
|------|-----|
| 屏幕分辨率 | 400 x 300 |
| 驱动芯片 | ST7305 |
| 通信接口 | SPI |
| 数据率 | 1 MHz |
| 显示类型 | 反射式 LCD (RLCD) |

---

## 2. 字体系统

使用 VT323 等宽像素字体（gfonts://VT323）：

| 字体 ID | 字号 | 用途 |
|---------|------|------|
| `font_display` | 40px | 时间、状态文字 |
| `font_body` | 24px | 传感器数值、SUCCESS/ERROR |
| `font_label` | 18px | 日期、天气、标签、操作提示 |
| `font_footer` | 12px | 底部提示文字 |

### 加粗效果

通过绘制两次偏移实现加粗（仅用于主要元素）：

```cpp
it.print(100, 36, id(font_display), fg, TextAlign::TOP_CENTER, time_buf);
it.print(101, 37, id(font_display), fg, TextAlign::TOP_CENTER, time_buf);  // 偏移 +1,+1
```

**加粗策略**：
- ✅ 加粗：时间显示 (12:34)、WORK 状态文字 (IDLE/GEN/BUILD/OK/ERR)、SUCCESS/ERROR
- ❌ 不加粗：日期、天气、传感器数值、任务列表、底部提示

---

## 3. 颜色系统

```cpp
auto bg = COLOR_ON;    // 深色背景 (黑色)
auto fg = COLOR_OFF;   // 前景色 (白色)
```

| 元素 | 背景 | 前景 |
|------|------|------|
| 主界面 | 黑色 | 白色 |
| 选中行 | 白色 | 黑色（反色） |

---

## 4. 边框系统

### 外边框
- 4px 实线边框（上下左右）

### 角线装饰
- 4px 对角线连接

### 分隔线
- 2px 实线（水平分隔）

### 竖线分隔
- 2px 实线（左右分栏）

---

## 5. 图标系统

图标存放在 `icons/` 目录，使用 SVG 格式，resize 为 BINARY 类型。

### 天气图标 (36x36)

| 天气状况 | 图标 ID | SVG 文件 |
|----------|---------|----------|
| 晴天 | `ic_sun` | weather-sunny.svg |
| 阴天 | `ic_cloud` | cloud.svg |
| 多云 | `ic_cloud_sun` | weather-partly-cloudy.svg |
| 雨天 | `ic_rain` | weather-rainy.svg |
| 雪天 | `ic_snow` | weather-snowy.svg |
| 雷暴 | `ic_thunder` | weather-lightning.svg |
| 雾天 | `ic_fog` | weather-fog.svg |
| 夜晚 | `ic_night` | weather-night.svg |

### 传感器图标 (24x24)

| 类型 | 图标 ID | SVG 文件 |
|------|---------|----------|
| 温度 | `ic_thermometer` | thermometer.svg |
| 湿度 | `ic_humidity` | droplets.svg |
| 电池 | `ic_battery` | battery.svg |
| 充电 | `ic_battery_charging` | battery-charging.svg |

### 状态图标 (20x20)

| 类型 | 图标 ID | SVG 文件 |
|------|---------|----------|
| 勾选 | `ic_check` | check-circle.svg |

### 图标使用示例

```yaml
# 反色显示（在黑色背景上显示白色图标）
it.image(282, 32, id(ic_sun), fg, bg);
```

---

## 6. 布局规格

### LIFE 模式布局

```
+--------------------------------------------------+
|                                         W:ON B:ON|
|                                                  |
|        +---------------+    +---------------+    |
|        |    12:34     | || |      O        |    |
|        | 2024.06.08   | || |    28C Sunny |    |
|        |    [SUN]     | || |              |    |
|        +---------------+    +---------------+    |
|  ------------------------------------------------|
|        |  🌡️  |  💧  |  🔋  |                      |
|        | 25.0C|  60% |  85% |                      |
|  ------------------------------------------------|
|  [TASKS]                                         |
|    ✅ BUY MILK                                   |
|    REVIEW PR                                     |
|    WEEKLY REPORT                                 |
|  ------------------------------------------------|
|              BOOT=MODE KEY=SELECT               |
+--------------------------------------------------+
```

### WORK 模式布局

```
+--------------------------------------------------+
| WORK                                    BAT:85%  |
|  ─────                                   ─────    |
|                                                  |
|                     IDLE                         |
|                   ELAPSED: 0s                    |
|  ------------------------------------------------|
|                                                  |
|                  [进度条]                        |
|                    25%                           |
|  ------------------------------------------------|
|              project :: branch                    |
|  ------------------------------------------------|
|              KEY LONG > START                    |
|  ------------------------------------------------|
|                   BOOT=MODE                      |
|               LONG BOOT > LIFE                   |
+--------------------------------------------------+
```

---

## 7. 坐标参考

### LIFE 模式关键坐标

| 元素 | X | Y |
|------|---|---|
| 顶部状态 | 388 | 6 |
| 时间 | 100 | 36 |
| 日期 | 100 | 76 |
| 竖线分隔 | 200 | 32 |
| 天气图标 | 282 | 32 |
| 天气文字 | 300 | 78 |
| 第一分隔线 | 8 | 108 |
| 三列图标 | ~66/200/333 | 114 |
| 三列数值 | ~66/200/333 | 140 |
| 第二分隔线 | 8 | 164 |
| [TASKS] | 16 | 168 |
| 待办项 1 | 20 | 194 |
| 待办项 2 | 20 | 215 |
| 待办项 3 | 20 | 236 |
| 底部 | 200 | 282 |

---

## 8. 传感器数据

```yaml
# 室内温度
sensor:
  - platform: shtc3
    id: temp_sensor
    # 范围: -40 ~ 125 C

# 室内湿度
sensor:
  - platform: shtc3
    id: hum_sensor
    # 范围: 0 ~ 100 %

# 电池电压
sensor:
  - platform: adc
    id: bat_voltage
    # 范围: 3.0 ~ 4.2V
    # 百分比计算: (voltage - 3.0) * 100 / 1.2
```

---

## 9. 图标文件列表

```
icons/
├── weather-sunny.svg        # ☀️ 晴天
├── weather-partly-cloudy.svg # ⛅ 多云
├── weather-cloudy.svg       # ☁️ 阴天
├── weather-rainy.svg        # 🌧️ 雨天
├── weather-snowy.svg        # ❄️ 雪天
├── weather-lightning.svg    # ⚡ 雷暴
├── weather-fog.svg          # 🌫️ 雾天
├── weather-night.svg        # 🌙 夜晚
├── cloud.svg               # ☁️ 云朵
├── thermometer.svg         # 🌡️ 温度
├── droplets.svg            # 💧 湿度
├── battery.svg             # 🔋 电池
├── battery-charging.svg    # ⚡ 充电
├── check-circle.svg        # ✅ 勾选
├── alarm.svg               # 闹钟
├── alert-circle.svg        # 警告圆
├── alert-triangle.svg      # 三角警告
├── x-circle.svg            # ❌ 错误
├── clock.svg               # 时钟
├── terminal.svg            # 终端
├── cpu.svg                 # CPU
├── microchip.svg           # 芯片
├── cog.svg                 # 设置
├── wifi.svg                # WiFi
├── bluetooth.svg           # 蓝牙
├── radio-tower.svg         # 信号塔
├── signal.svg              # 信号
├── power.svg               # 电源
├── harddisk.svg            # 硬盘
└── memory-stick.svg        # 内存
```

---

## 10. 调试技巧

### 刷新策略
```yaml
update_interval: never  # 静态显示，不自动刷新
```

### 手动刷新
```yaml
on_boot:
  - display.my_display.update()
```

### 清除缓存
```bash
rm -rf .esphome
esphome compile dual_mode.yaml --erase-all
```

---

## 11. 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0.0 | - | 基础 UI 实现 |
| v1.1.0 | 2024-06-08 | 添加 MDI 图标、优化加粗效果、图标化传感器标签 |

---

*最后更新: 2024-06-08*
