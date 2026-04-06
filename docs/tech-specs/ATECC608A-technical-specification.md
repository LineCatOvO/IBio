# ATECC608A 技术规格文档

**文档版本**：v1.0
**创建时间**：2026-04-06
**作者**：IBio 项目团队
**状态**：初稿

---

## 一、概述

### 1.1 ATECC608A 简介

ATECC608A 是 Microchip Technology 公司生产的安全加密认证芯片，属于 CryptoAuthentication 产品系列。该芯片专门设计用于提供硬件级别的安全密钥存储和加密操作，广泛应用于 IoT 设备、安全认证器、嵌入式系统等领域。

ATECC608A 核心定位：
- **安全密钥存储**：私钥永远无法离开芯片，确保密钥安全
- **加密操作执行**：硬件执行 ECDSA 签名、ECDH 密钥协商、SHA-256 哈希
- **防篡改保护**：内置多种物理和逻辑安全机制
- **低成本方案**：单价仅 $1-2，是市场上最具性价比的安全芯片

### 1.2 核心特性

| 特性 | 描述 |
|------|------|
| **安全等级** | EAL4+（Common Criteria 认证） |
| **密钥存储** | 16 个独立密钥槽位，密钥不可导出 |
| **算法支持** | ECDSA（ES256）、ECDH、SHA-256、AES-128 |
| **接口** | I2C（高达 1 MHz Fast Mode Plus） |
| **工作电压** | 2.0-5.5V（兼容 3.3V 和 5V 系统） |
| **封装** | 8-pad SOIC（5mm x 6mm） |
| **工作温度** | -40°C to +85°C（工业级） |

### 1.3 适用场景

ATECC608A 适用于以下场景：

| 场景 | 说明 |
|------|------|
| **FIDO2 安全认证器** | 私钥存储、ES256 签名、用户验证 |
| **IoT 设备安全** | 设备身份认证、安全通信 |
| **Secure Boot** | 固件签名验证 |
| **密钥管理** | 密钥生成、密钥存储、密钥使用 |
| **防篡改系统** | 物理攻击检测、密钥销毁 |

**IBio 项目应用**：
- 存储 FIDO2 凭证私钥（resident credential）
- 执行 ES256 签名操作
- 实现 ClientPIN 加密通道（ECDH）
- 提供设备 attestation 认证

### 1.4 文档目的

本技术规格文档旨在：
- 详细描述 ATECC608A 的技术参数和规格
- 为 IBio 项目硬件设计和固件开发提供技术依据
- 指导密钥槽位分配和系统集成
- 提供电路设计和应用建议

---

## 二、安全等级规格

### 2.1 安全认证

| 认证项目 | 详情 |
|----------|------|
| **认证标准** | Common Criteria (ISO/IEC 15408) |
| **认证等级** | EAL4+（EAL4 + Flaw Remediation） |
| **认证时间** | 2019 年 |
| **认证机构** | Common Criteria Portal |
| **证书编号** | 可查阅 Microchip 官方文档 |

### 2.2 安全特性

ATECC608A 提供多层次的安全特性：

#### 2.2.1 密钥安全

| 安全特性 | 实现方式 | FIDO2 要求 |
|----------|----------|------------|
| **私钥不可导出** | 密钥在芯片内部生成，永远无法被读取或导出 | 必须 |
| **私钥不可读取** | 密钥只能用于签名/ECDH 操作，无法读取密钥值 | 必须 |
| **密钥隔离存储** | 16 个独立槽位，每个槽位密钥相互隔离 | 必须 |
| **密钥加密存储** | 密钥在芯片内部加密存储，防止物理攻击 | 推荐 |

#### 2.2.2 防篡改检测

| 检测类型 | 实现方式 | 触发响应 |
|----------|----------|----------|
| **电压检测** | 检测异常电压（过高/过低） | 锁定芯片、拒绝操作 |
| **温度检测** | 检测异常温度（超出工作范围） | 锁定芯片、拒绝操作 |
| **频率检测** | 检测异常时钟频率 | 拒绝 I2C 通信 |

#### 2.2.3 防侧信道攻击

| 攻击类型 | 防护措施 | 说明 |
|----------|----------|------|
| **功耗分析攻击** | 功耗随机化、恒定功耗设计 | 防止通过功耗分析推断密钥 |
| **时间分析攻击** | 操作时间恒定 | 防止通过操作时间推断密钥 |
| **电磁辐射攻击** | 电磁屏蔽设计 | 减少电磁辐射泄露 |

#### 2.2.4 安全启动支持

| 功能 | 说明 |
|------|------|
| **Secure Boot** | 可用于固件签名验证 |
| **签名验证** | Verify 命令支持公钥签名验证 |
| **设备认证** | attestation 密钥证明设备真实性 |

### 2.3 物理安全

ATECC608A 提供以下物理安全机制：

| 物理攻击类型 | 防护措施 | 防护效果 |
|--------------|----------|----------|
| **电压攻击** | 电压检测、电压范围限制 | 防止电压注入攻击 |
| **温度攻击** | 温度检测、工作温度限制 | 防止温度攻击 |
| **光照攻击** | 芯片封装遮光设计 | 防止光照攻击 |
| **物理拆解** | 芯片封装保护、密钥加密存储 | 增加拆解难度 |

**EAL4+ 安全等级限制**：
- 可防止软件攻击和简单物理攻击
- 不能防止高级物理攻击（如激光攻击、高级故障注入）
- 适用于 DIY 项目和消费级产品
- 不适用于高安全要求的企业级场景

---

## 三、密钥存储规格

### 3.1 密钥槽位配置

ATECC608A 提供 16 个独立密钥槽位：

