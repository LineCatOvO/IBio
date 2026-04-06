# 通信模块技术规格文档

**文档版本**：v1.0
**创建时间**：2026-04-06
**作者**：IBio 项目团队
**状态**：初稿

---

## 一、概述

### 1.1 模块简介

IBio 通信模块采用 **USB + BLE 双模通信方案**，基于 ESP32-S3 内置 USB OTG 和 BLE 5.0 硬件，实现 FIDO2 标准要求的 CTAP over HID 和 CTAP over BLE 传输协议。

**核心架构**：
```
┌─────────────────────────────────────────────────────────────────┐
│                    通信模块架构                                   │
└─────────────────────────────────────────────────────────────────┘

           CTAP 协议栈（应用层）
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    传输协议模块                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │ USB HID 传输│  │ BLE GATT传输│  │ 消息分发    │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
├─────────────────────────────────────────────────────────────────┤
│                    硬件抽象层（HAL）                              │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ USB OTG驱动 │  │ BLE 5.0驱动 │                               │
│  └─────────────┘  └─────────────┘                               │
├─────────────────────────────────────────────────────────────────┤
│                    ESP32-S3 硬件                                 │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ USB OTG 1.1 │  │ BLE 5.0射频 │                               │
│  │ (内置)      │  │ (内置)      │                               │
│  └─────────────┘  └─────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 核心特性

| 特性 | 说明 |
|------|------|
| **双模支持** | USB HID + BLE GATT 双模，支持桌面和移动场景 |
| **内置硬件** | ESP32-S3 内置 USB OTG 和 BLE 5.0，无外接芯片 |
| **成本最优** | 通信模块成本仅 USB-C 连接器（$0.5-1） |
| **标准协议** | 符合 CTAP over HID 和 CTAP over BLE 规范 |
| **无缝切换** | USB/BLE 同时支持，无缝切换 |

### 1.3 适用场景

| 场景 | 通信模式 | 说明 |
|------|----------|------|
| **桌面认证** | USB HID | Windows/Mac/Linux 桌面浏览器 |
| **移动认证** | BLE GATT | iOS/Android 移动设备 |
| **便携场景** | BLE GATT | 无 USB 连接时的便携认证 |
| **双模并发** | USB + BLE | USB 和 BLE 同时可用 |

### 1.4 文档目的

本技术规格文档详细定义 IBio 通信模块的技术参数，用于指导：
- 硬件设计（USB-C 接口设计、BLE 天线设计）
- 固件开发（USB HID 驱动、BLE GATT 服务、传输协议实现）
- 测试验证（兼容性测试、性能测试）

---

## 二、USB HID 规格

### 2.1 USB 设备规格

| 项目 | 规格 | 说明 |
|------|------|------|
| **USB 版本** | USB 2.0 | Full Speed (12 Mbps) |
| **设备类型** | USB HID（Human Interface Device） | 设备类 0x03 |
| **设备子类** | 0x00 | 接口定义子类 |
| **设备协议** | 0x00 | 接口定义协议 |
| **端点类型** | Interrupt IN/OUT | 中断传输 |
| **端点数量** | 2 个（IN + OUT） | HID 标准配置 |
| **数据包大小** | 64 字节 | HID 报告大小 |
| **传输间隔** | 1 ms | HID 中断间隔 |
| **最大传输速率** | 64 KB/s | 64 bytes / 1 ms interval |
| **超时时间** | 30 秒 | CTAP 响应超时 |

### 2.2 USB 设备描述符

#### 设备描述符（Device Descriptor）

| 字段 | 值 | 说明 |
|------|-----|------|
| **bLength** | 18 | 描述符长度 |
| **bDescriptorType** | 1 | 设备描述符类型 |
| **bcdUSB** | 0x0200 | USB 2.0 |
| **bDeviceClass** | 0x00 | 接口定义类 |
| **bDeviceSubClass** | 0x00 | 接口定义子类 |
| **bDeviceProtocol** | 0x00 | 接口定义协议 |
| **bMaxPacketSize0** | 64 | 控制端点最大包大小 |
| **idVendor** | 0x???? | 厂商 ID（待 FIDO 注册） |
| **idProduct** | 0x???? | 产品 ID（待分配） |
| **bcdDevice** | 0x0100 | 设备版本 1.0 |
| **iManufacturer** | 1 | 厂商字符串索引 |
| **iProduct** | 2 | 产品字符串索引 |
| **iSerialNumber** | 3 | 序列号字符串索引（必须） |
| **bNumConfigurations** | 1 | 配置数量 |

**厂商 ID/产品 ID 说明**：
- 官方 FIDO 认证需向 FIDO Alliance 注册 VID
- 开发测试阶段可使用测试 VID（如 0xFFFF）
- SoloKeys 使用 VID: 0x0483

#### 配置描述符（Configuration Descriptor）

| 字段 | 值 | 说明 |
|------|-----|------|
| **bLength** | 9 | 描述符长度 |
| **bDescriptorType** | 2 | 配置描述符类型 |
| **wTotalLength** | 41 | 总描述符长度 |
| **bNumInterfaces** | 1 | 接口数量（仅 HID） |
| **bConfigurationValue** | 1 | 配置值 |
| **iConfiguration** | 0 | 配置字符串索引 |
| **bmAttributes** | 0x80 | Bus-powered, No Remote Wakeup |
| **bMaxPower** | 100 | 最大电流 100 mA (500 mA / 2) |

#### 接口描述符（Interface Descriptor）

| 字段 | 值 | 说明 |
|------|-----|------|
| **bLength** | 9 | 描述符长度 |
| **bDescriptorType** | 4 | 接口描述符类型 |
| **bInterfaceNumber** | 0 | 接口编号 |
| **bAlternateSetting** | 0 | 备用设置 |
| **bNumEndpoints** | 2 | 端点数量（IN + OUT） |
| **bInterfaceClass** | 0x03 | HID 设备类 |
| **bInterfaceSubClass** | 0x00 | 无子类 |
| **bInterfaceProtocol** | 0x00 | 无协议 |
| **iInterface** | 0 | 接口字符串索引 |

#### HID 描述符（HID Descriptor）

| 字段 | 值 | 说明 |
|------|-----|------|
| **bLength** | 9 | HID 描述符长度 |
| **bDescriptorType** | 0x21 | HID 描述符类型 |
| **bcdHID** | 0x0111 | HID 版本 1.11 |
| **bCountryCode** | 0 | 不支持国家代码 |
| **bNumDescriptors** | 1 | 报告描述符数量 |
| **bDescriptorType** | 0x22 | 报告描述符类型 |
| **wDescriptorLength** | 33 | 报告描述符长度 |

#### 端点描述符（Endpoint Descriptor）

**端点 IN（Endpoint IN）**：
| 字段 | 值 | 说明 |
|------|-----|------|
| **bLength** | 7 | 描述符长度 |
| **bDescriptorType** | 5 | 端点描述符类型 |
| **bEndpointAddress** | 0x81 | 端点地址（IN, 端点 1） |
| **bmAttributes** | 0x03 | Interrupt 端点 |
| **wMaxPacketSize** | 64 | 最大包大小 |
| **bInterval** | 1 | 中断间隔 1 ms |

**端点 OUT（Endpoint OUT）**：
| 字段 | 值 | 说明 |
|------|-----|------|
| **bLength** | 7 | 描述符长度 |
| **bDescriptorType** | 5 | 端点描述符类型 |
| **bEndpointAddress** | 0x01 | 端点地址（OUT, 端点 1） |
| **bmAttributes** | 0x03 | Interrupt 端点 |
| **wMaxPacketSize** | 64 | 最大包大小 |
| **bInterval** | 1 | 中断间隔 1 ms |

### 2.3 HID 报告描述符

**CTAP HID 报告描述符（参考 FIDO 规范）**：

```
Usage Page (0xF1D0)      // FIDO Alliance Usage Page
Usage (0x01)             // FIDO U2F Authenticator Device
Collection (Application) // Application Collection
    Usage (0x20)         // Raw In Data Report
    Report Size (8)      // Report Size: 8 bits (1 byte)
    Report Count (64)    // Report Count: 64 bytes
    Input (Data, Var, Abs) // Input Report (64 bytes)
    
    Usage (0x21)         // Raw Out Data Report
    Report Size (8)      // Report Size: 8 bits (1 byte)
    Report Count (64)    // Report Count: 64 bytes
    Output (Data, Var, Abs) // Output Report (64 bytes)
