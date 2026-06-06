# PCF85063 RTC 时钟配置

## 概述

PCF85063 是 NXP 生产的 CMOS 实时时钟 (RTC) 和日历芯片，优化了低功耗特性，适用于电池供电设备。

---

## 芯片特性

| 特性 | 说明 |
|------|------|
| 工作电压 | 1.2V - 5.5V |
| 工作电流 | 0.25μA (典型值) |
| 精度 | ±20 ppm (-40°C 至 +85°C) |
| 温度补偿 | 支持外接 NTC |
| I2C 地址 | 0x51 |
| 闰年自动处理 | 支持 |
| 双闹钟 | Alarm 0 / Alarm 1 |
| 定时器 | 1Hz / 60Hz / 1kHz / 4.096kHz |

---

## ESPHome 配置

### 基本配置

```yaml
time:
  - platform: pcf85063
    id: my_time
    address: 0x51
    update_interval: 15min
```

### 完整配置示例

```yaml
esphome:
  on_boot:
    then:
      # 启动时从 RTC 读取时间
      - pcf85063.read_time:
          id: my_time

time:
  # PCF85063 RTC
  - platform: pcf85063
    id: my_time
    address: 0x51
    # 不自动同步，依赖外部时间源
    update_interval: never
    
  # 网络时间同步
  - platform: homeassistant
    id: ha_time
    on_time_sync:
      then:
        # 网络同步成功后写入 RTC
        - pcf85063.write_time:
            id: my_time
```

---

## I2C 配置

```yaml
i2c:
  sda: GPIO13
  scl: GPIO14
  scan: true
  id: bus_a
```

---

## API 操作

### 写入时间到 RTC

```yaml
on_...:
  - pcf85063.write_time:
      id: my_time
```

### 从 RTC 读取时间

```yaml
on_...:
  - pcf85063.read_time:
      id: my_time
```

---

## 闹钟配置

### ESP-IDF 示例

```c
// 设置闹钟 (每天 8:00)
pcf85063_set_alarm(8, 0, 0, 0xFF, true);

// 闹钟中断回调
void IRAM_ATTR rtc_isr(void)
{
    // 处理闹钟事件
    alarm_triggered = true;
}
```

### 闹钟参数

```c
bool setAlarm0(
    uint8_t hour,           // 小时 (0-23)
    uint8_t minute,         // 分钟 (0-59)
    uint8_t second,         // 秒 (0-59)
    uint8_t day_or_weekday, // 0x01-0x31: 日; 0x01-0x07: 周; 0xFF: 忽略
    bool enable             // 是否启用
);
```

---

## 定时器配置

PCF85063 支持周期性定时器输出：

| 频率 | 说明 |
|------|------|
| 4.096kHz | 高速定时 |
| 1kHz | 毫秒级定时 |
| 60Hz | 约 16.67ms |
| 1Hz | 秒脉冲 |

### ESP-IDF 配置

```c
// 设置 1Hz 方波输出
pcf85063_set_clock_out(PCF85063_CLKOUT_1HZ);
```

---

## 后备电池

### RTC 独立供电

ESP32-S3-RLCD-4.2 支持 RTC 独立电源：

```
┌─────────────────────────────────────┐
│         主电源 (USB/电池)           │
│                │                    │
│           ┌────┴────┐               │
│           │  ESP32  │               │
│           │  -S3    │               │
│           └────┬────┘               │
│                │ I2C                │
│           ┌────┴────┐               │
│           │PCF85063 │◄── RTC 电池   │
│           │         │   (CR1220)    │
│           └─────────┘               │
└─────────────────────────────────────┘
```

### 后备电池规格

| 参数 | 值 |
|------|-----|
| 电池类型 | CR1220 / CR1225 |
| 电压 | 3V |
| 容量 | 40-50mAh |
| 守时时间 | 数年 |

---

## 寄存器映射

| 寄存器 | 地址 | 说明 |
|--------|------|------|
| Control_1 | 0x00 | 控制寄存器 1 |
| Control_2 | 0x01 | 控制寄存器 2 |
| Offset | 0x02 | 频率偏移校准 |
| RAM_byte | 0x03 | 用户 RAM |
| Seconds | 0x04 | 秒 (BCD) |
| Minutes | 0x05 | 分 (BCD) |
| Hours | 0x06 | 时 (BCD) |
| Days | 0x07 | 日 (BCD) |
| Weekdays | 0x08 | 星期 (BCD) |
| Months | 0x09 | 月 (BCD) |
| Years | 0x0A | 年 (BCD) |
| Alarm_sec | 0x0B | 闹钟秒 |
| Alarm_min | 0x0C | 闹钟分 |
| Alarm_hour | 0x0D | 闹钟时 |
| Alarm_day | 0x0E | 闹钟日 |
| Alarm_weekday | 0x0F | 闹钟星期 |
| Timer_value | 0x10 | 定时器值 |
| Timer_mode | 0x11 | 定时器模式 |

---

## 时间格式

### BCD 编码

PCF85063 使用 BCD (Binary Coded Decimal) 编码：

```c
// BCD 转 十进制
uint8_t bcd_to_dec(uint8_t bcd) {
    return (bcd >> 4) * 10 + (bcd & 0x0F);
}

// 十进制转 BCD
uint8_t dec_to_bcd(uint8_t dec) {
    return ((dec / 10) << 4) | (dec % 10);
}
```

### 时间设置示例

```c
// 设置时间: 2024-01-15 10:30:00
pcf85063_set_time(2024, 1, 15, 10, 30, 0);
```

---

## 精度校准

### 频率偏移校准

通过 Offset 寄存器 (0x02) 校准晶振频率：

| 偏移值 | 频率调整 |
|--------|----------|
| 0x00 | 无偏移 |
| 0x01 - 0x7F | +2.17ppm 至 +275ppm |
| 0x80 | 无效 |
| 0x81 - 0xFF | -2.17ppm 至 -275ppm |

### 温度补偿

外接 NTC 热敏电阻可实现温度补偿：

- NTC 规格：10kΩ @ 25°C, B=3975K
- 补后精度：±5 ppm (年误差 < ±15 秒)

---

## 低功耗唤醒

PCF85063 可作为 ESP32 深度睡眠唤醒源：

```c
// 配置闹钟唤醒
esp_sleep_enable_ext0_wakeup(GPIO_NUM_XX, 0);  // INT 引脚低电平唤醒

// 设置闹钟
pcf85063_set_alarm(8, 0, 0, 0xFF, true);

// 进入深度睡眠
esp_deep_sleep_start();
```

---

## 参考资源

- [PCF85063 数据手册](https://www.nxp.com/docs/en/data-sheet/PCF85063A.pdf)
- [ESPHome PCF85063 组件](https://esphome.io/components/time/pcf85063/)
- [ESPP PCF85063 驱动](https://esp-cpp.github.io/espp/rtc/pcf85063.html)