| 槽位编号 | 类型 | 容量 | 用途 | 默认权限 |
|----------|------|------|------|----------|
| **Slot 0** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 1** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 2** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 3** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 4** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 5** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 6** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 7** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 8** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 9** | 密钥槽 | 36 bytes | ECC 私钥存储 | 不可读、可签名 |
| **Slot 10** | 数据槽 | 72 bytes | 数据存储/临时密钥 | 可读写、可 ECDH |
| **Slot 11** | 数据槽 | 72 bytes | 数据存储 | 可读写 |
| **Slot 12** | 数据槽 | 72 bytes | 数据存储 | 可读写 |
| **Slot 13** | 数据槽 | 72 bytes | 数据存储 | 可读写 |
| **Slot 14** | 密钥槽 | 36 bytes | attestation 密钥 | 不可读、可签名 |
| **Slot 15** | 密钥槽 | 36 bytes | ECDH 密钥 | 不可读、可 ECDH |

### 3.2 槽位类型详解

#### 3.2.1 密钥槽位（Slot 0-9, 14, 15）

| 属性 | 规格 |
|------|------|
| **存储容量** | 36 bytes（ECC P-256 私钥 32 bytes + 配置） |
| **密钥类型** | ECC 私钥（P-256 或 P-384） |
| **权限** | 可配置（不可读、可签名、可 ECDH、可生成） |
| **密钥生成** | GenKey 命令硬件生成 |
| **密钥使用** | Sign 命令签名、ECDH 命令密钥协商 |

#### 3.2.2 数据槽位（Slot 10-13）

| 属性 | 规格 |
|------|------|
| **存储容量** | 72 bytes（每槽位） |
| **用途** | 通用数据存储、临时密钥存储 |
| **权限** | 可读写、可加密存储 |
| **访问控制** | 可配置读写权限 |

### 3.3 密钥权限配置

每个槽位可配置以下权限：

| 权限类型 | 说明 | 配置值 |
|----------|------|--------|
| **ReadKey** | 密钥读取权限 | 禁止（密钥不可读） |
| **WriteKey** | 密钥写入权限 | 允许（可生成新密钥覆盖） |
| **UseSign** | 签名权限 | 允许（ES256 签名） |
| **UseECDH** | ECDH 权限 | 允许（密钥协商） |
| **UseEncrypt** | 加密权限 | 可配置 |
| **UseVerify** | 验证权限 | 可配置 |
| **IsSecret** | 密钥属性 | 设置（密钥保密） |

### 3.4 密钥生成与存储流程

**密钥生成流程**：
```
1. 调用 GenKey 命令，指定槽位和密钥类型
2. ATECC608A 使用内部 TRNG 生成随机私钥
3. 私钥自动存储在指定槽位，无法被读取
4. 返回对应的公钥（65 bytes uncompressed 格式）
```

**密钥使用流程**：
```
1. 调用 Sign 命令，指定槽位和消息
2. ATECC608A 使用槽位内的私钥签名
3. 返回签名结果（64 bytes raw signature）
4. 私钥始终不离开芯片
```

---

## 四、算法支持规格

### 4.1 ECDSA（ES256）

ATECC608A 支持完整的 ECDSA 签名和验证：

| 参数 | 规格 |
|------|------|
| **算法名称** | ECDSA with SHA-256（ES256） |
| **曲线** | NIST P-256（secp256r1） |
| **曲线参数** | 256 bits（32 bytes） |
| **签名格式** | Raw signature（r + s = 64 bytes） |
| **公钥格式** | Uncompressed（65 bytes：0x04 + X + Y） |
| **签名时间** | 约 50-100 ms |
| **验证时间** | 约 50-100 ms |

**ES256 COSE 算法标识**：
- COSE Algorithm Value：-7
- FIDO2 必须支持的算法

**签名操作流程**：
```
输入：消息（任意长度）、槽位索引
处理：
1. SHA-256 计算消息哈希（32 bytes）
2. 使用槽位内私钥签名
输出：签名（r, s）= 64 bytes
```

**签名格式**：
```
Raw Signature（64 bytes）：
- r（32 bytes）：签名的 r 部分
- s（32 bytes）：签名的 s 部分

需转换为 DER 格式用于某些应用：
- 0x30 [总长度] 0x02 [r长度] [r值] 0x02 [s长度] [s值]
```

### 4.2 ECDH

ATECC608A 支持 ECDH 密钥协商：

| 参数 | 规格 |
|------|------|
| **算法名称** | ECDH with P-256 |
| **曲线** | NIST P-256（secp256r1） |
| **输入** | 对方公钥（65 bytes uncompressed） |
| **输出** | 共享密钥（32 bytes） |
| **用途** | PIN 加密通道、凭证加密 |
| **操作时间** | 约 50-100 ms |

**ECDH 操作流程**：
```
输入：对方公钥（65 bytes）、槽位索引
处理：
1. 使用槽位内私钥计算共享密钥
2. K = privateKey * peerPublicKey（曲线点乘）
输出：共享密钥（32 bytes）
```

**FIDO2 ClientPIN 应用**：
```
1. 客户端生成临时密钥对 pinUvAuthTokenPubKey
2. 认证器使用 PIN 密钥槽位执行 ECDH
3. 计算共享密钥 K = ECDH(pinUvAuthTokenPubKey, PIN_Key)
4. 使用 K 加密 PIN 相关数据
```

### 4.3 SHA-256

ATECC608A 支持 SHA-256 哈希计算：

| 参数 | 规格 |
|------|------|
| **算法名称** | SHA-256 |
| **输入长度** | 任意长度（分块处理） |
| **输出长度** | 32 bytes（256 bits） |
| **块大小** | 64 bytes |
| **操作时间** | 约 10-50 ms（取决于输入长度） |

**SHA-256 使用场景**：
- 消息签名前的哈希计算
- PIN 哈希计算（左 16 bytes）
- rpIdHash 计算
- 挑战哈希

**SHA 命令模式**：
```
SHA 命令支持三种模式：
- SHA_MODE_SHA256_START (0x00)：开始新哈希计算
- SHA_MODE_SHA256_UPDATE (0x01)：追加数据
- SHA_MODE_SHA256_END (0x02)：结束并返回哈希值
```

### 4.4 AES-128（有限支持）

ATECC608A 提供有限的 AES-128 支持：

| 参数 | 规格 |
|------|------|
| **算法名称** | AES-128 |
| **密钥长度** | 128 bits（16 bytes） |
| **加密模式** | CBC 模式（有限） |
| **用途** | 数据加密、密钥加密 |
| **限制** | 功能有限，建议使用主控芯片 AES |

