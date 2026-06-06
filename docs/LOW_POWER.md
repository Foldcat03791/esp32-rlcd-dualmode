# 低功耗模式配置

## 概述

ESP32-S3 支持多种低功耗模式，配合 RLCD 反射式显示屏（静态显示不耗电），可实现超长待机时间。

---

## 功耗模式对比

| 模式 | 功耗 | 唤醒时间 | 状态保持 |
|------|------|----------|----------|
| Active | ~100mA | - | 全部运行 |
| DFS (降频) | ~50mA | 即时 | 全部运行 |
| Light-sleep | ~0.8mA | < 1ms | RAM 保持 |
| Deep-sleep | ~10μA | ~100ms | 仅 RTC |

---

## DFS 动态频率调节

DFS 根据应用需求动态调整 CPU 和 APB 频率。

### ESP-IDF 配置

```c
#include "esp_pm.h"

// 启用电源管理
esp_pm_configure(&(esp_pm_config_t){
    .max_freq_mhz = 240,
    .min_freq_mhz = 40,
    .light_sleep_enable = false,
});

// 获取高性能锁
esp_pm_lock_handle_t pm_lock;
esp_pm_lock_create(ESP_PM_CPU_FREQ_MAX, &pm_lock);
esp_pm_lock_acquire(pm_lock);  // 升急任务时锁定高频

// 释放锁，允许降频
esp_pm_lock_release(pm_lock);
```

### 配置选项

```ini
# menuconfig
CONFIG_PM_ENABLE=y
CONFIG_PM_DFS_INIT_AUTO=y
CONFIG_FREERTOS_HZ=1000
```

---

## Light-sleep 轻睡眠

Light-sleep 关闭大部分模块，但保持 RAM 内容。

### 自动 Light-sleep

```c
#include "esp_pm.h"

// 启用自动 Light-sleep
esp_pm_configure(&(esp_pm_config_t){
    .max_freq_mhz = 240,
    .min_freq_mhz = 40,
    .light_sleep_enable = true,  // 自动进入
});

// 配置唤醒源
esp_sleep_enable_timer_wakeup(30 * 1000000);  // 30秒后唤醒
esp_sleep_enable_gpio_wakeup();  // GPIO 唤醒
```

### 唤醒源

| 唤醒源 | 配置函数 |
|--------|----------|
| 定时器 | `esp_sleep_enable_timer_wakeup()` |
| GPIO | `esp_sleep_enable_gpio_wakeup()` |
| 触摸 | `esp_sleep_enable_touchpad_wakeup()` |
| UART | `esp_sleep_enable_uart_wakeup()` |
| WiFi | `esp_sleep_enable_wifi_wakeup()` |

---

## Deep-sleep 深度睡眠

Deep-sleep 仅保留 RTC 域，功耗最低。

### 基本配置

```c
#include "esp_sleep.h"

// 配置唤醒源
esp_sleep_enable_timer_wakeup(60 * 1000000);  // 60秒

// 配置 GPIO 唤醒 (KEY 按钮)
esp_sleep_enable_ext0_wakeup(GPIO_NUM_18, 0);  // 低电平唤醒

// 进入深度睡眠
esp_deep_sleep_start();
```

### 唤醒后处理

```c
void app_main(void)
{
    // 检查唤醒原因
    esp_sleep_wakeup_cause_t cause = esp_sleep_get_wakeup_cause();
    
    switch (cause) {
        case ESP_SLEEP_WAKEUP_TIMER:
            ESP_LOGI(TAG, "定时器唤醒");
            break;
        case ESP_SLEEP_WAKEUP_EXT0:
            ESP_LOGI(TAG, "GPIO 唤醒 (KEY 按钮)");
            break;
        case ESP_SLEEP_WAKEUP_UNDEFINED:
        default:
            ESP_LOGI(TAG, "正常启动");
            break;
    }
}
```

### RTC 内存

使用 RTC 内存保持数据跨深度睡眠：

```c
// RTC 内存 (深度睡眠后保持)
RTC_DATA_ATTR static int sleep_count = 0;

void app_main(void)
{
    sleep_count++;
    ESP_LOGI(TAG, "唤醒次数: %d", sleep_count);
    
    if (sleep_count >= 10) {
        sleep_count = 0;
        // 正常运行
    } else {
        // 继续睡眠
        esp_deep_sleep_start();
    }
}
```

---

## ULP 协处理器

ULP (Ultra Low Power) 协处理器可在深度睡眠期间运行简单程序。

### ULP 程序示例

```c
#include "ulp_main.h"

// ULP 程序：周期性读取 ADC
const ulp_insn_t program[] = {
    I_MOVI(R0, 0),          // R0 = 0
    I_ADC(R1, 0, 0),        // R1 = ADC(ADC1_CHANNEL_0)
    I_WAKE(),               // 唤醒主 CPU
    I_HALT(),               // 停止
};

// 加载 ULP 程序
ulp_load_binary(0, program, sizeof(program) / sizeof(ulp_insn_t));

// 启动 ULP
ulp_run(0);
```

