# 音频配置 (ES8311 + ES7210)

## 概述

ESP32-S3-RLCD-4.2 配备完整的音频系统：
- **ES8311**：低功耗单声道音频编解码器 (DAC)
- **ES7210**：四通道音频 ADC，支持双麦克风阵列
- **双麦克风**：支持回声消除和噪声抑制

---

## 硬件配置

### I2S 引脚映射

| GPIO | 功能 | 说明 |
|------|------|------|
| GPIO8 | I2S DOUT | 扬声器输出 |
| GPIO9 | I2S BCLK | 位时钟 |
| GPIO10 | I2S DIN | 麦克风输入 |
| GPIO16 | I2S MCLK | 主时钟 |
| GPIO45 | I2S LRCLK | 左右声道时钟 |
| GPIO46 | Speaker EN | 功放使能 |

### I2C 设备地址

| 芯片 | I2C 地址 |
|------|----------|
| ES8311 (DAC) | 0x18 |
| ES7210 (ADC) | 0x40 |
| SHTC3 (传感器) | 0x70 |
| PCF85063 (RTC) | 0x51 |

---

## ES8311 DAC 配置

### ESPHome I2S 配置

```yaml
i2s_audio:
  - id: i2s_out
    i2s_lrclk_pin: GPIO45  # WS
    i2s_bclk_pin: GPIO9    # BCK
    i2s_mclk_pin: GPIO16   # MCLK
    i2s_mode: primary
    bits_per_sample: 16bit
    sample_rate: 16000
```

### 扬声器输出

```yaml
speaker:
  - platform: i2s_audio
    id: my_speaker
    i2s_audio_id: i2s_out
    i2s_dout_pin: GPIO8
    bits_per_sample: 16bit
    sample_rate: 16000
```

### 功放使能控制

```yaml
output:
  - platform: gpio
    id: speaker_enable
    pin: GPIO46

# 播放前使能功放
on_play:
  - output.turn_on: speaker_enable
  
on_stop:
  - output.turn_off: speaker_enable
```

---

## ES7210 ADC 配置

### 麦克风输入

```yaml
microphone:
  - platform: i2s_audio
    id: my_microphone
    i2s_audio_id: i2s_in
    i2s_din_pin: GPIO10
    bits_per_sample: 16bit
    sample_rate: 16000
    channel: left
```

### 双麦克风阵列

```yaml
i2s_audio:
  - id: i2s_in
    i2s_lrclk_pin: GPIO45
    i2s_bclk_pin: GPIO9
    i2s_mclk_pin: GPIO16
    i2s_mode: primary
    bits_per_sample: 16bit
    sample_rate: 16000
    # 双麦克风配置
    channel_format: stereo
```

---

## ESP-IDF 配置

### I2S 驱动初始化

```c
#include "driver/i2s_std.h"

i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_0, I2S_ROLE_MASTER);
chan_cfg.dma_desc_num = 6;
chan_cfg.dma_frame_num = 240;

i2s_std_config_t std_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(16000),
    .slot_cfg = I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = GPIO_NUM_16,
        .bclk = GPIO_NUM_9,
        .ws = GPIO_NUM_45,
        .dout = GPIO_NUM_8,
        .din = GPIO_NUM_10,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};

i2s_new_channel(&chan_cfg, &tx_handle, &rx_handle);
i2s_channel_init_std_mode(tx_handle, &std_cfg);
i2s_channel_init_std_mode(rx_handle, &std_cfg);
```

### ES8311 初始化

```c
#include "es8311.h"

es8311_handle_t es8311_handle = es8311_create(i2c_port, ES8311_DEF_ADDR);

// 配置音频参数
es8311_config(es8311_handle, 16000, I2S_BITS_PER_SAMPLE_16BIT, ES8311_MODE_MASTER);

// 设置音量 (0-100)
es8311_set_voice_volume(es8311_handle, 80);

// 使能 DAC
es8311_enable(es8311_handle);
```

---

## 音频处理

### 回声消除 (AEC)

双麦克风阵列配合 ES7210 可实现回声消除：

```c
#include "esp_aec.h"

// 创建 AEC 实例
aec_config_t aec_cfg = {
    .sample_rate = 16000,
    .frame_size = 512,
    .filter_length = 1024,
};

aec_handle_t aec = aec_create(&aec_cfg);

// 处理音频
int16_t near_end[512];  // 麦克风输入
int16_t far_end[512];   // 扬声器输出 (参考)
int16_t output[512];    // 回声消除后输出

aec_process(aec, near_end, far_end, output);
```

### 噪声抑制 (NS)

```c
#include "esp_ns.h"

ns_config_t ns_cfg = {
    .sample_rate = 16000,
    .mode = NS_MODE_SS,  // 单通道
};

ns_handle_t ns = ns_create(&ns_cfg);

int16_t input[512];
int16_t output[512];

ns_process(ns, input, output);
```

---

## 语音助手配置

### ESPHome Voice Assistant

```yaml
voice_assistant:
  id: va
  microphone: my_microphone
  speaker: my_speaker
  on_wake_word_detected:
    - logger.log: "唤醒词检测！"
  on_stt_end:
    - logger.log: 
        format: "识别结果: %s"
        args: [x]
  on_tts_end:
    - voice_assistant.play: x
```

### 唤醒词配置

```yaml
micro_wake_word:
  models:
    - okay_nabu
    - hey_jarvis
  on_wake_word_detected:
    - voice_assistant.start:
```

---

## 音频参数建议

### 语音识别场景

| 参数 | 建议值 |
|------|--------|
| 采样率 | 16kHz |
| 位深 | 16-bit |
| 声道 | 单声道 |
| 编码 | PCM |

### 音乐播放场景

| 参数 | 建议值 |
|------|--------|
| 采样率 | 44.1kHz 或 48kHz |
| 位深 | 16-bit 或 24-bit |
| 声道 | 立体声 |
| 编码 | PCM / MP3 |

---

## 音量控制

### ESPHome 配置

```yaml
number:
  - platform: template
    id: volume
    name: "音量"
    min_value: 0
    max_value: 100
    step: 1
    set_action:
      - speaker.set_volume:
          id: my_speaker
          volume: !lambda 'return x / 100.0;'
```

### 软件音量控制

```c
// 线性音量
float volume = 0.8;  // 80%
int16_t sample = raw_sample * volume;

// 对数音量 (更自然)
float db = -20.0 * (1.0 - volume);  // 0% -> -20dB, 100% -> 0dB
float gain = pow(10.0, db / 20.0);
int16_t sample = raw_sample * gain;
```

---

## 功耗优化

### 低功耗音频配置

```c
// 降低采样率
es8311_set_sample_rate(es8311_handle, 8000);  // 8kHz 足够语音

// 降低位深
es8311_set_bits_per_sample(es8311_handle, I2S_BITS_PER_SAMPLE_16BIT);

// 不使用时关闭
es8311_disable(es8311_handle);
i2s_channel_disable(tx_handle);
gpio_set_level(GPIO_NUM_46, 0);  // 关闭功放
```

---

## 参考资源

- [ES8311 数据手册](https://www.espressif.com/sites/default/files/documentation/ES8311%20Datasheet%20v1.0.pdf)
- [ES7210 数据手册](https://www.espressif.com/sites/default/files/documentation/ES7210%20Datasheet%20v1.0.pdf)
- [ESP-ADF 音频开发框架](https://docs.espressif.com/projects/esp-adf/)
- [ESP32-S3-Korvo-2 音频板](https://docs.espressif.com/projects/esp-adf/en/latest/design-guide/dev-boards/user-guide-esp32-s3-korvo-2-v3.0.html)