**AES 使用建议**：
- 对于需要完整 AES 功能的场景，建议使用 ESP32-S3 硬件 AES
- ATECC608A AES 主要用于内部数据加密保护

### 4.5 随机数生成（TRNG）

ATECC608A 内置真随机数生成器：

| 参数 | 规格 |
|------|------|
| **类型** | TRNG（True Random Number Generator） |
| **输出长度** | 32 bytes（256 bits） |
| **随机性来源** | 物理噪声源 |
| **质量** | 符合 NIST SP 800-90 标准 |
| **用途** | 密钥生成、挑战生成、nonce |

**Random 命令**：
```
输入：无
输出：32 bytes 随机数
用途：密钥生成、挑战值、nonce
```

---

## 五、接口规格

### 5.1 I2C 接口参数

| 参数 | 规格 | 说明 |
|------|------|------|
| **接口类型** | I2C（Inter-Integrated Circuit） | 标准两线接口 |
| **I2C 地址** | 0xC0（8-bit）/ 0x60（7-bit） | 默认地址，可配置 |
| **时钟频率** | 最高 1 MHz | Fast Mode Plus |
| **最低频率** | 100 kHz | Standard Mode |
| **数据格式** | 标准 I2C 数据格式 | 符合 I2C 规范 |
| **CRC 校验** | CRC-16 | 每个数据包包含 CRC |

### 5.2 I2C 通信时序

| 时序参数 | 规格 |
|----------|------|
| **响应等待时间** | 最大 50 ms（取决于命令类型） |
| **命令执行时间** | 20-50 ms（取决于命令） |
| **数据包间隔** | 最小 1.3 ms |
| **START-STOP 间隔** | 最小 4.7 µs（1 MHz） |

### 5.3 I2C 数据包格式

**写数据包格式**：
```
[Start] [Device Address + Write] [Word Address] [Count] [Data] [CRC-16] [Stop]
```

| 字段 | 长度 | 说明 |
|------|------|------|
| **Start** | 1 bit | I2C 开始条件 |
| **Device Address** | 8 bits | 设备地址（0xC0）+ 读写位（0=写） |
| **Word Address** | 1 byte | 命令类型标识 |
| **Count** | 1 byte | 数据长度 |
| **Data** | N bytes | 命令参数 |
| **CRC-16** | 2 bytes | 数据 CRC 校验 |
| **Stop** | 1 bit | I2C 停止条件 |

**读数据包格式**：
```
[Start] [Device Address + Read] [Count] [Data] [CRC-16] [Stop]
```

| 字段 | 长度 | 说明 |
|------|------|------|
| **Start** | 1 bit | I2C 开始条件 |
| **Device Address** | 8 bits | 设备地址（0xC0）+ 读写位（1=读） |
| **Count** | 1 byte | 返回数据长度 |
| **Data** | N bytes | 响应数据 |
| **CRC-16** | 2 bytes | 数据 CRC 校验 |
| **Stop** | 1 bit | I2C 停止条件 |

### 5.4 ESP32-S3 I2C 连接方案

| ESP32-S3 引脚 | ATECC608A 引脚 | 功能 | 说明 |
|---------------|----------------|------|------|
| **GPIO21** | SDA | I2C 数据线 | 双向数据传输 |
| **GPIO22** | SCL | I2C 时钟线 | 时钟信号输出 |
| **GPIO23** | RST（可选） | 复位信号 | 芯片复位控制 |
| **3.3V** | VCC | 电源 | 3.3V 供电 |
| **GND** | GND | 地 | 地线连接 |

**I2C 配置建议**：
- I2C 频率：400 kHz（推荐）或 1 MHz（最大）
- 上拉电阻：4.7kΩ（外部）或使用 ESP32-S3 内部上拉
- 复位信号可选，可通过 I2C 命令复位

---

## 六、功能规格

### 6.1 核心命令列表

ATECC608A 支持以下核心命令：

| 命令 | 命令码 | 功能 | 参数 | 输出 |
|------|--------|------|------|------|
| **GenKey** | 0x40 | 生成密钥对 | Slot, Mode, OtherData | PublicKey (65 bytes) |
| **Sign** | 0x41 | 签名操作 | Slot, Message | Signature (64 bytes) |
| **Verify** | 0x45 | 验证签名 | Slot/Key, Message, Signature | Success/Fail |
| **ECDH** | 0x43 | 密钥协商 | Slot, PubKey | SharedSecret (32 bytes) |
| **SHA** | 0x47 | SHA-256 哈希 | Mode, Data | Hash (32 bytes) |
| **Random** | 0x1B | 随机数生成 | - | Random (32 bytes) |
| **Read** | 0x02 | 读取数据/公钥 | Slot, Offset, Length | Data |
| **Write** | 0x12 | 写入数据 | Slot, Offset, Data | Success/Fail |
| **Lock** | 0x17 | 锁定配置/数据 | Zone, Summary | Success/Fail |
| **Counter** | 0x24 | 单调计数器 | CounterId, Value | CounterValue |
| **Info** | 0x30 | 获取信息 | Mode | InfoData |
| **Nonce** | 0x16 | 生成 nonce | Mode, Input | TempKey/Nonce |
| **MAC** | 0x28 | 计算 MAC | Mode, KeyId, Challenge | MAC (32 bytes) |
| **CheckMAC** | 0x29 | 验证 MAC | Mode, KeyId, MAC | Success/Fail |
| **DeriveKey** | 0x1D | 派生密钥 | Mode, KeyId | Success/Fail |
| **SelfTest** | 0x33 | 自测试 | Mode | Success/Fail |

### 6.2 密钥生成功能

**GenKey 命令详解**：

| 参数 | 说明 |
|------|------|
| **Mode** | 生成模式（0x04=生成新密钥） |
| **KeyId** | 目标槽位（0-15） |
| **OtherData** | 可选附加数据（用于确定性密钥） |

**生成模式**：
| Mode 值 | 说明 |
|---------|------|
| 0x04 | 使用 TRNG 生成新密钥 |
| 0x08 | 从 TempKey 派生密钥 |
| 0x10 | 确定性密钥生成 |