End Collection
```

**字节格式（C 语言）**：
```c
const uint8_t hid_report_descriptor[] = {
    0x06, 0xD0, 0xF1,    // Usage Page (0xF1D0) - FIDO Alliance
    0x09, 0x01,          // Usage (0x01) - U2F Authenticator Device
    0xA1, 0x01,          // Collection (Application)
    0x09, 0x20,          // Usage (0x20) - Raw In Data Report
    0x15, 0x00,          // Logical Minimum (0)
    0x26, 0xFF, 0x00,    // Logical Maximum (255)
    0x75, 0x08,          // Report Size (8 bits)
    0x95, 0x40,          // Report Count (64 bytes)
    0x81, 0x02,          // Input (Data, Var, Abs)
    0x09, 0x21,          // Usage (0x21) - Raw Out Data Report
    0x15, 0x00,          // Logical Minimum (0)
    0x26, 0xFF, 0x00,    // Logical Maximum (255)
    0x75, 0x08,          // Report Size (8 bits)
    0x95, 0x40,          // Report Count (64 bytes)
    0x91, 0x02,          // Output (Data, Var, Abs)
    0xC0                 // End Collection
};
```

### 2.4 ESP32-S3 USB OTG 规格

| 项目 | 规格 | 说明 |
|------|------|------|
| **USB OTG 版本** | USB OTG 1.1 | ESP32-S3 内置 |
| **支持模式** | Device, Host | 支持 Device 和 Host 模式 |
| **端点数量** | 6 IN + 6 OUT | 可配置端点 |
| **端点类型** | Control, Interrupt, Bulk, Isochronous | 全端点类型支持 |
| **FIFO 大小** | 可配置（推荐 64 字节） | HID 端点 FIFO |
| **驱动支持** | ESP-IDF + esp_tinyusb | 官方驱动支持 |
| **引脚定义** | USB_DP (GPIO20), USB_DN (GPIO19) | USB 数据引脚 |

**ESP32-S3 USB 引脚映射**：
| ESP32-S3 引脚 | 功能 | 说明 |
|---------------|------|------|
| **USB_DP (GPIO20)** | USB D+ | USB 数据正 |
| **USB_DN (GPIO19)** | USB D- | USB 数据负 |
| **VBUS** | 电源检测 | 检测 USB 连接状态 |

### 2.5 USB-C 连接器规格

| 项目 | 规格 | 说明 |
|------|------|------|
| **连接器类型** | USB-C (Type-C) | 可逆插拔设计 |
| **USB 版本** | USB 2.0 | Full Speed |
| **引脚定义** | D+, D-, VBUS, GND, CC1, CC2 | USB-C 标准引脚 |
| **电流能力** | 500 mA | USB 2.0 标准电流 |
| **接触电阻** | < 30 mΩ | 低接触电阻 |
| **插入寿命** | > 10000 次 | 高耐用性 |

**USB-C 引脚映射**：
| USB-C 引脚 | ESP32-S3 引脚 | 功能 |
|------------|---------------|------|
| **D+** | USB_DP (GPIO20) | USB 数据正 |
| **D-** | USB_DN (GPIO19) | USB 数据负 |
| **VBUS** | 5V 输入 | USB 电源 |
| **GND** | GND | 地 |
| **CC1** | GPIO25（可选） | 配置通道 1 |
| **CC2** | GPIO26（可选） | 配置通道 2 |

---

## 三、BLE GATT 规格

### 3.1 BLE 设备规格

| 项目 | 规格 | 说明 |
|------|------|------|
| **BLE 版本** | BLE 5.0 | ESP32-S3 内置 |
| **BLE 角色** | Peripheral（外设模式） | 作为外设设备 |
| **射频频率** | 2.4 GHz ISM 频段 | BLE 标准频段 |
| **发射功率** | -20 to +10 dBm | 可调发射功率 |
| **接收灵敏度** | -98 dBm | BLE 5.0 灵敏度 |
| **调制方式** | GFSK | BLE 标准调制 |
| **最大连接数** | 9 个 | ESP32-S3 支持 |
| **MTU 最大值** | 517 字节 | ESP32-S3 支持大 MTU |

### 3.2 BLE 连接参数

| 项目 | 推荐值 | 范围 | 说明 |
|------|--------|------|------|
| **最小连接间隔** | 15 ms | 7.5 ms - 4000 ms | 快速响应 |
| **最大连接间隔** | 30 ms | 7.5 ms - 4000 ms | 平衡功耗 |
| **从设备延迟** | 0 | 0 - 499 | 无延迟（快速响应） |
| **监督超时** | 500 ms | 100 ms - 32 s | 连接超时 |
| **MTU 协商** | 64 字节 | 23 - 517 | 推荐 64+ |

**连接参数优化**：
- **快速响应场景**：15-30 ms 连接间隔，确保 CTAP 响应快速
- **功耗优化场景**：30-100 ms 连接间隔，降低功耗（可选）
- **稳定性场景**：监督超时 500 ms，防止连接断开

### 3.3 FIDO GATT 服务定义

#### 服务 UUID 定义

| 服务/特征 | UUID | 说明 |
|-----------|------|------|
| **FIDO Service** | 0xFFFD (16-bit) 或 FIDO Service UUID (128-bit) | FIDO GATT 服务 |
| **FIDO Control Point** | 0xFFFDC1 (128-bit) | 发送 CTAP 命令 |
| **FIDO Status** | 0xFFFDC2 (128-bit) | 返回 CTAP 状态 |
| **FIDO Service Revision** | 0xFFFDC3 (128-bit) | 返回服务版本 |
| **FIDO Service Revision Bitfield** | 0xFFFDC4 (128-bit) | 版本协商 |

**UUID 详细定义**：
```
FIDO Service UUID (128-bit):
0000FFFD-0000-1000-8000-002AC5D8A7C2

