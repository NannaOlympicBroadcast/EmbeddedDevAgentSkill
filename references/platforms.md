# 嵌入式平台参考速查表
## Table of Contents
1. [ESP32 系列 (Espressif)](#esp32)
2. [STM32 系列 (ST Microelectronics)](#stm32)
3. [Raspberry Pi 系列](#raspberry-pi)
4. [Nordic nRF 系列 (BLE/Thread)](#nordic-nrf)
5. [Zephyr RTOS 通用](#zephyr)
6. [FreeRTOS 通用](#freertos)
7. [Arduino 兼容板](#arduino)
8. [Radxa OS（Rockchip 系列）](#radxa-os)
9. [BeagleBone / AM335x](#beaglebone)

---

## ESP32 系列 {#esp32}

| 型号 | 架构 | 官方文档 |
|------|------|---------|
| ESP32 | Xtensa LX6 dual-core | https://docs.espressif.com/projects/esp-idf/en/latest/ |
| ESP32-S3 | Xtensa LX7 dual-core | https://docs.espressif.com/projects/esp-idf/en/latest/ |
| ESP32-C3 | RISC-V single-core | https://docs.espressif.com/projects/esp-idf/en/latest/ |
| ESP32-H2 | RISC-V + 802.15.4 | https://docs.espressif.com/projects/esp-idf/en/latest/ |

**工具链安装：**
```bash
# ESP-IDF 安装（推荐方式）
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf && ./install.sh esp32s3  # 替换为对应芯片
. ./export.sh

# 串口设备（Linux）
ls /dev/ttyUSB* /dev/ttyACM*
# 添加用户到 dialout 组
sudo usermod -aG dialout $USER
```

**常用命令：**
```bash
idf.py set-target esp32s3
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
idf.py monitor                    # 仅监控串口
idf.py menuconfig                 # 配置项目
esptool.py chip_id                # 读取芯片信息
```

**串口波特率：** 115200（默认）/ 921600（快速）

---

## STM32 系列 {#stm32}

| 系列 | 核心 | 参考文档 |
|------|------|---------|
| STM32F0/F1/F3 | Cortex-M0/M3/M4 | https://www.st.com/en/microcontrollers-microprocessors/ |
| STM32F4/F7 | Cortex-M4/M7 | https://wiki.st.com/stm32mcu/ |
| STM32H7 | Cortex-M7 (480MHz) | https://wiki.st.com/stm32mcu/ |
| STM32L4 (低功耗) | Cortex-M4 | https://wiki.st.com/stm32mcu/ |

**工具链：**
```bash
# ARM GCC 工具链
sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi

# OpenOCD (调试/烧录)
sudo apt install openocd

# STM32CubeProgrammer (GUI 烧录)
# 下载: https://www.st.com/en/development-tools/stm32cubeprog.html
```

**使用 OpenOCD 烧录：**
```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg \
  -c "program build/firmware.elf verify reset exit"
```

**使用 Zephyr + STM32：**
```bash
west build -b stm32f4_disco
west flash
west espressif monitor  # 或 minicom -D /dev/ttyACM0 -b 115200
```

---

## Raspberry Pi 系列 {#raspberry-pi}

| 型号 | 芯片 | 架构 |
|------|------|------|
| Pi 4 Model B | BCM2711 | ARM Cortex-A72 (64-bit) |
| Pi 5 | BCM2712 | ARM Cortex-A76 (64-bit) |
| Pi Zero 2 W | RP3A0 | ARM Cortex-A53 |
| Pi Pico | RP2040 | ARM Cortex-M0+ dual |

**官方文档：** https://www.raspberrypi.com/documentation/

**串口连接 (Pi 4)：**
```
Pi GPIO14 (TXD) → USB-TTL RX
Pi GPIO15 (RXD) → USB-TTL TX
Pi GND          → USB-TTL GND
波特率: 115200
```

**SSH 连接：**
```bash
ssh pi@raspberrypi.local  # mDNS
ssh pi@<IP地址>            # 直连
# 默认密码: raspberry（新版本已无默认密码，需预配置）
```

**常用调试命令：**
```bash
vcgencmd measure_temp          # CPU 温度
vcgencmd get_throttled         # 限速状态（0x0 = 正常）
gpio readall                   # 查看 GPIO 状态
i2cdetect -y 1                 # 扫描 I2C 设备
raspi-config                   # 系统配置
journalctl -f                  # 实时系统日志
dmesg | tail -50               # 内核日志
```

---

## Nordic nRF 系列 {#nordic-nrf}

| 型号 | 特性 | 文档 |
|------|------|------|
| nRF52840 | BLE 5.0 + 802.15.4 | https://docs.nordicsemi.com/ |
| nRF9160 | LTE-M/NB-IoT | https://docs.nordicsemi.com/ |
| nRF5340 | Dual-core BLE | https://docs.nordicsemi.com/ |

**nRF Connect SDK 工作流：**
```bash
# 安装 nRF Connect SDK (通过 nRF Connect for Desktop)
# https://www.nordicsemi.com/Products/Development-tools/nrf-connect-for-desktop

west build -b nrf52840dk/nrf52840 samples/bluetooth/peripheral_hr
west flash
# 串口监控
minicom -D /dev/ttyACM0 -b 115200
```

---

## Zephyr RTOS 通用 {#zephyr}

**官方文档：** https://docs.zephyrproject.org/latest/

**安装：**
```bash
pip install west
west init ~/zephyrproject
cd ~/zephyrproject && west update
west zephyr-export
pip install -r zephyr/scripts/requirements.txt
```

**基础工作流：**
```bash
west build -b <board> <app_path>    # 编译
west flash                           # 烧录
west build -t menuconfig             # 配置
west build -t boards                 # 查看支持的开发板

# 常用 Shell 命令（通过串口）
kernel threads        # 查看线程
kernel stacks         # 查看栈使用
device list           # 查看设备驱动
sensor get <dev>      # 读取传感器
```

**支持的开发板：** https://docs.zephyrproject.org/latest/boards/index.html

---

## FreeRTOS 通用 {#freertos}

**官方文档：** https://www.freertos.org/Documentation/

**调试常用：**
```c
// 任务信息
vTaskList(pcBuffer);          // 列出所有任务
uxTaskGetStackHighWaterMark() // 检查栈余量

// 运行时统计
vTaskGetRunTimeStats(buffer); // 需开启 configGENERATE_RUN_TIME_STATS
```

---

## Arduino 兼容板 {#arduino}

**文档：** https://docs.arduino.cc/

```bash
# Arduino CLI
arduino-cli compile --fqbn arduino:avr:uno sketch/
arduino-cli upload -p /dev/ttyACM0 --fqbn arduino:avr:uno sketch/
arduino-cli monitor -p /dev/ttyACM0 --config baudrate=115200
```

---

## Radxa OS（Rockchip 系列）{#radxa-os}

**官方文档：** https://docs.radxa.com/en/
**Radxa Wiki（旧版参考）：** https://wiki.radxa.com/

| 开发板 | SoC | 架构 | 文档 |
|--------|-----|------|------|
| ROCK 5B / 5B+ | RK3588 | ARM Cortex-A76×4 + A55×4 (64-bit) | https://docs.radxa.com/en/rock5/rock5b |
| ROCK 5C | RK3588S2 | ARM Cortex-A76×4 + A55×4 (64-bit) | https://docs.radxa.com/en/rock5/rock5c |
| ROCK 4D | RK3576 | ARM Cortex-A72×4 + A53×4 (64-bit) | https://docs.radxa.com/en/rock4/rock4d |
| ROCK 3C | RK3566 | ARM Cortex-A55×4 (64-bit) | https://docs.radxa.com/en/rock3/rock3c |
| ROCK 3B | RK3568 | ARM Cortex-A55×4 (64-bit) | https://docs.radxa.com/en/rock3/rock3b |
| Radxa Zero 3W | RK3566 | ARM Cortex-A55×4 (64-bit) | https://docs.radxa.com/en/zero/zero3 |
| Radxa E25 | RK3568 | ARM Cortex-A55×4 (64-bit) | https://docs.radxa.com/en/rock3/e25 |

### ⚠️ 串口波特率（与 RPi 不同！）

Rockchip 系列板子默认串口配置：
```
波特率：1500000（1.5M baud）— 不是 115200！
数据位：8  停止位：1  校验：None  流控：None
串口设备：/dev/ttyFIQ0（调试串口）
```

> 注意：CH340/PL2303 某些型号不支持 1.5M baud，推荐使用 **FT232RL** 或 **CP210x**（确认固件版本）。

**GPIO 40-pin 调试串口（通用）：**
```
Pin 8  → TX（接 USB-TTL RX）
Pin 10 → RX（接 USB-TTL TX）
Pin 6  → GND
```

### 默认账户

```
用户名：radxa    密码：radxa   （大多数 Radxa OS 镜像）
用户名：rock     密码：rock    （ROCK 系列部分镜像）
root 默认无密码，可通过 sudo passwd root 设置
```

### SSH 连接

```bash
# SSH 默认开启（port 22）
ssh radxa@<IP地址>
ssh radxa@radxa.local          # mDNS（需同网络）

# 查找板子 IP
ip a                           # 在串口终端执行
# 或在路由器 DHCP 页面查看
```

### rsetup — 系统配置工具（类似 raspi-config）

```bash
sudo rsetup                    # 进入 TUI 配置界面

# 主要功能路径：
# System → System Update       系统更新
# Overlays → Manage Overlays   启用/禁用设备树 overlay（I2C/SPI/UART/NPU 等）
# Overlays → View Overlay Info 查看已启用的 overlay
# Network → Wireless           WiFi 配置
# User Settings → Change Password
```

### 设备树 Overlay（DT Overlay）

```bash
# 通过 rsetup 启用 overlay（以 UART4-M1 为例）
sudo rsetup → Overlays → Manage Overlays → 选中 "Enable UART4-M1" → 重启

# 手动查看已加载 overlay
cat /boot/config.txt | grep dtoverlay

# 可用 overlay 列表
ls /boot/dtbo/

# overlay 生效后检查设备节点
ls /dev/ttyS*          # UART
ls /dev/i2c-*          # I2C
ls /dev/spidev*        # SPI
```

### NPU（神经网络加速）— RK3588 / RK356x

```bash
# RK3588 系列
sudo apt install rknpu2-rk3588

# RK356x 系列（RK3566/RK3568）
sudo apt install rknpu2-rk356x
# 需先通过 rsetup 启用 NPU overlay：
# sudo rsetup → Overlays → Manage Overlays → Enable NPU → 重启

# 检查 NPU 驱动版本
sudo dmesg | grep "Initialized rknpu"

# RKNN 模型转换（在 x86 PC 上）
pip install rknn-toolkit2
# 模型部署（在 Radxa 板上）
pip install rknn-toolkit-lite2
```

### 常用调试命令

```bash
# 系统信息
cat /proc/device-tree/model && echo    # 读取板子型号
cat /proc/cpuinfo | grep -E "model|Hardware|Revision"
uname -r                               # 内核版本

# 温度与频率
cat /sys/class/thermal/thermal_zone*/temp   # 各温度区域（÷1000 = °C）
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq

# GPIO
gpiodetect                             # 列出 GPIO chip
gpioinfo gpiochip0                     # 查看 GPIO 状态

# 外设
ls /dev/ttyS*                          # UART 设备
i2cdetect -y 0                         # 扫描 I2C-0 总线
ls /dev/spidev*                        # SPI 设备

# 网络
nmcli device status                    # 网络设备状态
nmcli device wifi list                 # 扫描 WiFi
sudo nmcli device wifi connect <SSID> password <PASSWD>

# APT 包管理（Radxa 专有源）
# /etc/apt/sources.list.d/apt-radxa-com.list
sudo apt update && sudo apt upgrade    # 常规更新
sudo rsetup → System → System Update   # 通过 rsetup 更新（推荐）

# 内核日志
dmesg | grep -E "rknpu|rga|iommu|mmc"
journalctl -b -k | tail -50
```

### 镜像烧录

```bash
# 工具：rkdeveloptool（Maskrom 模式）或 balenaEtcher（SD 卡）

# 官方镜像下载
# https://github.com/radxa-build/<board-name>/releases

# rkdeveloptool 烧录（Maskrom 模式）
rkdeveloptool ld                       # 列出连接设备
rkdeveloptool db <loader.bin>          # 下载 bootloader
rkdeveloptool wl 0 <image.img>         # 写入镜像
rkdeveloptool rd                       # 重启设备
```

---

## BeagleBone / AM335x {#beaglebone}

**文档：** https://docs.beagleboard.io/

```bash
# 默认串口: /dev/ttyUSB0, 115200 baud
# SSH: ssh debian@192.168.7.2 (USB网络)
# 默认密码: temppwd

config-pin P9.11 uart   # 配置引脚为 UART
gpioinfo                 # 查看 GPIO 信息
i2cdetect -y 2           # I2C 扫描
```