**输出**：
- 公钥（65 bytes uncompressed 格式）
- 私钥自动存储在指定槽位，无法读取

### 6.3 签名操作功能

**Sign 命令详解**：

| 参数 | 说明 |
|------|------|
| **Mode** | 签名模式（0x00=外部消息，0x80=内部消息） |
| **KeyId** | 密钥槽位（0-15） |
| **Message** | 待签名消息（32 bytes） |

**签名流程**：
```
1. 接收 32 bytes 消息（通常是 SHA-256 哈希）
2. 使用指定槽位内的私钥签名
3. 返回 64 bytes 签名（r + s）
```

**FIDO2 ES256 签名应用**：
- 输入：挑战哈希（32 bytes）
- 处理：使用凭证私钥槽位签名
- 输出：ES256 签名（64 bytes）

### 6.4 验证签名功能

**Verify 命令详解**：

| 参数 | 说明 |
|------|------|
| **Mode** | 验证模式（使用外部公钥或内部槽位公钥） |
| **KeyId** | 公钥槽位或外部公钥 |
| **Message** | 待验证消息（32 bytes） |
| **Signature** | 待验证签名（64 bytes） |

**验证模式**：
| Mode 值 | 说明 |
|---------|------|
| 0x00 | 使用外部公钥验证 |
| 0x01 | 使用槽位内公钥验证 |
| 0x02 | 验证存储公钥 |

### 6.5 密钥协商功能

**ECDH 命令详解**：

| 参数 | 说明 |
|------|------|
| **Mode** | ECDH 模式 |
| **KeyId** | 私钥槽位 |
| **PubKey** | 对方公钥（65 bytes） |

**ECDH 流程**：
```
1. 接收对方公钥（65 bytes uncompressed）
2. 使用指定槽位内的私钥计算共享密钥
3. 返回共享密钥（32 bytes）
```

**FIDO2 ClientPIN 应用**：
- Slot 15 存储 PIN 密钥
- 客户端发送临时公钥
- ECDH 计算共享密钥用于 AES 加密

### 6.6 SHA-256 哈希功能

**SHA 命令详解**：

| 参数 | 说明 |
|------|------|
| **Mode** | SHA 模式（Start/Update/End） |
| **Data** | 待哈希数据 |

**SHA 模式**：
| Mode 值 | 说明 |
|---------|------|
| 0x00 | SHA_MODE_SHA256_START - 开始新哈希 |
| 0x01 | SHA_MODE_SHA256_UPDATE - 追加数据 |
| 0x02 | SHA_MODE_SHA256_END - 结束并返回哈希 |

**SHA 使用流程**：
```
1. SHA Start：初始化哈希上下文
2. SHA Update：追加数据块
3. SHA End：结束哈希，返回 32 bytes 哈希值
```

### 6.7 随机数生成功能

**Random 命令详解**：

| 参数 | 说明 |
|------|------|
| **Mode** | 随机数模式 |

**输出**：
- 32 bytes 真随机数

**用途**：
- 密钥生成随机源
- 挑战值生成
- nonce 生成
- 防重放令牌

### 6.8 数据读写功能

**Read 命令详解**：

| 参数 | 说明 |
|------|------|
| **Zone** | 读取区域（Config/OTP/Data/Key） |
| **Slot** | 槽位编号 |
| **Offset** | 数据偏移 |
| **Length** | 数据长度 |

**Write 命令详解**：

| 参数 | 说明 |
|------|------|
| **Zone** | 写入区域 |
| **Slot** | 槽位编号 |
| **Offset** | 数据偏移 |
| **Data** | 写入数据 |

### 6.9 槽位锁定功能

**Lock 命令详解**：

| 参数 | 说明 |
|------|------|
| **Zone** | 锁定区域（Config/Data） |
| **Summary** | CRC 校验值 |

**锁定类型**：
| Zone 值 | 说明 |
|---------|------|
| 0x00 | LockZoneConfig - 锁定配置区 |
| 0x01 | LockZoneOTP - 锁定 OTP 区 |
| 0x02 | LockZoneData - 锁定数据区 |

**重要说明**：
- 锁定操作不可逆，锁定后无法修改配置
- 建议在生产前测试配置，确认后再锁定

---

## 七、功耗规格

### 7.1 电压参数

| 参数 | 规格 | 说明 |
|------|------|------|
| **工作电压范围** | 2.0V - 5.5V | 兼容 3.3V 和 5V 系统 |
| **推荐工作电压** | 3.3V | 与 ESP32-S3 电压匹配 |
| **最大电压** | 5.5V | 超过可能导致损坏 |
| **最小电压** | 2.0V | 低于可能无法工作 |

### 7.2 电流参数

| 参数 | 规格 | 条件 |
|------|------|------|
| **工作电流** | 5-10 mA | 签名/ECDH 操作时 |
| **待机电流** | < 1 mA | Idle 模式 |
| **休眠电流** | < 100 µA | Sleep 模式 |
| **最大电流** | 15 mA | 峰值电流 |

### 7.3 功耗模式

| 模式 | 说明 | 进入条件 | 退出条件 |
|------|------|----------|----------|
| **Active** | 活动模式，执行命令 | I2C 命令触发 | 命令完成 |
| **Idle** | 待机模式，等待命令 | 命令完成后自动进入 | I2C 通信 |
| **Sleep** | 低功耗休眠模式 | Sleep 命令或长时间无通信 | Wake 信号或 I2C |

### 7.4 功耗优化建议

| 建议 | 说明 |
|------|------|
| **电压选择** | 使用 3.3V 供电，与 ESP32-S3 共用电源 |
| **休眠模式** | 不使用时发送 Sleep 命令进入休眠 |
| **唤醒优化** | 使用 GPIO 唤醒而非 I2C 唤醒 |
| **批量操作** | 连续操作减少唤醒次数 |

---

## 八、物理规格

### 8.1 封装信息

