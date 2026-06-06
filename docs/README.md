# ESP32-S3-RLCD-4.2 开发文档

## 设备概述

Waveshare ESP32-S3-RLCD-4.2 是一款基于 ESP32-S3 的 AIoT 开发板，配备：

- **4.2 英寸反射式 LCD** (ST7305, 400×300 单色)
- **双麦克风阵列** (ES7210 ADC)
- **扬声器输出** (ES8311 DAC)
- **温湿度传感器** (SHTC3)
- **RTC 时钟** (PCF85063)
- **18650 电池座** + 充放电管理

---

## 文档目录

### 1. [引脚映射](../PIN_MAPPING.md)

GPIO 引脚分配、外设配置、按键定义。

### 2. [LVGL 界面设计](./UI_DESIGN.md)

- LVGL 架构与三大核心接口
- RLCD 单色显示适配
- 常用控件示例
- 按钮输入映射
- 性能优化

### 3. [电池充放电管理](./POWER_MANAGEMENT.md)

- 电池 ADC 监测
- BQ25896/ETA6096 充电芯片
- 电源路径管理
- 低电量保护
- 充电安全

### 4. [RTC 时钟配置](./RTC_CONFIG.md)

- PCF85063 ESPHome 配置
- 闹钟与定时器
- 后备电池
- 时间格式与校准
- 低功耗唤醒

### 5. [音频配置](./AUDIO_CONFIG.md)

- ES8311 DAC 扬声器输出
- ES7210 ADC 麦克风输入
- 回声消除与噪声抑制
- 语音助手配置
- 音量控制

### 6. [低功耗模式](./LOW_POWER.md)

- DFS 动态频率调节
- Light-sleep 轻睡眠
- Deep-sleep 深度睡眠
- ULP 协处理器
- 电池续航估算

---

## 快速开始

### ESPHome 基础配置

```yaml
esphome:
  name: rlcd-device

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: esp-idf

psram:
  mode: octal
  speed: 80MHz

# I2C 总线
i2c:
  sda: GPIO13
  scl: GPIO14

# SPI 总线 (显示屏)
spi:
  clk_pin: GPIO11
  mosi_pin: GPIO12

# 显示屏
display:
  - platform: st7305_rlcd
    model: WAVESHARE_400X300
    cs_pin: GPIO40
    dc_pin: GPIO5
    reset_pin: GPIO41
```

---

## 外设清单

| 外设 | 芯片 | 接口 | 地址 |
|------|------|------|------|
| 显示屏 | ST7305 | SPI | - |
| DAC | ES8311 | I2S + I2C | 0x18 |
| ADC | ES7210 | I2S + I2C | 0x40 |
| 温湿度 | SHTC3 | I2C | 0x70 |
| RTC | PCF85063 | I2C | 0x51 |

---

## 参考资源

### 官方文档

- [Waveshare 产品页面](https://www.waveshare.com/esp32-s3-rlcd-4.2.htm)
- [ESPHome 设备数据库](https://devices.esphome.io/devices/waveshare-esp32-s3-rlcd-42/)
- [ESP-IDF 编程指南](https://docs.espressif.com/projects/esp-idf/)

### 社区资源

- [Home Assistant 社区讨论](https://community.home-assistant.io/)
- [ESPHome Discord](https://discord.gg/5EHKQnV)
- [GitHub ST7305 驱动](https://github.com/kylehase/ESPHome-ST7305-RLCD)

---

## 更新日志

| 日期 | 更新内容 |
|------|----------|
| 2024-01-XX | 初始文档创建 |