FIDO Control Point UUID:
0000FFFDC1-0000-1000-8000-002AC5D8A7C2

FIDO Status UUID:
0000FFFDC2-0000-1000-8000-002AC5D8A7C2

FIDO Service Revision UUID:
0000FFFDC3-0000-1000-8000-002AC5D8A7C2

FIDO Service Revision Bitfield UUID:
0000FFFDC4-0000-1000-8000-002AC5D8A7C2
```

#### FIDO Control Point 特征值

| 项目 | 规格 | 说明 |
|------|------|------|
| **UUID** | 0xFFFDC1 | Control Point 特征值 |
| **属性** | Write, Write Without Response | 可写入 |
| **值长度** | 可变（MTU 限制） | CTAP 命令数据 |
| **用途** | 发送 CTAP 命令 | 客户端写入 CTAP 请求 |

#### FIDO Status 特征值

| 项目 | 规格 | 说明 |
|------|------|------|
| **UUID** | 0xFFFDC2 | Status 特征值 |
| **属性** | Read, Notify | 可读可通知 |
| **值长度** | 1 字节 | 状态值 |
| **用途** | 返回 CTAP 状态 | 认证器返回处理状态 |

**状态值定义**：
| 状态值 | 说明 |
|--------|------|
| **0x00** | 空闲（Idle） |
| **0x01** | 处理中（Processing） |
| **0x02** | 等待用户操作（Waiting for User Action） |
| **0x03** | 处理完成（Processing Complete） |
| **0xFF** | 错误（Error） |

#### FIDO Service Revision 特征值

| 项目 | 规格 | 说明 |
|------|------|------|
| **UUID** | 0xFFFDC3 | Service Revision 特征值 |
| **属性** | Read | 可读 |
| **值长度** | 可变 | 版本字符串 |
| **用途** | 返回服务版本 | 返回 FIDO 服务版本 |

**版本值定义**：
| 版本 | 值 | 说明 |
|------|-----|------|
| **U2F** | "U2F_V2" | CTAP1/U2F 版本 |
| **FIDO2** | "FIDO2_1" | CTAP2.1 版本 |

#### FIDO Service Revision Bitfield 特征值

| 项目 | 规格 | 说明 |
|------|------|------|
| **UUID** | 0xFFFDC4 | Service Revision Bitfield 特征值 |
| **属性** | Read, Write | 可读可写 |
| **值长度** | 1-2 字节 | 版本位域 |
| **用途** | 版本协商 | 协商支持的版本 |

**版本位域定义**：
| 位 | 说明 |
|----|------|
| **Bit 0** | 支持 U2F (CTAP1) |
| **Bit 1** | 支持 FIDO2 (CTAP2) |
| **Bit 2** | 支持 FIDO2.1 (CTAP2.1) |

### 3.4 BLE 广播数据

**广播包内容**：

| 项目 | 值 | 说明 |
|------|-----|------|
| **Flags** | 0x06 | LE General Discoverable, BR/EDR Not Supported |
| **Complete Local Name** | "IBio FIDO2" | 设备名称 |
| **Service UUIDs** | 0xFFFD | FIDO Service UUID |
| **Appearance** | 0x?? | 外观类别（可选） |
| **TX Power Level** | +0 dBm | 发射功率（可选） |

**广播包字节格式（C 语言）**：
```c
// 广播数据
static uint8_t adv_data[] = {
    0x02, 0x01, 0x06,              // Flags: LE General Discoverable
    0x0B, 0x09, 'I', 'B', 'i', 'o', ' ', 'F', 'I', 'D', 'O', '2', // Name
    0x03, 0x03, 0xFD, 0xFF,        // Service UUIDs: 0xFFFD
};

