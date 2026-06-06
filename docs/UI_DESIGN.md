# LVGL 界面设计指南

## 概述

LVGL (Light and Versatile Graphics Library) 是一个开源的嵌入式图形库，用于在资源受限的微控制器上创建图形用户界面 (GUI)。它提供了丰富的控件（如按钮、标签、滑块、图表等），使开发者能够高效地构建交互式图形界面。

---

## LVGL 架构

LVGL 通过硬件抽象层 (HAL) 将图形处理逻辑与具体硬件操作分离：

```
┌─────────────────────────────────────┐
│           Application               │
├─────────────────────────────────────┤
│            LVGL Core                │
│  (Widgets, Animations, Events)      │
├─────────────────────────────────────┤
│         Hardware Abstraction        │
│  ┌─────────────┬─────────────────┐  │
│  │ Display     │ Input Device    │  │
│  │ Driver      │ Driver          │  │
│  └─────────────┴─────────────────┘  │
├─────────────────────────────────────┤
│         Hardware (ESP32-S3)         │
│  ┌─────────────┬─────────────────┐  │
│  │ ST7305 RLCD │ Touch/Buttons   │  │
│  └─────────────┴─────────────────┘  │
└─────────────────────────────────────┘
```

---

## 三大核心接口

### 1. 显示驱动接口 (Display Driver)

用于将 LVGL 渲染的图像数据发送到显示设备。

```c
void my_disp_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p)
{
    // 将 color_p 中的像素数据发送到显示控制器
    // 通过 SPI 或其他总线协议传输
    
    // 通知 LVGL 刷新完成
    lv_disp_flush_ready(disp_drv);
}
```

### 2. 输入设备驱动接口 (Input Device Driver)

用于报告输入设备状态（触摸屏、物理按钮、编码器等）。

```c
void my_input_read(lv_indev_drv_t *drv, lv_indev_data_t *data)
{
    // 读取硬件状态并转换为 LVGL 格式
    data->point.x = touch_x;
    data->point.y = touch_y;
    data->state = LV_INDEV_STATE_PRESSED; // 或 LV_INDEV_STATE_RELEASED
}
```

### 3. 系统滴答接口 (System Tick)

LVGL 内部任务（动画、事件处理）依赖稳定的时间参考。

```c
uint32_t my_tick_get(void)
{
    return millis(); // 返回系统启动后的毫秒数
}
```

---

## ESP32-S3-RLCD-4.2 LVGL 配置

### 安装库

| 库名称 | 版本 | 说明 |
|--------|------|------|
| lvgl | v8.4.0 | LVGL 核心库 |
| GFX Library for Arduino | v1.4.9 | 显示驱动图形库 |

### lv_conf.h 配置

```c
/* 显示分辨率 */
#define LV_HOR_RES_MAX  (400)
#define LV_VER_RES_MAX  (300)

/* 颜色深度 */
#define LV_COLOR_DEPTH  1  /* 单色显示 */

/* 内存配置 */
#define LV_MEM_CUSTOM   1
#define LV_MEM_SIZE     (32 * 1024)  /* 32KB for LVGL */

/* 刷新率 */
#define LV_DISP_DEF_REFR_PERIOD  30  /* ms */
```

---

## RLCD 显示适配

ST7305 RLCD 是单色反射式 LCD，需要特殊适配：

### 颜色映射

```c
/* 单色显示：LVGL 颜色 -> RLCD 像素 */
static void set_pixel(int x, int y, bool color)
{
    if (color) {
        // 设置像素为黑色
        display_buffer[y * 400 + x] = 1;
    } else {
        // 设置像素为白色
        display_buffer[y * 400 + x] = 0;
    }
}
```

### 刷新回调

```c
void my_disp_flush(lv_disp_drv_t *disp, const lv_area_t *area, lv_color_t *color_p)
{
    int x1 = area->x1;
    int y1 = area->y1;
    int x2 = area->x2;
    int y2 = area->y2;
    
    for (int y = y1; y <= y2; y++) {
        for (int x = x1; x <= x2; x++) {
            bool pixel = color_p->full;
            set_pixel(x, y, pixel);
            color_p++;
        }
    }
    
    // 触发显示更新
    display_update_partial(x1, y1, x2, y2);
    
    lv_disp_flush_ready(disp);
}
```

---

## 常用控件示例

### 创建标签

```c
lv_obj_t *label = lv_label_create(lv_scr_act());
lv_label_set_text(label, "Hello World!");
lv_obj_align(label, LV_ALIGN_CENTER, 0, 0);
```

### 创建按钮

```c
lv_obj_t *btn = lv_btn_create(lv_scr_act());
lv_obj_set_size(btn, 120, 50);
lv_obj_align(btn, LV_ALIGN_CENTER, 0, 40);
lv_obj_add_event_cb(btn, btn_event_cb, LV_EVENT_CLICKED, NULL);

lv_obj_t *btn_label = lv_label_create(btn);
lv_label_set_text(btn_label, "Button");

void btn_event_cb(lv_event_t *e)
{
    // 按钮点击处理
}
```

### 创建图表

```c
lv_obj_t *chart = lv_chart_create(lv_scr_act());
lv_obj_set_size(chart, 200, 150);
lv_chart_set_type(chart, LV_CHART_TYPE_LINE);
lv_chart_set_point_count(chart, 10);
lv_chart_set_range(chart, LV_CHART_AXIS_PRIMARY_Y, 0, 100);

lv_chart_series_t *ser1 = lv_chart_add_series(chart, lv_palette_main(LV_PALETTE_RED), LV_CHART_AXIS_PRIMARY_Y);
```

---

## 按钮输入适配

将物理按钮映射到 LVGL 输入：

```c
void button_read(lv_indev_drv_t *drv, lv_indev_data_t *data)
{
    static uint32_t last_btn = 0;
    static bool btn_state = false;
    
    if (gpio_get_level(GPIO_NUM_0) == 0) {  // BOOT 按钮
        data->key = LV_KEY_UP;
        data->state = LV_INDEV_STATE_PRESSED;
    } else if (gpio_get_level(GPIO_NUM_18) == 0) {  // KEY 按钮
        data->key = LV_KEY_ENTER;
        data->state = LV_INDEV_STATE_PRESSED;
    } else {
        data->state = LV_INDEV_STATE_RELEASED;
    }
}
```

---

## 性能优化

### 1. 减少刷新区域

```c
/* 只刷新变化的区域 */
lv_obj_invalidate(obj);  // 标记对象需要刷新
```

### 2. 使用部分刷新

```c
/* RLCD 支持部分刷新，减少传输数据量 */
display_update_partial(x1, y1, x2, y2);
```

### 3. 降低刷新率

```c
/* 对于静态内容，降低刷新率 */
#define LV_DISP_DEF_REFR_PERIOD  100  /* 100ms */
```

---

## 参考资源

- [LVGL 官方文档](https://docs.lvgl.io/)
- [Waveshare LVGL 教程](https://docs.waveshare.com/ESP32-Arduino-Tutorials/LVGL)
- [ESPHome ST7305 组件](https://devices.esphome.io/devices/waveshare-esp32-s3-rlcd-42/)
