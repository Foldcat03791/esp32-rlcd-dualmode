# 电池充放电管理

## 概述

ESP32-S3-RLCD-4.2 配备 18650 电池座和充放电管理电路，支持电池供电和 USB 供电双模式。

---

## 硬件配置

### 电池规格

| 参数 | 值 |
|------|-----|
| 电池类型 | 18650 锂电池 |
| 标称电压 | 3.7V |
| 充电截止电压 | 4.2V |
| 放电截止电压 | 3.0V |
| 容量 | 通常 2000-3000mAh |

### 电池 ADC 监测

```yaml
sensor:
  - platform: adc
    id: battery_voltage
    pin: GPIO4
    attenuation: 12db
    filters:
      - multiply: 3.0  # 分压比 1/3
      - lambda: return x;
    update_interval: 60s
```

### 电池电量计算

```yaml
sensor:
  - platform: template
    id: battery_percent
    lambda: |-
      float voltage = id(battery_voltage).state;
      // 线性映射: 3.0V -> 0%, 4.2V -> 100%
      float percent = (voltage - 3.0) / (4.2 - 3.0) * 100.0;
      if (percent < 0) percent = 0;
      if (percent > 100) percent = 100;
      return percent;
    update_interval: 60s
```

---

## BQ25896 充电管理芯片

BQ25896 是 TI 的高集成度 3A 开关模式电池充电管理芯片，适用于单节锂离子/锂聚合物电池。

### 主要特性

| 特性 | 说明 |
|------|------|
| 输入电压 | 3.9V - 14V |
| 充电电流 | 最高 3A |
| 充电效率 | 92.5% @ 2A |
| I2C 控制 | 是 |
| 电源路径管理 | 支持 |
| USB OTG | 支持 |

### ESP-IDF 配置示例

```c
#include "bq25896.h"

// 初始化 I2C
i2c_master_bus_config_t i2c_config = {
    .clk_source = I2C_CLK_SRC_DEFAULT,
    .i2c_port = I2C_NUM_0,
    .scl_io_num = GPIO_NUM_14,
    .sda_io_num = GPIO_NUM_13,
    .glitch_ignore_cnt = 7,
    .flags.enable_internal_pullup = true,
};

// 初始化 BQ25896
bq25896_handle_t bq_handle;
bq25896_init(i2c_bus, &bq_handle);

// 设置充电电流
bq25896_set_charge_current(bq_handle, 2000);  // 2A

// 读取电池电压
float vbat = bq25896_get_battery_voltage(bq_handle);
```

### 关键寄存器

| 寄存器 | 地址 | 说明 |
|--------|------|------|
| REG00 | 0x00 | 输入源控制 |
| REG02 | 0x02 | 充电电流控制 |
| REG03 | 0x03 | 充电控制 |
| REG04 | 0x04 | 充电电压控制 |
| REG0A | 0x0A | 电池电压 ADC |
| REG0B | 0x0B | 系统电压 ADC |

---

## ETA6096 充电芯片

部分 Waveshare 开发板使用 ETA6096 高效充电芯片。

### 特性

- 输入电压：5V
- 充电电流：可调
- 充电效率：> 90%
- 状态指示：CHG 引脚

### 充电状态指示

| CHG LED 状态 | 说明 |
|--------------|------|
| 常亮 | 充电中 |
| 熄灭 | 充满 |
| 快闪 | 充电错误 |

---

## 电源路径管理

### NVDC 架构

BQ25896 采用 NVDC (Narrow VDC) 电源路径管理：

```
┌─────────────────────────────────────────┐
│              VBUS (USB)                 │
│                 │                       │
│            ┌────┴────┐                  │
│            │  Input  │                  │
│            │  Limiter│                  │
│            └────┬────┘                  │
│                 │                       │
│            ┌────┴────┐                  │
│            │  Buck   │                  │
│            │Converter│                  │
│            └────┬────┘                  │
│                 │                       │
│     ┌───────────┼───────────┐          │
│     │           │           │          │
│ ┌───┴───┐  ┌────┴────┐  ┌───┴───┐     │
│ │ SYS   │  │ BATFET  │  │ OTG   │     │
│ │ Output│  │         │  │ Boost │     │
│ └───┬───┘  └────┬────┘  └───────┘     │
│     │           │                      │
│     │      ┌────┴────┐                 │
│     │      │  BAT    │                 │
│     │      │ (电池)  │                 │
│     │      └─────────┘                 │
│     │                                  │
│  系统供电                               │
└─────────────────────────────────────────┘
```

### 优势

1. **无电池启动**：即使没有电池，USB 供电也能正常工作
2. **动态电源分配**：自动平衡系统功耗和充电电流
3. **电池保护**：防止过充、过放、过流

---

## 低电量保护

### ESPHome 配置

```yaml
binary_sensor:
  - platform: template
    id: low_battery
    lambda: |-
      return id(battery_percent).state < 10;
    on_press:
      - logger.log: "低电量警告！"
      # 可选：进入深度睡眠保护电池
      # - deep_sleep.enter: deep_sleep_mode
```

### 电池保护阈值

| 阈值 | 电压 | 说明 |
|------|------|------|
| 满充 | 4.2V | 充电截止 |
| 正常 | 3.7V | 标称电压 |
| 低电量 | 3.3V | 警告阈值 |
| 关机 | 3.0V | 放电截止 |

---

## 充电安全

### 注意事项

1. **电池极性**：确保电池正负极正确连接，WRN 指示灯亮起表示反接
2. **温度监测**：充电时注意电池温度，避免过热
3. **充电环境**：在 0-45°C 环境下充电
4. **电池质量**：使用正规品牌 18650 电池

### JEITA 标准

BQ25896 支持 JEITA 温度监测标准：

| 温度范围 | 充电行为 |
|----------|----------|
| < 0°C | 禁止充电 |
| 0-10°C | 降流充电 (0.1C) |
| 10-45°C | 正常充电 |
| > 45°C | 禁止充电 |

---

## 参考资源

- [BQ25896 数据手册](https://www.ti.com/product/BQ25896)
- [ESP-IDF BQ25896 组件](https://components.espressif.com/components/kodediy/kode_bq25896)
- [Waveshare 电池使用指南](https://docs.waveshare.com/ESP32-S3-Touch-LCD-1.28)