// 扫描响应数据
static uint8_t scan_rsp_data[] = {
    0x0B, 0x09, 'I', 'B', 'i', 'o', ' ', 'F', 'I', 'D', 'O', '2', // Name
};
```

### 3.5 ESP32-S3 BLE 规格

| 项目 | 规格 | 说明 |
|------|------|------|
| **BLE 版本** | BLE 5.0 | ESP32-S3 内置 |
| **控制器** | BLE Controller | ESP32-S3 内置 |
| **主机** | Bluedroid 或 NimBLE | ESP-IDF 支持 |
| **射频频率** | 2.4 GHz | ISM 频段 |
| **发射功率** | -20 to +10 dBm | 可调 |
| **接收灵敏度** | -98 dBm @ BLE 5.0 | 高灵敏度 |
| **调制方式** | GFSK | BLE 标准调制 |
| **连接数量** | 最多 9 个 | 多连接支持 |
| **MTU 最大值** | 517 字节 | ESP32-S3 特性 |
| **天线** | PCB 天线或外接天线 | 内置或外接 |

**BLE 功耗规格**：
| 状态 | 功耗 | 说明 |
|------|------|------|
| **广播状态** | ~20 mA | 广播发射功耗 |
| **连接维持** | ~15 mA | 连接维持功耗 |
| **数据传输** | ~30 mA | 数据传输功耗 |
| **睡眠状态** | < 1 mA | BLE 睡眠功耗 |

---

## 四、CTAP 传输协议规格

### 4.1 CTAP over HID 协议

#### HID 报告格式

**初始化包（Initialization Packet）**：
| 字节偏移 | 字段 | 长度 | 说明 |
|----------|------|------|------|
| **0-3** | CID | 4 bytes | Channel Identifier |
| **4** | CMD | 1 byte | Command（最高位为 1） |
| **5** | BN | 1 byte | 包序号（0x00） |
| **6-7** | DLCN | 2 bytes | 数据长度（高字节在前） |
| **8-63** | Data | 56 bytes | 数据（最多 56 bytes） |

**继续包（Continuation Packet）**：
| 字节偏移 | 字段 | 长度 | 说明 |
|----------|------|------|------|
| **0-3** | CID | 4 bytes | Channel Identifier |
| **4** | SEQ | 1 byte | 序号（0x00-0xFF） |
| **5-63** | Data | 59 bytes | 数据（最多 59 bytes） |

#### HID 命令定义

| 命令 | 值 | 说明 |
|------|-----|------|
| **INIT** | 0x80 | 初始化通道 |
| **PING** | 0x81 | Ping 测试 |
| **MSG** | 0x83 | 发送 CTAP 消息 |
| **CANCEL** | 0x85 | 取消操作 |
| **ERROR** | 0x86 | 错误响应 |
| **KEEPALIVE** | 0x87 | 保活响应 |
| **WINK** | 0x88 | Wink（视觉反馈） |
| **LOCK** | 0x84 | 锁定通道 |

#### HID 错误码定义

| 错误码 | 值 | 说明 |
|--------|-----|------|
| **ERR_INVALID_CMD** | 0x01 | 无效命令 |
| **ERR_INVALID_PAR** | 0x02 | 无效参数 |
| **ERR_INVALID_LEN** | 0x03 | 无效长度 |
| **ERR_INVALID_SEQ** | 0x04 | 无效序号 |
| **ERR_MSG_TIMEOUT** | 0x05 | 消息超时 |
| **ERR_CHANNEL_BUSY** | 0x06 | 通道繁忙 |
| **ERR_LOCK_REQUIRED** | 0x07 | 需要锁定 |
| **ERR_INVALID_CID** | 0x08 | 无效通道 ID |
| **ERR_OTHER** | 0x09 | 其他错误 |

#### HID 消息分片示例

**发送 100 字节 CTAP 消息**：
```
初始化包（64 字节）：
[CID][CMD=0x83][BN=0x00][DLCN=100][Data=56 bytes]