| 参数 | 规格 |
|------|------|
| **封装类型** | 8-pad SOIC（Small Outline Integrated Circuit） |
| **封装代码** | 8-lead SOIC (150 mil) |
| **封装尺寸** | 5mm x 6mm x 1.5mm |
| **引脚间距** | 1.27mm (50 mil) |
| **封装材质** | 塑料封装 |

### 8.2 尺寸规格

```
ATECC608A 尺寸图：

        ┌────────────────────┐
        │                    │
        │   ATECC608A        │  Height: 1.5mm
        │                    │
        │   5mm x 6mm        │
        │                    │
        └────────────────────┘
```

### 8.3 工作环境

| 参数 | 规格 | 说明 |
|------|------|------|
| **工作温度范围** | -40°C to +85°C | 工业级温度范围 |
| **存储温度范围** | -40°C to +85°C | 存储环境温度 |
| **湿度范围** | 0-95% RH（非凝露） | 相对湿度 |

### 8.4 ESD 保护

| 参数 | 规格 |
|------|------|
| **ESD 保护等级** | HBM > 2000V |
| **人体模型（HBM）** | 符合 JEDEC 标准 |

### 8.5 机械特性

| 参数 | 规格 |
|------|------|
| **重量** | 约 0.1g |
| **焊接方式** | 表面贴装（SMT） |
| **回流焊温度** | 最高 260°C |
| **焊接时间** | < 10 秒（260°C 以上） |

---

## 九、引脚定义

### 9.1 引脚图

```
ATECC608A 8-pad SOIC 引脚图：

        ┌───────────────────┐
        │                   │
   VCC  │ o1             o8 │  GND
        │                   │
        │                   │
   SCL  │ o2             o7 │  NC
        │                   │
        │                   │
   SDA  │ o3             o6 │  NC
        │                   │
        │                   │
   RST  │ o4             o5 │  NC
        │                   │
        └───────────────────┘
```

### 9.2 引脚功能表

| 引脚编号 | 引脚名称 | 类型 | 功能说明 |
|----------|----------|------|----------|
| **Pad 1** | VCC | Power | 电源输入（2.0-5.5V） |
| **Pad 2** | SCL | Input | I2C 时钟线 |
| **Pad 3** | SDA | I/O | I2C 数据线（双向） |
| **Pad 4** | RST | Input | 复位信号（可选） |
| **Pad 5** | NC | - | 无连接 |
| **Pad 6** | NC | - | 无连接 |
| **Pad 7** | NC | - | 无连接 |
| **Pad 8** | GND | Power | 地线 |

### 9.3 引脚电气特性

| 引脚 | 参数 | 规格 |
|------|------|------|
| **VCC** | 输入电压范围 | 2.0V - 5.5V |
| **VCC** | 输入电流 | 最大 15 mA |
| **SCL** | 输入高电平 | > 0.7 * VCC |
| **SCL** | 输入低电平 | < 0.3 * VCC |
| **SDA** | 输入高电平 | > 0.7 * VCC |
| **SDA** | 输入低电平 | < 0.3 * VCC |
| **SDA** | 输出低电平 | < 0.4V（@ 3.5mA） |
| **RST** | 输入高电平 | > 0.7 * VCC |
| **RST** | 输入低电平 | < 0.3 * VCC |

### 9.4 引脚连接建议

| 引脚 | 连接建议 |
|------|----------|
| **VCC** | 连接到稳定的 3.3V 电源，建议添加滤波电容 |
| **GND** | 连接到系统地线 |
| **SCL** | 连接到 I2C 时钟线，添加 4.7kΩ 上拉电阻 |
| **SDA** | 连接到 I2C 数据线，添加 4.7kΩ 上拉电阻 |
| **RST** | 可连接到 MCU GPIO，或悬空（使用 I2C 复位） |
| **NC** | 悬空，无需连接 |

---

## 十、通信协议

### 10.1 命令包格式

**I2C 命令包格式**：

```
┌───────────┬───────────┬───────────┬───────────────┬───────────┐
│ Word Addr │  Length   │  Command  │   Parameters  │   CRC-16  │
│  (1 byte) │  (1 byte) │  (1 byte) │   (N bytes)   │  (2 bytes)│
└───────────┴───────────┴───────────┴───────────────┴───────────┘
```

| 字段 | 长度 | 说明 |
|------|------|------|
| **Word Address** | 1 byte | 命令类型标识（0x03 = 命令） |
| **Length** | 1 byte | 整个数据包长度（不含 CRC） |
| **Command** | 1 byte | 命令码（如 0x40 = GenKey） |
| **Parameters** | N bytes | 命令参数（取决于命令） |
| **CRC-16** | 2 bytes | CRC 校验值 |

### 10.2 响应包格式

**I2C 响应包格式**：

```
┌───────────┬───────────┬───────────────┬───────────┐
│  Length   │  Status   │     Data      │   CRC-16  │
│  (1 byte) │  (1 byte) │   (N bytes)   │  (2 bytes)│
└───────────┴───────────┴───────────────┴───────────┘
```

| 字段 | 度度 | 说明 |
|------|------|------|
| **Length** | 1 byte | 整个响应包长度（不含 CRC） |
| **Status** | 1 byte | 状态码 |
| **Data** | N bytes | 响应数据（取决于命令） |
| **CRC-16** | 2 bytes | CRC 校验值 |

### 10.3 状态码定义

| 状态码 | 名称 | 说明 |
|--------|------|------|
| **0x00** | SUCCESS | 操作成功 |
| **0x01** | CHECKMAC_FAIL | MAC 验证失败 |
| **0x03** |PARSE_FAIL | 命令解析失败 |
| **0x05** | ECC_FAIL | ECC 操作失败 |
| **0x07** | SELF_TEST_FAIL | 自测试失败 |
| **0x0F** | INVALID_SIZE | 数据大小无效 |
| **0x11** | INVALID_COMMAND | 无效命令 |
| **0x12** | RX_FAIL | 接收失败 |
| **0x14** | RESYNC_FAIL | 同步失败 |
| **0x20** | WATCHDOG_EXPIRED | 看门狗超时 |
| **0x21** | INVALID_ZONE | 无效区域 |
| **0x22** | LOCKED | 区域已锁定 |
| **0x23** | UNKNOWN | 未知错误 |