---

## WiFi 低功耗

### Modem-sleep

WiFi 空闲时自动关闭 RF：

```c
// 启用 WiFi Modem-sleep
esp_wifi_set_ps(WIFI_PS_MIN_MODEM);  // 最小功耗
// 或
esp_wifi_set_ps(WIFI_PS_MAX_MODEM);  // 最大功耗，但延迟更高
```

### 连接策略

```c
// 快速连接 (跳过扫描)
esp_wifi_set_fast_connect(true);

// 设置重连间隔
esp_wifi_set_auto_reconnect(true);
```

---

## RLCD 低功耗优势

反射式 LCD 的独特优势：

| 特性 | RLCD | 传统 LCD |
|------|------|----------|
| 静态显示功耗 | **0μA** | 20-100mA |
| 背光 | 不需要 | 必须 |
| 阳光可读性 | 优秀 | 差 |
| 刷新功耗 | ~10mA | ~20mA |

### 低功耗显示策略

```c
// 1. 仅在内容变化时刷新
void display_update_if_needed()
{
    if (content_changed) {
        display_refresh();
        content_changed = false;
    }
}

// 2. 进入睡眠前刷新一次
void enter_sleep()
{
    display_refresh();  // 显示睡眠状态
    esp_deep_sleep_start();
}

// 3. 唤醒后更新显示
void on_wakeup()
{
    display_refresh();  // 更新时间/状态
}
```

---

## ESPHome 低功耗配置

### 深度睡眠配置

```yaml
deep_sleep:
  id: deep_sleep_mode
  sleep_duration: 5min
  wakeup_pin: GPIO18
  wakeup_pin_mode: INVERTED
```

### 条件睡眠

```yaml
on_...:
  - if:
      condition:
        - binary_sensor.is_off: charging
      then:
        - deep_sleep.enter: deep_sleep_mode
```

### 定期唤醒更新

```yaml
deep_sleep:
  id: deep_sleep_mode
  sleep_duration: 15min
  run_duration: 10s  # 唤醒后运行 10 秒更新数据
```

---

## 功耗测量

### 典型场景功耗

| 场景 | 功耗 | 说明 |
|------|------|------|
| 深度睡眠 | ~10μA | 仅 RTC 运行 |
| ULP 运行 | ~50μA | ULP + RTC |
| Light-sleep | ~0.8mA | RAM 保持 |
| WiFi 连接 | ~100mA | RF 激活 |
| 显示刷新 | ~15mA | RLCD 刷新 |
| 音频播放 | ~120mA | 扬声器输出 |

### 电池续航估算

```
假设：
- 电池容量：2000mAh
- 深度睡眠：10μA
- 每小时唤醒一次，运行 5 秒
- 运行功耗：100mA

计算：
- 睡眠功耗：10μA × 3595s = 35.95μAs
- 运行功耗：100mA × 5s = 500mAs
- 每小时功耗：535.95μAs ≈ 0.149mAh

续航时间：
2000mAh / 0.149mAh/h ≈ 13422 小时 ≈ 559 天
```

---

## 最佳实践

### 1. 分层功耗管理

```
┌─────────────────────────────────────┐
│  Active (WiFi/音频/显示刷新)        │  ~100mA
├─────────────────────────────────────┤
│  DFS (CPU 降频)                    │  ~50mA
├─────────────────────────────────────┤
│  Light-sleep (等待事件)            │  ~0.8mA
├─────────────────────────────────────┤
│  Deep-sleep (长时间待机)           │  ~10μA
└─────────────────────────────────────┘
```

### 2. 唤醒策略

- **定时唤醒**：定期更新传感器数据/时间显示
- **事件唤醒**：按键/触摸触发
- **ULP 唤醒**：ADC 阈值/传感器监测

### 3. 外设管理

```c
// 不使用时关闭外设
void disable_peripherals(void)
{
    // 关闭 WiFi
    esp_wifi_stop();
    
    // 关闭蓝牙
    esp_bt_controller_disable();
    
    // 关闭 I2S
    i2s_channel_disable(tx_handle);
    i2s_channel_disable(rx_handle);
    
    // 关闭功放
    gpio_set_level(GPIO_NUM_46, 0);
    
    // 关闭 SPI (显示屏)
    spi_bus_free(SPI2_HOST);
}
```

---

## 参考资源

- [ESP-IDF 低功耗指南](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-guides/low-power-mode/)
- [ESP32 低功耗设计](https://docs.espressif.com/projects/esp-idf/zh_CN/v5.1.3/esp32s3/api-guides/low-power-mode.html)
- [ESPHome 深度睡眠](https://esphome.io/components/deep_sleep.html)
- [ESP32 ePaper 低功耗设计](https://www.esp32s.com/blog/the-complete-guide-to-esp32-e-paper-displays/)