继续包 1（64 字节）：
[CID][SEQ=0x00][Data=59 bytes]

继续包 2（64 字节）：
[CID][SEQ=0x01][Data=剩余 bytes]

总数据：56 + 59 + X = 100 bytes
```

#### HID 超时处理

| 超时类型 | 时间 | 说明 |
|----------|------|------|
| **响应超时** | 30 秒 | CTAP 响应超时 |
| **初始化超时** | 100 ms | 初始化包等待超时 |
| **继续包超时** | 100 ms | 继续包等待超时 |
| **KEEPALIVE 间隔** | 100 ms | 保活响应间隔 |

### 4.2 CTAP over BLE 协议

#### BLE 消息格式

**Control Point 写入格式**：
| 字节偏移 | 字段 | 长度 | 说明 |
|----------|------|------|------|
| **0** | CMD | 1 byte | CTAP 命令字节 |
| **1-N** | Data | N bytes | CTAP 命令数据（CBOR） |

**Status 通知格式**：
| 字节偏移 | 字段 | 镰度 | 说明 |
|----------|------|------|------|
| **0** | Status | 1 byte | 状态值（0x00-0xFF） |

#### BLE 分片规则

由于 BLE GATT MTU 限制（最小 20 字节），长消息需要分片：

| MTU 大小 | 最大单次传输 | 说明 |
|----------|--------------|------|
| **23 字节** | 20 字节 | BLE 4.2 默认 MTU |
| **64 字节** | 61 字节 | 推荐 MTU |
| **517 字节** | 514 字节 | ESP32-S3 最大 MTU |

**分片实现方式**：
- 客户端负责分片发送（写入 Control Point）
- 认证器负责重组消息
- 使用 Status 特征值通知处理状态

#### BLE 超时处理

| 超时类型 | 时间 | 说明 |
|----------|------|------|
| **响应超时** | 30 秒 | CTAP 响应超时 |
| **连接超时** | 500 ms | BLE 连接监督超时 |
| **MTU 协商超时** | 30 秒 | MTU 交换超时 |

### 4.3 传输协议实现要求

| 要求 | 说明 |
|------|------|
| **双模统一消息处理** | USB/BLE 接收的消息统一处理 |
| **通道隔离** | 不同传输通道的消息隔离处理 |
| **超时管理** | 所有操作都有超时处理 |
| **错误返回** | 错误情况返回标准错误码 |
| **并发支持** | 支持 USB/BLE 同时连接 |

---

## 五、接口规格

### 5.1 USB 接口规格

#### ESP32-S3 USB 引脚规格

| 引脚 | 功能 | 电平 | 说明 |
|------|------|------|------|
| **USB_DP (GPIO20)** | USB D+ | 3.3V | USB 数据正信号 |
| **USB_DN (GPIO19)** | USB D- | 3.3V | USB 数据负信号 |
| **VBUS** | 电源检测 | 5V | USB VBUS 检测 |

#### USB-C 连接器引脚规格

| USB-C 引脚 | 信号 | 电平 | 说明 |
|------------|------|------|------|
| **A5/B5 (CC1/CC2)** | 配置通道 | 5V | USB-C 配置检测 |
| **A6/B6 (D+)** | USB D+ | 3.3V | USB 数据正 |
| **A7/B7 (D-)** | USB D- | 3.3V | USB 数据负 |
| **A4/B4, A9/B9 (VBUS)** | 电源 | 5V | USB 电源 |
| **A1/B1, A12/B12 (GND)** | 地 | 0V | USB 地 |

#### USB 信号电平规格

| 信号 | 电平范围 | 说明 |
|------|----------|------|
| **USB D+/D- 高电平** | 2.8V - 3.6V | USB 高电平 |
| **USB D+/D- 低电平** | 0V - 0.3V | USB 低电平 |
| **USB VBUS** | 4.4V - 5.25V | USB 电源范围 |
| **USB termination** | 1.5 kΩ | USB 端接电阻 |

### 5.2 BLE 接口规格

#### ESP32-S3 BLE 射频规格

| 项目 | 规格 | 说明 |
|------|------|------|
| **射频频率** | 2400 - 2483.5 MHz | BLE ISM 频段 |
| **信道数量** | 40 个 | BLE 5.0 信道 |
| **调制方式** | GFSK | BLE 标准调制 |
| **发射功率** | -20 to +10 dBm | 可调发射功率 |
| **接收灵敏度** | -98 dBm @ BLE 5.0 | 接收灵敏度 |
| **天线阻抗** | 50 Ω | 标准阻抗 |

#### BLE 天线规格

**PCB 天线规格（推荐）**：
| 项目 | 规格 | 说明 |
|------|------|------|
| **天线类型** | PCB 倒 F 天线 | ESP32-S3 模块内置 |
| **频率范围** | 2400 - 2500 MHz | BLE 频段 |
| **增益** | 0 - 2 dBi | PCB 天线增益 |
| **阻抗** | 50 Ω | 标准阻抗 |
| **效率** | > 50% | 天线效率 |

**外接天线规格（可选）**：
| 项目 | 规格 | 说明 |
|------|------|------|
| **天线类型** | 外接陶瓷天线 | 更好的天线性能 |
| **增益** | 2 - 5 dBi | 外接天线增益 |
| **接口** | U.FL 或 SMA | 天线接口 |
| **阻抗** | 50 Ω | 标准阻抗 |

---

## 六、功耗规格

### 6.1 USB 通信功耗

| 状态 | 功耗 | 说明 |
|------|------|------|
| **USB 连接维持** | ~50 mA | USB 连接维持功耗 |
| **USB 数据传输** | ~60 mA | USB 数据传输功耗 |
| **USB 待机** | ~10 mA | USB 待机功耗 |
| **USB 断开** | ~1 mA | USB 断开功耗 |

### 6.2 BLE 通信功耗

| 状态 | 功耗 | 说明 |
|------|------|------|
| **BLE 广播** | ~20 mA | BLE 广播发射功耗 |
| **BLE 连接维持** | ~15 mA | BLE 连接维持功耗 |
| **BLE 数据传输** | ~30 mA | BLE 数据传输功耗 |
| **BLE 睡眠** | ~1 mA | BLE 睡眠功耗 |

### 6.3 双模并发功耗

| 状态 | 功耗 | 说明 |
|------|------|------|
| **USB + BLE 连接** | ~80 mA | 双模同时连接功耗 |
| **USB + BLE 传输** | ~90 mA | 双模同时传输功耗 |
| **USB + BLE 待机** | ~20 mA | 双模待机功耗 |

### 6.4 总功耗估算

| 场景 | 功耗 | 说明 |
|------|------|------|
| **USB 认证（含指纹）** | ~80 mA | USB + 指纹采集 |
| **BLE 认证（含指纹）** | ~50 mA | BLE + 指纹采集 |
| **USB + BLE 认证** | ~100 mA | 双模 + 指纹采集 |
| **待机状态** | < 10 mA | USB 连接维持 |

---

## 七、通信协议实现方案

### 7.1 USB HID 实现方案

#### ESP-IDF USB HID 驱动架构

| 步骤 | 实现内容 | ESP-IDF API |
|------|----------|-------------|
| **1. USB 初始化** | 初始化 USB Device 控制器 | `esp_tinyusb_init()` |
| **2. HID 描述符配置** | 配置 HID 设备描述符 | `tusb_hid_descriptor` |
| **3. HID 端点配置** | 配置 Interrupt IN/OUT 端点 | `usbd_edpt_open()` |
| **4. HID 数据收发** | 实现 HID 数据收发回调 | `tud_hid_get_report_cb()`, `tud_hid_set_report_cb()` |
| **5. CTAP 消息处理** | 实现 CTAP over HID 协议 | 自定义实现 |

#### ESP-IDF USB HID 示例代码结构

```c
// USB HID 初始化
esp_err_t usb_hid_init(void) {
    // 初始化 ESP-IDF USB Device
    esp_tinyusb_config_t tusb_cfg = {
        .external_phy = false,  // 使用内置 USB PHY
    };
    esp_tinyusb_init(&tusb_cfg);
    
    // 配置 HID 描述符
    tusb_desc_device_t desc_device = {
        .bLength = 18,
        .bDescriptorType = 1,
        .bcdUSB = 0x0200,
        .bDeviceClass = 0x00,
        // ... 其他字段
    };
    
    // 注册 HID 回调
    tud_hid_get_report_cb = usb_hid_get_report;
    tud_hid_set_report_cb = usb_hid_set_report;
    
    return ESP_OK;
}