### 10.4 CRC 计算

**CRC-16 计算方法**：
```
CRC-16-CCITT
Polynomial: 0x1021
Initial Value: 0xFFFF
计算范围：整个数据包（不含 CRC 字段）
```

### 10.5 常用命令详解

#### 10.5.1 GenKey 命令（0x40）

**命令包**：
```
WordAddr: 0x03
Length: 0x07
Command: 0x40
Param1: Mode (0x04 = Generate)
Param2: KeyId (Slot 0-15)
CRC-16: [CRC bytes]
```

**响应包**：
```
Length: 0x43
Status: 0x00 (Success)
Data: PublicKey (65 bytes)
CRC-16: [CRC bytes]
```

#### 10.5.2 Sign 命令（0x41）

**命令包**：
```
WordAddr: 0x03
Length: 0x27
Command: 0x41
Param1: Mode (0x80 = Internal, 0x00 = External)
Param2: KeyId (Slot 0-15)
Param3: Message (32 bytes)
CRC-16: [CRC bytes]
```

**响应包**：
```
Length: 0x42
Status: 0x00 (Success)
Data: Signature (64 bytes)
CRC-16: [CRC bytes]
```

#### 10.5.3 ECDH 命令（0x43）

**命令包**：
```
WordAddr: 0x03
Length: 0x47
Command: 0x43
Param1: Mode
Param2: KeyId (Slot 0-15)
Param3: PubKey (65 bytes)
CRC-16: [CRC bytes]
```

**响应包**：
```
Length: 0x22
Status: 0x00 (Success)
Data: SharedSecret (32 bytes)
CRC-16: [CRC bytes]
```

#### 10.5.4 SHA 命令（0x47）

**命令包**：
```
WordAddr: 0x03
Length: [Variable]
Command: 0x47
Param1: Mode (Start/Update/End)
Param2: [Reserved]
Param3: Data (Variable length)
CRC-16: [CRC bytes]
```

**响应包**（SHA End）：
```
Length: 0x22
Status: 0x00 (Success)
Data: Hash (32 bytes)
CRC-16: [CRC bytes]
```

#### 10.5.5 Random 命令（0x1B）

**命令包**：
```
WordAddr: 0x03
Length: 0x03
Command: 0x1B
Param1: Mode (0x00)
CRC-16: [CRC bytes]
```

**响应包**：
```
Length: 0x22
Status: 0x00 (Success)
Data: Random (32 bytes)
CRC-16: [CRC bytes]
```

#### 10.5.6 Read 命令（0x02）

**命令包**：
```
WordAddr: 0x03
Length: 0x07
Command: 0x02
Param1: Zone (Config/Data/OTP)
Param2: Address
Param3: Length (optional)
CRC-16: [CRC bytes]
```

**响应包**：
```
Length: [Variable]
Status: 0x00 (Success)
Data: [Read data]
CRC-16: [CRC bytes]
```

#### 10.5.7 Write 命令（0x12）

**命令包**：
```
WordAddr: 0x03
Length: [Variable]
Command: 0x12
Param1: Zone
Param2: Address
Param3: Data
CRC-16: [CRC bytes]
```

**响应包**：
```
Length: 0x03
Status: 0x00 (Success)
CRC-16: [CRC bytes]
```

### 10.6 通信时序要求

| 操作 | 时序要求 |
|------|----------|
| **命令发送后等待** | 等待芯片响应（最大 50 ms） |
| **唤醒序列** | 发送 Wake 信号，等待 1.3 ms |
| **Sleep 序列** | 发送 Sleep 命令，立即进入休眠 |
| **I2C 间隔** | 命令间最小间隔 1.3 ms |

---

## 十一、电气特性

### 11.1 最大额定值

| 参数 | 最大值 | 说明 |
|------|--------|------|
| **供电电压** | 5.5V | 超过可能导致损坏 |
| **工作电流** | 15 mA | 峰值电流 |
| **存储温度** | -40°C to +125°C | 存储环境温度 |
| **ESD（HBM）** | 2000V | 人体模型 ESD |

### 11.2 推荐工作条件

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| **供电电压** | 3.3V | 与 ESP32-S3 匹配 |
| **I2C 频率** | 400 kHz | 推荐 I2C 时钟频率 |
| **工作温度** | 0°C to +70°C | 推荐工作温度范围 |

### 11.3 DC 电气特性

| 参数 | 条件 | 最小 | 典型 | 最大 |
|------|------|------|------|------|
| **VCC 工作电压** | - | 2.0V | 3.3V | 5.5V |
| **ICC 工作电流** | 签名操作 | - | 8 mA | 15 mA |
| **ICC 待机电流** | Idle 模式 | - | 0.5 mA | 1 mA |
| **ICC 休眠电流** | Sleep 模式 | - | 30 µA | 100 µA |
| **VIL 输入低电平** | SCL/SDA | - | - | 0.3VCC |
| **VIH 输入高电平** | SCL/SDA | 0.7VCC | - | - |
| **VOL 输出低电平** | SDA @ 3.5mA | - | - | 0.4V |

### 11.4 AC 电气特性（I2C）

| 参数 | 符号 | Fast Mode Plus | 说明 |
|------|------|----------------|------|
| **时钟频率** | fSCL | 100 kHz - 1 MHz | I2C 时钟频率 |
| **START 保持时间** | tHD;STA | 0.26 µs | START 条件保持 |
| **SCL 低电平时间** | tLOW | 0.5 µs | SCL 低电平持续时间 |
| **SCL 高电平时间** | tHIGH | 0.26 µs | SCL 高电平持续时间 |
| **数据建立时间** | tSU;DAT | 50 ns | 数据建立时间 |
| **数据保持时间** | tHD;DAT | 0 ns | 数据保持时间 |
| **STOP 建立时间** | tSU;STO | 0.26 µs | STOP 条件建立 |

---

## 十二、密钥槽位分配建议

### 12.1 IBio 项目密钥槽位分配方案

基于 FIDO2 功能需求，推荐以下槽位分配方案：

