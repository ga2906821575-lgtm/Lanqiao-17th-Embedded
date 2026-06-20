# 第17届蓝桥杯嵌入式省赛真题 — 裸机版

> **作者**: 孙苏明  
> **平台**: STM32G431RB (Cortex-M4, 80MHz)  
> **架构**: 裸机 while(1) 超级循环 + 定时器中断  
> **竞赛**: 蓝桥杯嵌入式赛道 — 第17届省赛

---

## 项目概述

本项目为**第17届蓝桥杯嵌入式组省赛**参赛代码，基于 **STM32G431RB** 微控制器实现的一套**压力/流量监测与控制系统**。

软件架构为经典的裸机 `while(1)` 超级循环 + 定时器中断回调模式，适合竞赛快速开发。

---

## 硬件平台

- **MCU**: STM32G431RB (Cortex-M4, 80MHz, 128KB Flash, 32KB SRAM)
- **显示**: LCD 屏幕（3个界面：MAIN / OUTP / PARA）
- **输入**: 4个按键 (K1~K4)
- **输出**: 8路 LED 指示 + PWM 输出
- **传感器**: 压力传感器 (ADC) + 流量传感器 (TIM3 输入捕获)

---

## 功能特性

### 三级菜单界面

| 界面 | 显示内容 |
|------|----------|
| **MAIN** | 实时压力 P、流量 F、累计流量 Q、占空比 D、电压 V、工作模式 |
| **OUTP** | 目标占空比 TarD、目标压力 TarP |
| **PARA** | 流量阈值 FH/FL、压力阈值 PL、占空比阈值 DH |

### 工作模式
- **手动模式 (MAN)**: 通过 K3/K4 直接调节 PWM 占空比
- **自动模式 (AUTO)**: 根据目标压力 TarP 自动调节 PWM 占空比（Bang-Bang 控制）

### 告警与指示
- **LED1**: 频率超限告警 (>8000Hz)
- **LED2**: 流量低 + 占空比高告警（延时2秒触发）
- **LED3**: 流量高 + 压力低告警（延时2秒触发）
- **LED4**: 占空比变化闪烁指示

### 参数校准
- **K3 长按 (>2s)**: 校准电压零点（v_offset = r37_volt）
- **K4 长按 (>2s)**: 清零累计流量 Q

---

## 软件架构

```
main()
  ├── 外设初始化 (GPIO/ADC/TIM/LCD)
  └── while(1)
        ├── key_scan()      // 50ms 周期轮询按键
        ├── adc_proc()      // ADC 采集 + 压力计算
        ├── pwm_proc()      // PWM 占空比更新
        ├── lcd_show()      // 100ms 周期 LCD 刷新
        └── led_proc()      // LED 状态更新

中断回调:
  ├── TIM3 IC中断  → 频率捕获 → 计算流量 F
  ├── TIM6 周期中断 → 100ms → 累计流量 Q
  └── TIM7 周期中断 → 1s    → 占空比调节 + LED4指示
```

---

## 外设配置

| 外设 | 配置 | 用途 |
|------|------|------|
| ADC2 CH15 | 12位, 单次转换 | R37电位器电压采集 → 压力 P |
| TIM2 CH2 PWM | 1MHz, 10kHz, 初始5% | PWM输出控制占空比 |
| TIM3 CH1 IC | 1MHz, 上升沿捕获 | R39频率测量 → 流量 F |
| TIM6 | 100ms 周期中断 | 累计流量 Q |
| TIM7 | 1s 周期中断 | 占空比渐变调节 |
| TIM1/TIM4/TIM17 | 自由运行计数器 | LED延时触发、按键长按计时 |
| GPIO | PC8~PC15 + PD2锁存 | 8路LED控制 |
| GPIO | PB0/PB1/PB2/PA0 | 4路按键输入 |

---

## 目录结构

```
project/
└── Project/
    ├── BSP/          # 业务逻辑 (LCD/按键/LED/ADC/PWM)
    │   ├── fun.c/h   # 主业务函数
    │   ├── lcd.c/h   # LCD驱动
    │   └── fonts.h   # 字库数据
    ├── Core/         # STM32 核心驱动 (HAL库)
    │   ├── Src/      # main.c, gpio.c, adc.c, tim.c, stm32g4xx_it.c
    │   └── Inc/      # 头文件
    ├── Drivers/      # CMSIS + STM32G4xx HAL驱动
    └── MDK-ARM/      # Keil 工程文件
```

---

## 编译说明

使用 **Keil MDK** 打开 `Project/Project.uvprojx` 即可编译下载。

---

## 关联项目

- **FreeRTOS 改造版**: [Lanqiao-17th-Embedded-FreeRTOS](https://github.com/ga2906821575-lgtm/Lanqiao-17th-Embedded-FreeRTOS) — 同一项目的多任务重构版本，引入 FreeRTOS 互斥量/信号量，适合学习 RTOS

---

## 作者

孙苏明

## 竞赛信息

- **竞赛**: 蓝桥杯 (Lanqiao Cup) 嵌入式赛道
- **届次**: 第17届省赛
- **时间**: 2026年