// HID 数据接收回调
void usb_hid_set_report(uint8_t instance, uint8_t report_id, 
                        uint8_t report_type, uint8_t const* buffer, 
                        uint16_t bufsize) {
    // 处理 HID 报告（CTAP 消息）
    ctap_hid_process_message(buffer, bufsize);
}

// HID 数据发送回调
uint16_t usb_hid_get_report(uint8_t instance, uint8_t report_id, 
                            uint8_t report_type, uint8_t* buffer, 
                            uint16_t reqlen) {
    // 返回 HID 报告（CTAP 响应）
    return ctap_hid_get_response(buffer, reqlen);
}
```

#### CTAP over HID 实现要点

| 要点 | 说明 |
|------|------|
| **CID 管理** | 为每个连接分配唯一 Channel ID |
| **消息分片** | 实现 HID 报告分片和重组 |
| **超时管理** | 实现 30 秒响应超时 |
| **错误处理** | 返回标准 HID 错误码 |
| **并发支持** | 支持多通道同时处理 |

### 7.2 BLE GATT 实现方案

#### ESP-IDF BLE GATT 驱动架构

| 步骤 | 实现内容 | ESP-IDF API |
|------|----------|-------------|
| **1. BLE 初始化** | 初始化 BLE 控制器和主机 | `esp_bluedroid_init()` |
| **2. GATT 服务注册** | 注册 FIDO GATT 服务 | `esp_ble_gatts_create_service()` |
| **3. 特征值添加** | 添加 Control Point/Status 特征值 | `esp_ble_gatts_add_char()` |
| **4. BLE 广播配置** | 配置 BLE 广播数据 | `esp_ble_gap_config_adv_data()` |
| **5. BLE 连接管理** | 实现 BLE 连接和参数更新 | `esp_ble_gap_update_conn_params()` |
| **6. CTAP 消息处理** | 实现 CTAP over BLE 协议 | 自定义实现 |

#### ESP-IDF BLE GATT 示例代码结构

```c
// BLE GATT 初始化
esp_err_t ble_gatt_init(void) {
    // 初始化 BLE 控制器
    esp_bt_controller_config_t bt_cfg = BT_CONTROLLER_INIT_CONFIG_DEFAULT();
    esp_bt_controller_init(&bt_cfg);
    esp_bt_controller_enable(ESP_BT_MODE_BLE);
    
    // 初始化 Bluedroid 主机
    esp_bluedroid_init();
    esp_bluedroid_enable();
    
    // 注册 GATT 服务
    esp_ble_gatts_register_callback(gatts_event_handler);
    esp_ble_gap_register_callback(gap_event_handler);
    esp_ble_gatts_app_register(0);
    
    return ESP_OK;
}