| 槽位 | 用途 | 密钥类型 | 权限配置 | 说明 |
|------|------|----------|----------|------|
| **Slot 0** | Resident Credential 1 | ES256 私钥 | Sign, NoRead | 第一个凭证密钥 |
| **Slot 1** | Resident Credential 2 | ES256 私钥 | Sign, NoRead | 第二个凭证密钥 |
| **Slot 2** | Resident Credential 3 | ES256 私钥 | Sign, NoRead | 第三个凭证密钥 |
| **Slot 3** | Resident Credential 4 | ES256 私钥 | Sign, NoRead | 第四个凭证密钥 |
| **Slot 4** | Resident Credential 5 | ES256 私钥 | Sign, NoRead | 扩展凭证密钥 |
| **Slot 5** | Resident Credential 6 | ES256 私钥 | Sign, NoRead | 扩展凭证密钥 |
| **Slot 6** | Resident Credential 7 | ES256 私钥 | Sign, NoRead | 扩展凭证密钥 |
| **Slot 7** | Resident Credential 8 | ES256 私钥 | Sign, NoRead | 扩展凭证密钥 |
| **Slot 8** | Resident Credential 9 | ES256 私钥 | Sign, NoRead | 扩展凭证密钥 |
| **Slot 9** | Resident Credential 10 | ES256 私钥 | Sign, NoRead | 扩展凭证密钥 |
| **Slot 10** | ECDH 临时密钥 | ECDH 密钥 | ECDH, Read | ClientPIN 加密通道临时密钥 |
| **Slot 11** | PIN Token 存储 | 数据 | Read, Write | PIN 加密 token 存储 |
| **Slot 12** | 保留 | - | - | 未来扩展 |
| **Slot 13** | 保留 | - | - | 未来扩展 |
| **Slot 14** | Attestation 密钥 | ES256 私钥 | Sign, NoRead | 认证器 attestation 密钥 |
| **Slot 15** | PIN 加密密钥 | ECDH 密钥 | ECDH, NoRead | ClientPIN 加密主密钥 |

### 12.2 槽位权限配置详解

#### 12.2.1 凭证密钥槽位（Slot 0-9）

**推荐权限配置**：
- **ReadKey**：禁止（私钥不可读取）
- **WriteKey**：允许（可生成新密钥覆盖）
- **UseSign**：允许（ES256 签名）
- **UseECDH**：禁止（凭证密钥不用于 ECDH）
- **IsSecret**：设置（密钥保密属性）
- **NoMac**：设置（禁用 MAC 操作）

#### 12.2.2 Attestation 密钥槽位（Slot 14）

**推荐权限配置**：
- **ReadKey**：禁止（私钥不可读取）
- **WriteKey**：禁止（生产时写入，之后锁定）
- **UseSign**：允许（attestation 签名）
- **IsSecret**：设置
- **Lockable**：设置（锁定后不可修改）

#### 12.2.3 PIN 加密密钥槽位（Slot 15）

**推荐权限配置**：
- **ReadKey**：禁止（私钥不可读取）
- **WriteKey**：禁止（生产时写入，之后锁定）
- **UseECDH**：允许（密钥协商）
- **IsSecret**：设置
- **Lockable**：设置

### 12.3 槽位容量分析

| 槽位类型 | 可存储密钥数 | 实际使用数 | 剩余容量 |
|----------|--------------|------------|----------|
| **凭证密钥槽** | 10 | 4-10 | 可扩展 |
| **数据槽** | 4 | 2 | 2 保留 |
| **特殊密钥槽** | 2 | 2 | 0 |

**容量评估**：
- 10 个 resident credential 槽位足够大多数用户需求
- FIDO2 标准支持服务器存储凭证，减少本地存储需求
- 保留槽位可用于未来功能扩展

---

## 十三、应用建议

### 13.1 电路设计建议

#### 13.1.1 电源设计

| 建议 | 说明 |
|------|------|
| **电源选择** | 使用 ESP32-S3 的 3.3V 输出供电 |
| **滤波电容** | VCC 引脚添加 100nF 滤波电容 |
| **电源稳定** | 确保电源纹波 < 100mV |
| **电源隔离** | 可考虑与主控芯片电源隔离 |

#### 13.1.2 I2C 设计

| 建议 | 说明 |
|------|------|
| **上拉电阻** | 使用 4.7kΩ 外部上拉电阻 |
| **上拉电压** | 上拉到 VCC（3.3V） |
| **信号保护** | 可添加 TVS 二极管保护 |
| **走线长度** | I2C 走线尽量短 |

#### 13.1.3 PCB 设计

| 建议 | 说明 |
|------|------|
| **布局位置** | 尽量靠近 ESP32-S3 |
| **地线设计** | 确保良好的地线连接 |
| **信号完整性** | I2C 走线远离高速信号 |
| **ESD 保护** | 添加适当的 ESD 保护措施 |

### 13.2 I2C 配置建议

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| **I2C 频率** | 400 kHz | 推荐，稳定可靠 |
| **最大频率** | 1 MHz | 可用，需注意信号完整性 |
| **超时时间** | 50 ms | 响应超时设置 |
| **重试次数** | 3 次 | I2C 通信失败重试 |

### 13.3 CryptoAuthLib 集成建议

#### 13.3.1 库集成步骤

| 步骤 | 内容 |
|------|------|
| **1. 导入库** | 从 GitHub 下载 CryptoAuthLib |
| **2. 配置 HAL** | 配置 ESP32-S3 I2C HAL 层 |
| **3. 初始化** | 调用 atcab_init() 初始化芯片 |
| **4. 测试通信** | 调用 atcab_info() 测试芯片通信 |
| **5. 功能测试** | 测试 GenKey、Sign、ECDH 功能 |

#### 13.3.2 关键 API

