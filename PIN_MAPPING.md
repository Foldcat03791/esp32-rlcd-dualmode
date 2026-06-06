# Waveshare ESP32-S3-RLCD-4.2 引脚映射

## 设备信息

- **型号**: Waveshare ESP32-S3-RLCD-4.2
- **芯片**: ESP32-S3-WROOM-1-N16R8
- **Flash**: 16MB
- **PSRAM**: 8MB (Octal)
- **显示屏**: ST7305 RLCD 400x300 单色反射式 LCD

---

## GPIO 引脚映射表

| GPIO | 功能 | 说明 |
|------|------|------|
| GPIO0 | BOOT 按键 | 低电平有效, strapping pin |
| GPIO4 | 电池 ADC | 电池电压检测 (分压比 1/3) |
| GPIO5 | 显示屏 DC | Data/Command 选择 |
| GPIO8 | I2S DOUT | 扬声器输出 |
| GPIO9 | I2S BCLK | 音频位时钟 |
| GPIO10 | I2S DIN | 麦克风输入 |
| GPIO11 | SPI CLK | 显示屏 SPI 时钟 |
| GPIO12 | SPI MOSI | 显示屏 SPI 数据 |
| GPIO13 | I2C SDA | I2C 数据线 (SHTC3, ES8311, ES7210) |
| GPIO14 | I2C SCL | I2C 时钟线 |
| GPIO16 | I2S MCLK | 音频主时钟 |
| **GPIO18** | **KEY 按键** | **低电平有效** |
| GPIO40 | 显示屏 CS | SPI 片选 |
| GPIO41 | 显示屏 RESET | 复位信号 |
| GPIO45 | I2S LRCLK | 音频左右声道时钟 |
| GPIO46 | 扬声器使能 | 功放使能信号 |

---

## 按键配置

### BOOT 按键 (GPIO0)

```yaml
binary_sensor:
  - platform: gpio
    id: btn_boot
    name: "Boot Button"
    pin:
      number: GPIO0
      inverted: true
      mode: INPUT_PULLUP
```

### KEY 按键 (GPIO18)

```yaml
binary_sensor:
  - platform: gpio
    id: btn_key
    name: "Key Button"
    pin:
      number: GPIO18
      inverted: true
      mode: INPUT_PULLUP
```

---

## 外设配置

### SPI (显示屏)

```yaml
spi:
  clk_pin: GPIO11
  mosi_pin: GPIO12
```

### I2C (传感器 + 音频芯片)

```yaml
i2c:
  sda: GPIO13
  scl: GPIO14
  scan: true
```

### 显示屏 (ST7305 RLCD)

```yaml
display:
  - platform: st7305_rlcd
    model: WAVESHARE_400X300
    cs_pin: GPIO40
    dc_pin: GPIO5
    reset_pin: GPIO41
```

### 温湿度传感器 (SHTC3)

```yaml
sensor:
  - platform: shtcx
    address: 0x70
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
```

### 电池电压 (ADC)

```yaml
sensor:
  - platform: adc
    pin: GPIO4
    attenuation: 12db
    filters:
      - multiply: 3.0  # 分压比 1/3
```

### RTC (PCF85063)

```yaml
time:
  - platform: pcf85063
    id: my_time
```

---

## PSRAM 配置

```yaml
psram:
  mode: octal
  speed: 80MHz
```

---

## 注意事项

1. **GPIO0** 是 strapping pin，使用时需小心
2. **GPIO18** 是 KEY 按键，需要 `inverted: true` + `INPUT_PULLUP`
3. 电池 ADC 分压比为 1/3，需要 `multiply: 3.0` 校准
4. 显示屏使用 SPI 接口，数据率建议 1MHz
5. I2C 总线共享给 SHTC3、ES8311(DAC)、ES7210(ADC)

---

## 参考来源

- [ESPHome 设备数据库](https://devices.esphome.io/devices/waveshare-esp32-s3-rlcd-42/)
- [Waveshare 官方文档](https://www.waveshare.com/esp32-s3-rlcd-4.2.htm)
- [Home Assistant 社区讨论](https://community.home-assistant.io/t/custom-component-for-waveshare-esp32-s3-4-2-rlcd-st7305/982089)