// GATT 服务创建回调
void gatts_event_handler(esp_gatts_cb_event_t event, 
                         esp_gatt_if_t gatts_if, 
                         esp_ble_gatts_cb_param_t *param) {
    switch (event) {
        case ESP_GATTS_REG_EVT:
            // 创建 FIDO Service
            esp_ble_gatts_create_service(gatts_if, 
                &service_uuid, 0xFFFD, 4, true);
            break;
        case ESP_GATTS_CREATE_EVT:
            // 添加特征值
            esp_ble_gatts_add_char(param->create.service_handle, 
                &char_uuid, ESP_GATT_PERM_WRITE, 
                ESP_GATT_CHAR_PROP_BIT_WRITE_NR, NULL, 0);
            break;
        case ESP_GATTS_WRITE_EVT:
            // 处理 Control Point 写入
            ctap_ble_process_message(param->write.value, 
                param->write.len);
            break;
    }
}

// BLE 广播配置
void ble_adv_config(void) {
    esp_ble_adv_data_t adv_data = {
        .set_scan_rsp = false,
        .include_name = true,
        .include_txpower = false,
        .min_interval = 0x0006, // 7.5 ms
        .max_interval = 0x0010, // 15 ms
        .appearance = 0x00,
        .manufacturer_len = 0,
        .p_manufacturer_data = NULL,
        .service_data_len = 0,
        .p_service_data = NULL,
        .service_uuid_len = 2,
        .p_service_uuid = (uint8_t*)&fido_service_uuid,
        .flag = (ESP_BLE_ADV_FLAG_GEN_DISC | ESP_BLE_ADV_FLAG_BREDR_NOT_SPT),
    };
    esp_ble_gap_config_adv_data(&adv_data);
}
```

#### CTAP over BLE 实现要点

| 要点 | 说明 |
|------|------|
| **FIDO Service UUID** | 使用标准 FIDO Service UUID (0xFFFD) |
| **特征值定义** | 正确定义 Control Point/Status/Revision |
| **MTU 协商** | 支持 MTU 协商（推荐 64+） |
| **连接参数优化** | 优化连接间隔（15-30 ms） |
| **消息分片** | 支持 BLE 消息分片和重组 |
| **状态通知** | 使用 Status Notify 返回状态 |

### 7.3 双模切换方案

#### 双模统一消息处理

```
┌─────────────────────────────────────────────────────────────────┐
│                    双模消息处理架构                               │
└─────────────────────────────────────────────────────────────────┘

           USB HID 接收          BLE GATT 接收
                │                    │
                ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    消息分发模块                                   │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ USB 消息队列│  │ BLE 消息队列│                               │