| API | 功能 | 说明 |
|------|------|------|
| `atcab_init()` | 初始化芯片 | 配置 I2C 接口 |
| `atcab_info()` | 获取芯片信息 | 测试通信 |
| `atcab_genkey()` | 生成密钥 | GenKey 命令 |
| `atcab_sign()` | 签名操作 | Sign 命令 |
| `atcab_ecdh()` | 密钥协商 | ECDH 命令 |
| `atcab_sha()` | SHA-256 计算 | SHA 命令 |
| `atcab_random()` | 随机数生成 | Random 命令 |
| `atcab_read()` | 读取数据 | Read 命令 |
| `atcab_write()` | 写入数据 | Write 命令 |
| `atcab_lock()` | 锁定配置 | Lock 命令 |

### 13.4 密钥管理建议

#### 13.4.1 密钥生成流程

```
1. 检查槽位是否可用
2. 调用 GenKey 命令生成密钥
3. 读取公钥并记录
4. 更新凭证管理表
```

#### 13.4.2 密钥使用流程

```
1. 根据凭证 ID 查找槽位索引
2. 调用 Sign 命令签名
3. 返回签名结果
```

#### 13.4.3 密钥销毁流程

```
1. 确认需要销毁的凭证
2. 使用 GenKey 命令生成新密钥覆盖槽位
3. 更新凭证管理表
```

### 13.5 安全配置建议

#### 13.5.1 芯片锁定

| 建议 | 说明 |
|------|------|
| **配置锁定** | 生产前锁定配置区 |
| **数据锁定** | 根据需求决定是否锁定数据区 |
| **锁定验证** | 锁定前验证所有配置 |

#### 13.5.2 安全启动

| 建议 | 说明 |
|------|------|
| **固件签名** | 使用 ATECC608A Verify 命令验证固件签名 |
| **Secure Boot** | ESP32-S3 Secure Boot 与 ATECC608A 配合使用 |
| **密钥管理** | 使用专用槽位存储 Secure Boot 密钥 |

#### 13.5.3 防篡改

| 建议 | 说明 |
|------|------|
| **物理封装** | 使用坚固的外壳封装 |
| **电压检测** | 监控电压异常 |
| **温度检测** | 监控温度异常（可选） |

---

## 十四、参考资料

### 14.1 官方文档

| 文档 | 链接 |
|------|------|
| **ATECC608A 产品页面** | https://www.microchip.com/en-us/product/ATECC608A |
| **ATECC608A 数据手册** | https://ww1.microchip.com/downloads/en/DeviceDoc/ATECC608A-CryptoAuthentication-Device-Summary-Data-Sheet-DS40001959B.pdf |
| **ATECC608A 技术手册** | https://ww1.microchip.com/downloads/en/DeviceDoc/ATECC608A-TN-CryptoAuthentication-Technical-Manual-DS40002239A.pdf |
| **CryptoAuthLib GitHub** | https://github.com/MicrochipTech/cryptoauthlib |
| **CryptoAuthLib 文档** | https://microchip.github.io/cryptoauthlib/ |
| **CryptoAuthLib ESP32 示例** | https://github.com/MicrochipTech/cryptoauthlib-esp-idf |

### 14.2 开源项目参考

| 项目 | 链接 | 说明 |
|------|------|------|
| **SoloKeys** | https://github.com/solokeys/solo | 开源 FIDO2 认证器，使用 ATECC608A |
| **OpenSK** | https://github.com/google/OpenSK | Google 开源 FIDO2 认证器 |
| **libfido2** | https://github.com/Yubico/libfido2 | Yubico FIDO2 库 |

### 14.3 应用笔记

| 文档 | 链接 |
|------|------|
| **ATECC608A FIDO2 应用** | https://www.microchip.com/wwwAppNotes/AppNotes.aspx?appnote=en594291 |
| **CryptoAuthentication 设计指南** | https://www.microchip.com/wwwAppNotes/AppNotes.aspx?appnote=en544945 |
| **I2C 通信指南** | https://www.microchip.com/wwwAppNotes/AppNotes.aspx?appnote=en544946 |

### 14.4 相关标准

| 标准 | 链接 |
|------|------|
| **FIDO2 标准** | https://fidoalliance.org/specs/fido-v2.0-ps-20190130/ |
| **CTAP2 协议** | https://fidoalliance.org/specs/fido-v2.0-ps-20190130/fido-client-to-authenticator-protocol-v2.0-ps-20190130.html |
| **WebAuthn 标准** | https://www.w3.org/TR/webauthn/ |
| **Common Criteria** | https://www.commoncriteriaportal.org/ |

### 14.5 相关决策文档

| 文档 | 路径 |
|------|------|
| **ADR-004 安全芯片选型** | docs/adr/ADR-004-security-chip-selection.md |
| **HLD 系统架构设计** | docs/HLD.md |
| **FIDO2 需求分析** | docs/research/fido2-requirements.md |

---

## 十五、附录

### 15.1 常用命令速查表

| 命令 | 命令码 | 参数 | 输出 | 用途 |
|------|--------|------|------|------|
| **GenKey** | 0x40 | Mode, Slot | PublicKey | 密钥生成 |
| **Sign** | 0x41 | Mode, Slot, Msg | Signature | ES256 签名 |
| **ECDH** | 0x43 | Mode, Slot, PubKey | SharedSecret | 密钥协商 |
| **SHA** | 0x47 | Mode, Data | Hash | SHA-256 |
| **Random** | 0x1B | - | Random | 随机数 |
| **Read** | 0x02 | Zone, Addr | Data | 读数据 |
| **Write** | 0x12 | Zone, Addr, Data | - | 写数据 |
| **Lock** | 0x17 | Zone, CRC | - | 锁定 |

### 15.2 状态码速查表

| 状态码 | 说明 |
|--------|------|
| 0x00 | 成功 |
| 0x01 | MAC 验证失败 |
| 0x03 | 解析失败 |
| 0x05 | ECC 操作失败 |
| 0x0F | 数据大小无效 |
| 0x11 | 无效命令 |
| 0x22 | 区域已锁定 |

### 15.3 I2C 地址配置

| 配置 | 地址 |
|------|------|
| **默认地址（8-bit）** | 0xC0 |
| **默认地址（7-bit）** | 0x60 |
| **读地址** | 0xC1 |
| **写地址** | 0xC0 |

---

**文档状态**：初稿完成
**下次审核**：集成 CryptoAuthLib 时验证技术参数