│  └─────────────┘  └─────────────┘                               │
│                │                    │                            │
│                └────────────┬───────┘                            │
│                             ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │           CTAP 协议处理（统一处理）                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                             │                                    │
│                ┌────────────┴───────┐                            │
│                ▼                    ▼                            │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ USB 响应发送│  │ BLE 响应发送│                               │
│  └─────────────┘  └─────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
```

#### 双模优先级策略

| 场景 | 优先级 | 说明 |
|------|--------|------|
| **USB + BLE 同时连接** | USB > BLE | USB 为主要通道 |
| **USB 断开，BLE 连接** | BLE | BLE 作为备用通道 |
| **仅 USB 连接** | USB | USB 单模工作 |
| **仅 BLE 连接** | BLE | BLE 单模工作 |

---

## 八、电气特性

### 8.1 USB 电气特性

| 项目 | 规格 | 说明 |
|------|------|------|
| **VBUS 电压范围** | 4.4V - 5.25V | USB 2.0 标准 |
| **VBUS 电流能力** | 500 mA | USB 2.0 标准 |
| **D+/D- 电平（高）** | 2.8V - 3.6V | USB 数据高电平 |
| **D+/D- 电平（低）** | 0V - 0.3V | USB 数据低电平 |
| **D+/D- 端接电阻** | 1.5 kΩ | USB 端接 |
| **USB PHY 电源** | 3.3V | ESP32-S3 USB PHY |

### 8.2 BLE 电气特性

| 项目 | 规格 | 说明 |
|------|------|------|
| **射频频率** | 2400 - 2483.5 MHz | BLE ISM 频段 |
| **发射功率范围** | -20 to +10 dBm | ESP32-S3 可调 |
| **推荐发射功率** | 0 dBm | 平衡功耗和距离 |
| **接收灵敏度** | -98 dBm | BLE 5.0 灵敏度 |
| **调制方式** | GFSK | BLE 标准调制 |
| **天线阻抗** | 50 Ω | 标准阻抗 |
| **BLE PHY 电源** | 3.3V | ESP32-S3 BLE PHY |

### 8.3 电气参数总表

| 参数 | USB | BLE | 说明 |
|------|-----|-----|------|
| **工作电压** | 5V (VBUS) | 3.3V | USB 供电，BLE 3.3V |
| **工作电流（最大）** | 100 mA | 30 mA | USB/BLE 最大电流 |
| **功耗（待机）** | 10 mA | 15 mA | 待机功耗 |
| **功耗（传输）** | 60 mA | 30 mA | 传输功耗 |
| **射频功率** | - | 0 dBm | BLE 射频功率 |

---

## 九、应用建议

### 9.1 USB 配置建议

| 建议 | 说明 |
|------|------|
| **使用标准 VID/PID** | 开发阶段使用测试 VID（如 0xFFFF），正式发布需 FIDO 注册 |
| **配置 HID 描述符** | 使用标准 FIDO HID Usage Page (0xF1D0) |
| **优化端点配置** | 使用 64 字节 HID 报告，1 ms 中断间隔 |
| **实现 Wink 功能** | 提供 LED 闪烁或声音反馈，便于用户识别设备 |
| **多平台测试** | Windows/Mac/Linux 全面测试兼容性 |

### 9.2 BLE 配置建议

| 建议 | 说明 |
|------|------|
| **使用标准 FIDO Service UUID** | 使用 0xFFFD (16-bit) 或 128-bit UUID |
| **优化连接参数** | 使用 15-30 ms 连接间隔，快速响应 |
| **协商大 MTU** | 请求 64+ MTU，减少分片开销 |
| **配置广播数据** | 包含设备名称和 FIDO Service UUID |
| **多设备测试** | iOS/Android 全面测试兼容性 |

### 9.3 CTAP 协议实现建议

| 建议 | 说明 |
|------|------|
| **参考 SoloKeys/OpenSK** | 参考成熟项目的传输协议实现 |
| **统一消息处理** | USB/BLE 消息统一处理，简化逻辑 |
| **完善错误处理** | 返回标准 CTAP 错误码，便于调试 |
| **实现超时管理** | 所有操作都有超时处理，防止阻塞 |
| **添加调试日志** | 添加传输协议调试日志，便于问题排查 |

### 9.4 双模切换建议

| 建议 | 说明 |
|------|------|
| **USB 优先策略** | USB 连接时优先使用 USB，BLE 作为备用 |
| **状态同步** | USB/BLE 状态同步，无缝切换 |
| **功耗优化** | USB 连接时降低 BLE 功耗，BLE 连接时降低 USB 功耗 |
| **并发测试** | 测试 USB/BLE 同时连接和操作 |

### 9.5 功耗优化建议

| 建议 | 说明 |
|------|------|
| **USB 待机降低功耗** | USB 待机时使用 Light Sleep 模式 |
| **BLE 连接参数优化** | 增大连接间隔（30-100 ms），降低功耗 |
| **BLE 广播优化** | 降低广播频率，减少广播功耗 |
| **动态功耗调整** | 根据使用场景动态调整功耗策略 |

---

## 十、参考资料

### 10.1 FIDO 标准文档

| 文档 | 链接 |
|------|------|
| **CTAP v2.0** | https://fidoalliance.org/specs/fido-v2.0-ps-20190130/fido-client-to-authenticator-protocol-v2.0-ps-20190130.html |
| **CTAP v2.1** | https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-errata-20220621.html |
| **FIDO U2F HID** | https://fidoalliance.org/specs/fido-u2f-v1.2-ps-20170411/fido-u2f-hid-protocol-v1.2-ps-20170411.html |
| **FIDO GATT Service** | https://fidoalliance.org/specs/fido-v2.0-ps-20190130/fido-client-to-authenticator-protocol-v2.0-ps-20190130.html |

### 10.2 ESP-IDF 文档

| 文档 | 链接 |
|------|------|
| **ESP32-S3 USB Device** | https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/peripherals/usb_device.html |
| **ESP32-S3 BLE** | https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/bluetooth/index.html |
| **ESP-IDF USB HID 示例** | https://github.com/espressif/esp-idf/tree/master/examples/peripherals/usb/device/tusb_hid |
| **ESP-IDF BLE GATT 示例** | https://github.com/espressif/esp-idf/tree/master/examples/bluetooth/bluedroid/ble/gatt_server |

### 10.3 开源项目参考

| 项目 | 链接 | 参考内容 |
|------|------|----------|
| **SoloKeys** | https://github.com/solokeys/solo | CTAP over HID 实现 |
| **OpenSK** | https://github.com/google/OpenSK | CTAP over BLE 实现 |
| **libfido2** | https://github.com/Yubico/libfido2 | CTAP 测试和验证 |
| **TinyUSB** | https://github.com/hathach/tinyusb | USB HID 库 |

### 10.4 相关决策文档

| 文档 | 说明 |
|------|------|
| **ADR-005** | 通信模块选型决策 |
| **ADR-002** | 主控芯片选型决策 |
| **HLD.md** | IBio 系统架构设计 |

---

## 十一、结论

IBio 通信模块技术规格定义完成，核心要点：

1. **USB HID 规格**：USB 2.0 Full Speed，64 字节 HID 报告，CTAP over HID 传输
2. **BLE GATT 规格**：BLE 5.0，FIDO Service UUID 0xFFFD，CTAP over BLE 传输
3. **传输协议**：CTAP over HID/BLE，分片重组，30 秒超时
4. **接口规格**：USB-C 连接器，PCB BLE 天线，ESP32-S3 内置硬件
5. **功耗规格**：USB ~50-60 mA，BLE ~15-30 mA，双模 ~80 mA
6. **实现方案**：ESP-IDF USB HID + BLE GATT 驱动，统一消息处理
7. **成本最优**：通信模块成本仅 USB-C 连接器（$0.5-1）

---

**文档状态**：初稿完成，待审核
**下一步**：根据技术规格实现通信模块固件