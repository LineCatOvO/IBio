# IBio 软件需求规格文档（SRS）

**文档版本**：v1.0
**创建时间**：2026-04-07
**作者**：IBio 项目团队
**状态**：初稿

---

## 一、引言

### 1.1 目的

本文档是 IBio 项目的软件需求规格文档（Software Requirements Specification, SRS），旨在详细描述 IBio FIDO2 安全密钥的功能需求、接口需求、非功能需求、业务规则和数据需求。

本文档的目标读者包括：
- 固件开发工程师
- 硬件设计工程师
- 测试工程师
- 项目管理人员
- 技术审核人员

### 1.2 范围

IBio 是一个开源的 FIDO2 安全密钥项目，集成了指纹生物识别验证功能，提供 USB + BLE 双模通信。

**项目范围**：
- 实现 FIDO2/CTAP2.0 标准认证器核心功能
- 集成指纹生物识别验证
- 提供 USB HID + BLE GATT 双模通信
- 硬件成本目标 < $31
- 开源硬件设计，便于社区复用

**本文档范围**：
- CTAP 协议功能需求（MakeCredential、GetAssertion 等）
- 硬件接口、软件接口、通信接口规格
- FIDO2 合规业务规则
- 性能、安全、可靠性非功能需求
- 凭证和指纹模板数据结构
- 功能、性能、安全验收标准

### 1.3 定义与缩略语

| 缩略语 | 全称 | 定义 |
|--------|------|------|
| **FIDO2** | Fast IDentity Online 2.0 | FIDO 联盟制定的无密码认证标准 |
| **CTAP** | Client-to-Authenticator Protocol | 客户端与认证器通信协议 |
| **WebAuthn** | Web Authentication | W3C 制定的浏览器 API 标准 |
| **ES256** | ECDSA with SHA-256 | 椭圆曲线数字签名算法 |
| **ECDH** | Elliptic Curve Diffie-Hellman | 椭圆曲线密钥协商协议 |
| **HID** | Human Interface Device | 人机接口设备 |
| **GATT** | Generic Attribute Profile | BLE 属性协议 |
| **UV** | User Verification | 用户验证 |
| **FAR** | False Acceptance Rate | 误识率 |
| **FRR** | False Rejection Rate | 拒识率 |
| **CBOR** | Concise Binary Object Representation | 简洁二进制对象表示 |
| **COSE** | CBOR Object Signing and Encryption | CBOR 对象签名和加密 |
| **RP** | Relying Party | 依赖方（网站/服务） |
| **Credential** | 凭证 | FIDO2 认证凭证，包含密钥和元数据 |
| **Assertion** | 断言 | FIDO2 认证响应，包含签名和元数据 |
| **Attestation** | 认证器认证 | 证明认证器真实性的签名 |

### 1.4 参考资料

| 参考资料 | 版本/日期 | 来源 |
|----------|-----------|------|
| **WebAuthn Level 2** | W3C Recommendation | https://www.w3.org/TR/webauthn-2/ |
| **CTAP v2.0** | FIDO Alliance 2019-01-30 | https://fidoalliance.org/specs/fido-v2.0-ps-20190130/ |
| **CTAP v2.1** | FIDO Alliance 2021-06-15 | https://fidoalliance.org/specs/fido-v2.1-ps-20210615/ |
| **IBio BRD.md** | v1.0, 2026-04-07 | 项目文档 |
| **IBio URD.md** | v1.0, 2026-04-07 | 项目文档 |
| **IBio HLD.md** | v1.0, 2026-04-06 | 项目文档 |
| **FIDO2 标准需求分析** | v1.0, 2026-04-06 | 项目文档 |
| **ATECC608A 技术规格** | v1.0, 2026-04-06 | 项目文档 |
| **FPM383C 技术规格** | v1.0, 2026-04-06 | 项目文档 |
| **通信模块技术规格** | v1.0, 2026-04-06 | 项目文档 |

---

## 二、总体描述

### 2.1 产品概述

IBio 是一款开源 FIDO2 安全密钥，通过指纹生物识别验证增强身份认证的安全性，支持 USB + BLE 双模通信，实现跨平台无密码认证。

**核心特性**：
- FIDO2/CTAP2.0 标准认证器
- 指纹生物识别本地验证
- USB HID + BLE GATT 双模通信
- ES256 签名算法
- 私钥硬件保护（ATECC608A）
- 低成本 DIY 方案（< $31）

### 2.2 用户特征

| 用户类型 | 特征描述 | 核心需求 |
|----------|----------|----------|
| **个人用户** | 关注账户安全、厌倦密码管理 | 低成本、易使用、安全可靠 |
| **企业用户** | IT 安全管理员、中小企业安全负责人 | 增强安全、降低管理成本 |
| **DIY/开发者** | 技术爱好者、嵌入式开发者 | 开源可定制、学习价值、可复现 |

### 2.3 运行环境

#### 硬件环境

| 组件 | 型号/规格 | 说明 |
|------|-----------|------|
| **主控芯片** | ESP32-S3-WROOM-1-N8R8 | Xtensa LX7 双核 240MHz, 8MB Flash, 8MB PSRAM |
| **指纹传感器** | FPM383C | 电容式指纹传感器，192×192 像素，200 模板容量 |
| **安全芯片** | ATECC608A-MAHDA-T | EAL4+ 安全芯片，16 密钥槽位 |
| **USB 接口** | USB-C 2.0 | USB HID 设备 |
| **BLE 射频** | ESP32-S3 内置 BLE 5.0 | BLE GATT 服务 |

#### 软件环境

| 组件 | 版本 | 说明 |
|------|------|------|
| **开发框架** | ESP-IDF v5.x | Espressif 官方 IoT 开发框架 |
| **实时操作系统** | FreeRTOS | ESP-IDF 内置 |
| **USB 协议栈** | TinyUSB | ESP-IDF 集成 |
| **BLE 协议栈** | Bluedroid/NimBLE | ESP-IDF 内置 |
| **安全库** | CryptoAuthLib | Microchip 官方 ATECC608A 驱动库 |

#### 主机平台兼容性

| 平台 | 支持方式 | 说明 |
|------|----------|------|
| **Windows 10/11** | USB HID | Windows Hello 集成 |
| **macOS** | USB HID | Safari/Chrome WebAuthn |
| **Linux** | USB HID | Chrome/Firefox WebAuthn |
| **iOS 14+** | BLE GATT | Safari WebAuthn |
| **Android 9+** | BLE GATT | Chrome WebAuthn |

### 2.4 设计约束

#### 成本约束

| 约束项 | 目标值 | 说明 |
|--------|--------|------|
| **总硬件成本** | < $31 | 核心硬件 + PCB 成本 |
| **主控芯片成本** | $5-8 | ESP32-S3 模块 |
| **指纹传感器成本** | $10-15 | FPM383C 模块 |
| **安全芯片成本** | $1-2 | ATECC608A |

#### 开源约束

| 约束项 | 说明 |
|--------|------|
| **硬件开源** | PCB 设计文件、元件清单公开 |
| **固件开源** | 源代码公开，使用开源许可证 |
| **文档完整** | 设计文档、用户指南、开发指南公开 |

#### FIDO2 合规约束

| 约束项 | 说明 |
|--------|------|
| **协议合规** | CTAP 2.0/2.1 核心功能 |
| **安全合规** | 私钥不可导出 |
| **非官方认证** | 跳过 FIDO Alliance 官方认证（DIY 项目定位） |

### 2.5 假设与依赖

#### 假设

| 假设项 | 说明 |
|--------|------|
| **平台支持** | 主流平台（Windows/macOS/Linux/iOS/Android）已支持 FIDO2 |
| **用户能力** | 用户具备基本的电脑/手机操作能力 |
| **网络环境** | 用户使用支持 WebAuthn 的网站/服务 |

#### 依赖

| 依赖项 | 说明 |
|--------|------|
| **ESP-IDF 框架** | 依赖 Espressif 官方 ESP-IDF 框架 |
| **CryptoAuthLib 库** | 依赖 Microchip 官方 ATECC608A 驱动库 |
| **FPM383C SDK/协议** | 依赖 FPC 官方提供的 UART 协议文档 |
| **FIDO2 标准** | 依赖 FIDO Alliance 发布的 CTAP 规范 |

---

## 三、功能需求

### 3.1 CTAP 协议功能

#### 3.1.1 MakeCredential（创建凭证）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F01 |
| **需求描述** | 认证器支持 MakeCredential 命令，用于创建新的 FIDO2 凭证 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F02（用户可使用指纹在网站注册新账户） |

**功能详细描述**：

MakeCredential 命令用于在用户首次注册时创建新的 FIDO2 凭证。认证器接收来自客户端的凭证创建请求，验证用户身份后，生成密钥对并存储私钥，返回公钥和凭证 ID。

**输入参数**：

| 参数 | 类型 | 必须 | 说明 |
|------|------|------|------|
| **clientDataHash** | byte[] | 是 | 客户端数据哈希（SHA-256，32 字节） |
| **rp** | PublicKeyCredentialRpEntity | 是 | 依赖方信息（id, name, icon） |
| **user** | PublicKeyCredentialUserEntity | 是 | 用户信息（id, name, displayName, icon） |
| **pubKeyCredParams** | PubKeyCredParams[] | 是 | 支持的算法列表 |
| **excludeList** | PublicKeyCredentialDescriptor[] | 否 | 排除的凭证列表 |
| **options** | Options | 否 | 选项（rk, uv） |
| **pinUvAuthToken** | byte[] | 条件 | PIN/UV 认证令牌 |
| **enterpriseAttestation** | int | 否 | 企业认证标识 |

**输出结果**：

| 字段 | 类型 | 说明 |
|------|------|------|
| **fmt** | string | Attestation 格式标识 |
| **authData** | byte[] | 认证器数据 |
| **attStmt** | attStmt | Attestation 语句 |

**处理流程**：

```
1. 解析请求参数，验证 clientDataHash、rp、user、pubKeyCredParams
2. 检查 excludeList，若存在已注册凭证，返回 CTAP2_ERR_CREDENTIAL_EXCLUDED
3. 验证算法支持，必须支持 ES256（COSE Value: -7）
4. 检查用户验证要求：
   - 若 options.uv=true，要求 PIN 或指纹验证
   - 若 options.up=true，要求用户在场确认
5. 生成密钥对（ATECC608A 硬件生成）
6. 生成凭证 ID（credential ID）
7. 存储 credential：
   - 若 options.rk=true，存储 resident credential
   - 若 options.rk=false，仅返回 credential ID
8. 生成 attestation 签名
9. 返回 MakeCredential 响应
```

**验收标准**：

| 标准 | 验收方法 |
|------|----------|
| MakeCredential 命令执行成功 | WebAuthn Demo (https://webauthn.io/) 注册成功 |
| 凭证存储正确 | 凭证可被后续 GetAssertion 使用 |
| 用户验证正确 | PIN 或指纹验证后才创建凭证 |
| Attestation 签名正确 | 签名可通过 OpenSSL 验证 |

---

#### 3.1.2 GetAssertion（获取认证断言）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F02 |
| **需求描述** | 认证器支持 GetAssertion 命令，用于获取认证签名断言 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F01（用户可使用指纹代替密码登录网站） |

**功能详细描述**：

GetAssertion 命令用于用户登录认证。认证器接收来自客户端的认证请求，查找凭证，验证用户身份后，使用私钥签名挑战，返回签名断言。

**输入参数**：

| 参数 | 类型 | 必须 | 说明 |
|------|------|------|------|
| **rpId** | string | 是 | 依赖方 ID |
| **clientDataHash** | byte[] | 是 | 客户端数据哈希（SHA-256，32 字节） |
| **allowList** | PublicKeyCredentialDescriptor[] | 否 | 允许的凭证列表 |
| **options** | Options | 否 | 选项（up, uv） |
| **pinUvAuthToken** | byte[] | 条件 | PIN/UV 认证令牌 |

**输出结果**：

| 字段 | 类型 | 说明 |
|------|------|------|
| **credential** | PublicKeyCredentialDescriptor | 凭证描述符 |
| **authData** | byte[] | 认证器数据 |
| **signature** | byte[] | 签名（ES256，64 字节） |
| **user** | PublicKeyCredentialUserEntity | 条件 | 用户信息（resident credential） |
| **numberOfCredentials** | int | 条件 | 凭证数量（多个凭证时） |

**处理流程**：

```
1. 解析请求参数，验证 rpId、clientDataHash
2. 计算 rpIdHash = SHA-256(rpId)
3. 查找凭证：
   - 若 allowList 存在，在 allowList 中查找
   - 若 allowList 不存在，查找 resident credential
4. 若未找到凭证，返回 CTAP2_ERR_NO_CREDENTIALS
5. 检查用户验证要求：
   - 若 options.uv=true，要求 PIN 或指纹验证
   - 若 options.up=true，要求用户在场确认
6. 执行用户验证（指纹或 PIN）
7. 构建待签名消息：authData || clientDataHash
8. 使用凭证私钥签名（ATECC608A 硬件签名）
9. 返回 GetAssertion 响应
```

**验收标准**：

| 标准 | 验收方法 |
|------|----------|
| GetAssertion 命令执行成功 | WebAuthn Demo 登录成功 |
| 签名验证正确 | 签名可通过公钥验证 |
| 用户验证正确 | PIN 或指纹验证后才签名 |
| 认证时间 < 2 秒 | 从触摸到签名完成 |

---

#### 3.1.3 GetNextAssertion（获取下一个断言）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F03 |
| **需求描述** | 认证器支持 GetNextAssertion 命令，用于获取多个凭证的认证断言 |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F16（用户可在多个网站使用同一设备） |

**功能详细描述**：

当 GetAssertion 返回多个 resident credential 时，客户端通过 GetNextAssertion 获取后续凭证的断言。

**处理流程**：

```
1. 检查是否有待返回的凭证
2. 获取下一个凭证
3. 构建断言
4. 签名
5. 返回断言
```

---

#### 3.1.4 GetInfo（获取认证器信息）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F04 |
| **需求描述** | 认证器支持 GetInfo 命令，返回认证器能力信息 |
| **优先级** | P0（必须） |
| **URD 映射** | - |

**功能详细描述**：

GetInfo 命令返回认证器支持的功能、算法、选项等信息，供客户端判断认证器能力。

**输出结果**：

| 字段 | 类型 | 说明 |
|------|------|------|
| **versions** | string[] | 支持的协议版本（"FIDO_2_0", "FIDO_2_1_PRE"） |
| **extensions** | string[] | 支持的扩展 |
| **aaguid** | byte[16] | 认证器 AAGUID |
| **options** | map | 支持的选项（rk, up, uv, plat, clientPin） |
| **maxMsgSize** | int | 最大消息大小 |
| **pinUvAuthProtocols** | int[] | 支持的 PIN/UV 认证协议 |
| **transports** | string[] | 支持的传输方式（"usb", "ble"） |
| **algorithms** | map[] | 支持的签名算法 |

**IBio GetInfo 响应示例**：

```json
{
  "versions": ["FIDO_2_0", "FIDO_2_1_PRE"],
  "aaguid": "IBIO-AAGUID-XXXX-XXXX",
  "options": {
    "rk": true,
    "up": true,
    "uv": true,
    "clientPin": true,
    "plat": false
  },
  "pinUvAuthProtocols": [1, 2],
  "transports": ["usb", "ble"],
  "algorithms": [
    {"type": "public-key", "alg": -7}
  ],
  "maxMsgSize": 1200
}
```

---

#### 3.1.5 ClientPIN（PIN 管理）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F05 |
| **需求描述** | 认证器支持 ClientPIN 命令，用于 PIN 设置、验证、修改、删除 |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F12（用户可设置 PIN 作为备用验证方式） |

**功能详细描述**：

ClientPIN 命令提供 PIN 管理功能，包括设置 PIN、验证 PIN、修改 PIN、删除 PIN，以及获取 PIN 重试次数。

**子命令**：

| 子命令 | 值 | 功能 |
|--------|-----|------|
| **getPinRetries** | 0x01 | 获取 PIN 重试次数 |
| **getKeyAgreement** | 0x02 | 获取认证器公钥用于密钥协商 |
| **setPIN** | 0x03 | 设置 PIN |
| **changePIN** | 0x04 | 修改 PIN |
| **getPinToken** | 0x05 | 获取 PIN 令牌 |
| **getPinUvAuthTokenUsingUvWithPermissions** | 0x06 | 使用 UV 获取令牌 |
| **getUvRetries** | 0x07 | 获取 UV 重试次数 |

**PIN 加密流程**：

```
1. 客户端调用 getKeyAgreement 获取认证器公钥
2. 客户端生成临时密钥对 pinUvAuthTokenPubKey
3. 双方计算共享密钥 K = ECDH(pinUvAuthTokenPubKey, ATECC608A_PinKey)
4. 客户端加密 PIN：pinHashEnc = AES(K, SHA256(PIN)[0:16])
5. 客户端发送 setPIN/changePIN 请求
6. 认证器解密并验证 PIN
```

**验收标准**：

| 标准 | 验收方法 |
|------|----------|
| PIN 设置成功 | 设置 PIN 后可验证 |
| PIN 验证正确 | 正确 PIN 验证成功，错误 PIN 验证失败 |
| PIN 加密传输 | PIN 不明文传输 |
| PIN 重试限制 | 连续错误 8 次后锁定 |

---

#### 3.1.6 Reset（重置认证器）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F06 |
| **需求描述** | 认证器支持 Reset 命令，用于重置认证器到出厂状态 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F15（用户可完全重置设备） |

**功能详细描述**：

Reset 命令清除所有凭证、指纹模板、PIN，将认证器重置到出厂状态。

**重置操作**：

| 操作 | 说明 |
|------|------|
| **删除所有凭证** | 清空凭证存储区 |
| **删除所有指纹模板** | 清空 FPM383C 模板库 |
| **删除 PIN** | 清除 PIN 设置 |
| **重置计数器** | 重置 PIN/UV 重试计数器 |
| **生成新 Attestation 密钥** | 可选：重新生成 attestation 密钥 |

**重置限制**：

| 限制 | 说明 |
|------|------|
| **时间限制** | 仅在认证器启动后 10 秒内可执行 Reset |
| **用户确认** | 需要用户触摸确认 |
| **不可恢复** | Reset 后数据不可恢复 |

---

#### 3.1.7 CredentialManagement（凭证管理）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F07 |
| **需求描述** | 认证器支持 CredentialManagement 命令，用于枚举和删除凭证 |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F13、UR-F14（用户可查看和删除凭证） |

**功能详细描述**：

CredentialManagement 命令提供凭证管理功能，包括枚举凭证、删除凭证。

**子命令**：

| 子命令 | 值 | 功能 |
|--------|-----|------|
| **getCredsMetadata** | 0x01 | 获取凭证数量 |
| **enumerateRPsBegin** | 0x02 | 开始枚举 RP |
| **enumerateRPsGetNextRP** | 0x03 | 获取下一个 RP |
| **enumerateCredentialsBegin** | 0x04 | 开始枚举凭证 |
| **enumerateCredentialsGetNextCredential** | 0x05 | 获取下一个凭证 |
| **deleteCredential** | 0x06 | 删除凭证 |
| **updateUserInformation** | 0x07 | 更新用户信息 |

---

#### 3.1.8 Selection（选择认证器）

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F08 |
| **需求描述** | 认证器支持 Selection 命令，用于在多认证器场景中选择此认证器 |
| **优先级** | P1（推荐） |
| **URD 映射** | - |

**功能详细描述**：

Selection 命令触发认证器执行用户可见的反馈（如 LED 闪烁），便于用户在多个认证器中选择。

---

### 3.2 生物识别功能

#### 3.2.1 指纹采集

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F09 |
| **需求描述** | 认证器支持指纹图像采集功能 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F05（用户可在 1 秒内完成指纹验证） |

**功能详细描述**：

指纹采集功能通过 FPM383C 指纹传感器采集用户指纹图像。

**采集规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **采集方式** | 按压式 | 用户手指按压传感器表面 |
| **采集时间** | < 500 ms | 单次采集时间 |
| **图像分辨率** | 192×192 像素 | FPM383C 规格 |
| **采集面积** | 10×10 mm | FPM383C 规格 |

**采集流程**：

```
1. 触发采集命令（自动或手动触发）
2. 等待手指触摸（TOUCH 引脚检测或命令触发）
3. 采集指纹图像
4. 评估图像质量
5. 返回采集结果
```

---

#### 3.2.2 指纹比对

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F10 |
| **需求描述** | 认证器支持指纹比对功能，验证用户身份 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F05、UR-NF06、UR-NF07 |

**功能详细描述**：

指纹比对功能将采集的指纹与存储的模板进行比对，验证用户身份。

**比对模式**：

| 模式 | 说明 | 应用场景 |
|------|------|----------|
| **1:1 比对** | 与指定模板 ID 比对 | 已知用户验证 |
| **1:N 搜索** | 搜索所有模板 | 未知用户识别 |

**比对性能**：

| 参数 | 规格 | FIDO2 要求 |
|------|------|------------|
| **误识率（FAR）** | < 0.001% | < 0.001% |
| **拒识率（FRR）** | < 2% | < 3% |
| **比对时间** | < 1 秒 | < 1 秒 |

**比对流程**：

```
1. 采集指纹图像
2. 提取指纹特征
3. 与模板库比对
4. 计算匹配分数
5. 判断匹配结果
6. 返回比对结果
```

---

#### 3.2.3 指纹注册

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F11 |
| **需求描述** | 认证器支持指纹注册功能，生成并存储指纹模板 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F10（用户可注册多个指纹） |

**功能详细描述**：

指纹注册功能通过多次采集生成高质量的指纹模板，并存储到 FPM383C 模块内部。

**注册规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **采集次数** | 3-5 次 | 提高模板质量 |
| **注册时间** | < 30 秒 | 完成注册 |
| **模板容量** | 200 个 | FPM383C 规格 |
| **模板 ID 范围** | 0-199 | 模板索引 |

**注册流程**：

```
1. 开始注册流程
2. 采集第 1 次指纹，评估质量
3. 采集第 2-5 次指纹，合并生成模板
4. 存储模板到指定 ID
5. 返回注册结果
```

---

#### 3.2.4 指纹模板管理

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F12 |
| **需求描述** | 认证器支持指纹模板管理功能，包括删除和枚举 |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F11（用户可删除已注册指纹） |

**功能详细描述**：

指纹模板管理功能提供删除、枚举指纹模板的能力。

**管理功能**：

| 功能 | 说明 |
|------|------|
| **删除模板** | 删除指定 ID 的指纹模板 |
| **清空模板库** | 删除所有指纹模板 |
| **枚举模板** | 返回已注册模板列表 |
| **查询模板数量** | 返回已注册模板数量 |

---

### 3.3 通信功能

#### 3.3.1 USB HID 通信

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F13 |
| **需求描述** | 认证器支持 USB HID 通信，实现 CTAP over HID |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F03、UR-F08 |

**功能详细描述**：

USB HID 通信功能实现 CTAP 协议 over USB HID，支持桌面平台（Windows/macOS/Linux）。

**USB HID 规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **设备类型** | USB HID | 设备类 0x03 |
| **Usage Page** | 0xF1D0 | FIDO Alliance Usage Page |
| **Usage** | 0x01 | U2F Authenticator Device |
| **端点类型** | Interrupt IN/OUT | HID 标准端点 |
| **数据包大小** | 64 字节 | HID 报告大小 |
| **传输间隔** | 1 ms | HID 中断间隔 |
| **响应超时** | 30 秒 | CTAP 响应超时 |

**HID 命令**：

| 命令 | 值 | 功能 |
|------|-----|------|
| **INIT** | 0x80 | 初始化通道 |
| **PING** | 0x81 | Ping 测试 |
| **MSG** | 0x83 | 发送 CTAP 消息 |
| **CANCEL** | 0x85 | 取消操作 |
| **ERROR** | 0x86 | 错误响应 |
| **KEEPALIVE** | 0x87 | 保活响应 |
| **WINK** | 0x88 | Wink（视觉反馈） |
| **LOCK** | 0x84 | 锁定通道 |

---

#### 3.3.2 BLE GATT 通信

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F14 |
| **需求描述** | 认证器支持 BLE GATT 通信，实现 CTAP over BLE |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F04、UR-F09 |

**功能详细描述**：

BLE GATT 通信功能实现 CTAP 协议 over BLE GATT，支持移动平台（iOS/Android）。

**BLE GATT 规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **BLE 版本** | BLE 5.0 | ESP32-S3 内置 |
| **设备角色** | Peripheral | 外设模式 |
| **FIDO Service UUID** | 0xFFFD | FIDO GATT 服务 |
| **Control Point UUID** | 0xFFFDC1 | 发送 CTAP 命令 |
| **Status UUID** | 0xFFFDC2 | 返回 CTAP 状态 |
| **连接间隔** | 15-30 ms | 推荐值 |
| **MTU** | 64+ 字节 | 推荐 MTU |

**GATT 特征值**：

| 特征值 | UUID | 属性 | 说明 |
|--------|------|------|------|
| **FIDO Control Point** | 0xFFFDC1 | Write | 发送 CTAP 命令 |
| **FIDO Status** | 0xFFFDC2 | Read, Notify | 返回 CTAP 状态 |
| **FIDO Service Revision** | 0xFFFDC3 | Read | 返回服务版本 |
| **FIDO Service Revision Bitfield** | 0xFFFDC4 | Read, Write | 版本协商 |

---

### 3.4 安全功能

#### 3.4.1 密钥生成与存储

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F15 |
| **需求描述** | 认证器支持密钥对生成和安全存储，私钥不可导出 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-NF12 |

**功能详细描述**：

密钥生成功能由 ATECC608A 安全芯片硬件生成密钥对，私钥存储在芯片内部，永不离开芯片。

**密钥生成规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **算法** | ES256 (ECDSA with SHA-256) | FIDO2 必须 |
| **曲线** | NIST P-256 (secp256r1) | 标准 ECC 曲线 |
| **生成方式** | 硬件 TRNG | ATECC608A 内部生成 |
| **私钥存储** | ATECC608A 槽位 | 私钥不可导出 |
| **公钥格式** | Uncompressed (65 bytes) | 标准格式 |

**ATECC608A 密钥槽位分配**：

| 槽位 | 用途 | 说明 |
|------|------|------|
| **Slot 0-9** | 凭证密钥 | 存储用户凭证私钥 |
| **Slot 14** | Attestation 密钥 | 认证器认证密钥 |
| **Slot 15** | PIN 密钥 | PIN 加密密钥 |

---

#### 3.4.2 ES256 签名

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F16 |
| **需求描述** | 认证器支持 ES256 签名算法 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-NF12 |

**功能详细描述**：

ES256 签名功能由 ATECC608A 硬件执行，私钥永不离开芯片。

**签名规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **算法** | ECDSA with SHA-256 | FIDO2 必须 |
| **输入** | 32 字节消息哈希 | SHA-256 输出 |
| **输出** | 64 字节签名 (r + s) | Raw signature |
| **签名时间** | < 100 ms | ATECC608A 规格 |

---

#### 3.4.3 ECDH 密钥协商

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F17 |
| **需求描述** | 认证器支持 ECDH 密钥协商，用于 PIN 加密通道 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F12 |

**功能详细描述**：

ECDH 密钥协商功能用于 ClientPIN 命令建立加密通道，保护 PIN 传输安全。

**ECDH 规格**：

| 参数 | 规格 | 说明 |
|------|------|------|
| **算法** | ECDH with P-256 | 标准 ECDH |
| **输入** | 对方公钥 (65 bytes) | 临时公钥 |
| **输出** | 共享密钥 (32 bytes) | 用于 AES 加密 |
| **操作时间** | < 100 ms | ATECC608A 规格 |

---

#### 3.4.4 用户验证

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F18 |
| **需求描述** | 认证器支持指纹和 PIN 用户验证 |
| **优先级** | P0（必须） |
| **URD 映射** | UR-F05、UR-F12 |

**功能详细描述**：

用户验证功能提供指纹和 PIN 两种验证方式，优先使用指纹，PIN 作为备用。

**验证方式**：

| 方式 | 优先级 | 说明 |
|------|--------|------|
| **指纹验证** | 首选 | 本地比对，< 1 秒 |
| **PIN 验证** | 备用 | 4-8 位数字，本地验证 |

**UV 标志**：

| UV 值 | 说明 |
|-------|------|
| **UV=1** | 用户验证成功（指纹或 PIN） |
| **UV=0** | 用户验证未执行 |
| **UV Blocked** | 连续失败锁定 |

---

### 3.5 凭证管理功能

#### 3.5.1 凭证存储

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F19 |
| **需求描述** | 认证器支持凭证存储，包括 resident credential |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F16 |

**功能详细描述**：

凭证存储功能将 FIDO2 凭证存储在认证器内部 Flash 中，支持 resident credential（凭证存储在认证器，无需服务器存储 credential ID）。

**存储容量**：

| 项目 | 规格 | 说明 |
|------|------|------|
| **最大凭证数量** | 10+ | 受密钥槽位和 Flash 空间限制 |
| **凭证大小** | 约 200-400 bytes | 包含私钥槽位映射、RP ID、用户信息等 |
| **存储位置** | ESP32-S3 内部 Flash | Flash 分区存储 |

---

#### 3.5.2 凭证检索

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F20 |
| **需求描述** | 认证器支持凭证检索，根据 rpId 或 credential ID 查找凭证 |
| **优先级** | P0（必须） |
| **URD 映射** | - |

**功能详细描述**：

凭证检索功能根据 rpId 或 credential ID 查找存储的凭证。

**检索方式**：

| 方式 | 说明 |
|------|------|
| **按 rpId 检索** | 遍历所有 resident credential，匹配 rpIdHash |
| **按 credential ID 检索** | 根据 credential ID 直接定位凭证 |

---

#### 3.5.3 凭证删除

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F21 |
| **需求描述** | 认证器支持删除指定凭证 |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F14 |

**功能详细描述**：

凭证删除功能删除指定 credential ID 的凭证，释放存储空间和密钥槽位。

---

#### 3.5.4 凭证枚举

| 项目 | 说明 |
|------|------|
| **需求编号** | SR-F22 |
| **需求描述** | 认证器支持枚举所有凭证 |
| **优先级** | P1（推荐） |
| **URD 映射** | UR-F13 |

**功能详细描述**：

凭证枚举功能返回认证器中存储的所有凭证列表，供用户查看和管理。

---

## 四、接口需求

### 4.1 硬件接口

#### 4.1.1 ESP32-S3 接口

| 接口 | 引脚 | 用途 | 说明 |
|------|------|------|------|
| **UART0** | GPIO16 (TX), GPIO17 (RX) | 指纹传感器通信 | FPM383C UART 接口 |
| **I2C0** | GPIO21 (SDA), GPIO22 (SCL) | 安全芯片通信 | ATECC608A I2C 接口 |
| **USB OTG** | GPIO20 (D+), GPIO19 (D-) | USB 通信 | 内置 USB PHY |
| **GPIO** | GPIO18, GPIO19 | 控制信号 | EN, TOUCH 信号 |
| **LED** | GPIO25, GPIO26 | 状态指示 | LED 输出 |

#### 4.1.2 FPM383C 指纹传感器接口

| ESP32-S3 引脚 | FPM383C 引脚 | 功能 | 说明 |
|---------------|--------------|------|------|
| GPIO16 (TX) | RX | UART 发送 | MCU → 模块 |
| GPIO17 (RX) | TX | UART 接收 | 模块 → MCU |
| GPIO18 | EN | 使能信号 | 模块使能控制 |
| GPIO19 | TOUCH | 触摸检测 | 触摸中断 |
| 3.3V | VCC | 电源 | 3.3V 供电 |
| GND | GND | 地 | 共地 |

**UART 参数**：

| 参数 | 规格 |
|------|------|
| 波特率 | 57600 bps（推荐） |
| 数据位 | 8 bit |
| 校验位 | 无 |
| 停止位 | 1 bit |
| 流控 | 无 |

#### 4.1.3 ATECC608A 安全芯片接口

| ESP32-S3 引脚 | ATECC608A 引脚 | 功能 | 说明 |
|---------------|----------------|------|------|
| GPIO21 | SDA | I2C 数据线 | 双向数据传输 |
| GPIO22 | SCL | I2C 时钟线 | 时钟信号输出 |
| GPIO23 | RST | 复位信号 | 芯片复位（可选） |
| 3.3V | VCC | 电源 | 3.3V 供电 |
| GND | GND | 地 | 共地 |

**I2C 参数**：

| 参数 | 规格 |
|------|------|
| I2C 地址 | 0xC0 (7-bit: 0x60) |
| 时钟频率 | 400 kHz（推荐） |
| 上拉电阻 | 4.7kΩ |

#### 4.1.4 USB-C 接口

| USB-C 引脚 | ESP32-S3 引脚 | 功能 | 说明 |
|------------|---------------|------|------|
| D+ | GPIO20 (USB_DP) | USB 数据正 | USB 2.0 |
| D- | GPIO19 (USB_DN) | USB 数据负 | USB 2.0 |
| VBUS | 5V | 电源 | USB 供电 |
| GND | GND | 地 | USB 地 |
| CC1 | GPIO25 | 配置通道 | USB-C 配置（可选） |
| CC2 | GPIO26 | 配置通道 | USB-C 配置（可选） |

### 4.2 软件接口

#### 4.2.1 HAL 层接口

**GPIO 驱动接口**：

```c
// GPIO 初始化
int gpio_init(uint8_t pin, gpio_mode_t mode);

// GPIO 读取
int gpio_read(uint8_t pin);

// GPIO 写入
int gpio_write(uint8_t pin, uint8_t value);

// GPIO 中断设置
int gpio_set_interrupt(uint8_t pin, void (*callback)(void));
```

**UART 驱动接口**：

```c
// UART 配置结构
typedef struct {
    uint32_t baudrate;
    uint8_t data_bits;
    uint8_t parity;
    uint8_t stop_bits;
} uart_config_t;

// UART 初始化
int uart_init(uint8_t port, uart_config_t *config);

// UART 读取
int uart_read(uint8_t port, uint8_t *data, uint32_t len, uint32_t timeout);

// UART 写入
int uart_write(uint8_t port, uint8_t *data, uint32_t len);
```

**I2C 驱动接口**：

```c
// I2C 配置结构
typedef struct {
    uint8_t address;
    uint32_t frequency;
} i2c_config_t;

// I2C 初始化
int i2c_init(uint8_t bus, i2c_config_t *config);

// I2C 读取
int i2c_read(uint8_t bus, uint8_t addr, uint8_t *data, uint32_t len);

// I2C 写入
int i2c_write(uint8_t bus, uint8_t addr, uint8_t *data, uint32_t len);
```

#### 4.2.2 安全模块接口

**密钥管理接口**：

```c
// 密钥句柄结构
typedef struct {
    uint8_t key_id;
    uint8_t key_type;  // ES256, ECDH
    uint8_t slot_index;
} key_handle_t;

// 密钥生成
int key_generate(uint8_t key_type, key_handle_t *handle);

// 获取公钥
int key_get_public_key(key_handle_t *handle, uint8_t *pubkey, uint32_t *len);

// 签名
int key_sign(key_handle_t *handle, uint8_t *data, uint32_t len, 
             uint8_t *sig, uint32_t *sig_len);

// ECDH 密钥协商
int key_ecdh(key_handle_t *handle, uint8_t *peer_pubkey, 
             uint8_t *shared_secret);

// 密钥删除
int key_delete(key_handle_t *handle);
```

**加密引擎接口**：

```c
// SHA-256 哈希
int crypto_sha256(uint8_t *data, uint32_t len, uint8_t *hash);

// 随机数生成
int crypto_random(uint8_t *random, uint32_t len);
```

**用户验证接口**：

```c
// 用户验证方式
typedef enum {
    UV_NONE,        // 无验证
    UV_PIN,         // PIN 验证
    UV_FINGERPRINT  // 指纹验证
} uv_method_t;

// 请求用户验证
int uv_request(uv_method_t method);

// 获取验证结果
int uv_get_result(uint8_t *success);

// 设置 PIN
int uv_set_pin(uint8_t *pin, uint32_t len);

// 验证 PIN
int uv_verify_pin(uint8_t *pin, uint32_t len);

// 获取 UV 重试次数
int uv_get_retries(uint8_t *retries);
```

#### 4.2.3 指纹模块接口

```c
// 指纹采集
int fp_capture_start(void);
int fp_capture_wait(uint8_t *image, uint32_t *len, uint32_t timeout);
int fp_capture_stop(void);

// 指纹比对
int fp_match(uint8_t template_id, uint8_t *match_score);
int fp_search(uint8_t *template_id, uint8_t *match_score);

// 模板管理
int fp_enroll_start(void);
int fp_enroll_add_image(uint8_t *image, uint32_t len, uint8_t *quality);
int fp_enroll_finish(uint8_t *template_id);
int fp_template_delete(uint8_t template_id);
int fp_template_list(uint8_t *list, uint32_t *count);
int fp_template_count(uint32_t *count);
int fp_template_clear_all(void);
```

#### 4.2.4 CTAP 协议栈接口

```c
// CTAP 请求结构
typedef struct {
    uint8_t command;
    uint8_t *params;
    uint32_t params_len;
} ctap_request_t;

// CTAP 响应结构
typedef struct {
    uint8_t status;
    uint8_t *data;
    uint32_t data_len;
} ctap_response_t;

// CTAP 请求处理
int ctap_process_request(ctap_request_t *req, ctap_response_t *resp);
```

**CTAP 命令定义**：

```c
#define CTAP_CMD_MAKE_CREDENTIAL       0x01
#define CTAP_CMD_GET_ASSERTION         0x02
#define CTAP_CMD_GET_INFO              0x04
#define CTAP_CMD_CLIENT_PIN            0x06
#define CTAP_CMD_RESET                 0x07
#define CTAP_CMD_GET_NEXT_ASSERTION    0x08
#define CTAP_CMD_CREDENTIAL_MANAGEMENT 0x0A
#define CTAP_CMD_SELECTION             0x0B
```

**CTAP 状态码定义**：

```c
#define CTAP_STATUS_OK                 0x00
#define CTAP_STATUS_ERR_INVALID_CMD    0x01
#define CTAP_STATUS_ERR_INVALID_PARAM  0x02
#define CTAP_STATUS_ERR_INVALID_LENGTH 0x03
#define CTAP_STATUS_ERR_PIN_INVALID    0x04
#define CTAP_STATUS_ERR_PIN_BLOCKED    0x05
#define CTAP_STATUS_ERR_UV_BLOCKED     0x06
#define CTAP_STATUS_ERR_NO_CREDENTIALS 0x07
#define CTAP_STATUS_ERR_CREDENTIAL_EXCLUDED 0x19
```

### 4.3 通信接口

#### 4.3.1 USB HID 接口

**USB HID 描述符**：

| 字段 | 值 | 说明 |
|------|-----|------|
| **Usage Page** | 0xF1D0 | FIDO Alliance Usage Page |
| **Usage** | 0x01 | U2F Authenticator Device |
| **Report Size** | 8 bits | 1 字节 |
| **Report Count** | 64 | 64 字节报告 |
| **Input/Output** | Data, Var, Abs | 数据报告 |

**USB HID 命令接口**：

```c
// USB HID 初始化
int usb_hid_init(void);

// USB HID 发送
int usb_hid_send(uint8_t *data, uint32_t len);

// USB HID 接收
int usb_hid_recv(uint8_t *data, uint32_t *len, uint32_t timeout);

// USB HID Wink（LED 闪烁）
int usb_hid_wink(void);
```

#### 4.3.2 BLE GATT 接口

**FIDO GATT 服务定义**：

| 服务/特征 | UUID | 属性 | 说明 |
|-----------|------|------|------|
| **FIDO Service** | 0xFFFD | - | FIDO GATT 服务 |
| **Control Point** | 0xFFFDC1 | Write | 发送 CTAP 命令 |
| **Status** | 0xFFFDC2 | Read, Notify | 返回 CTAP 状态 |
| **Service Revision** | 0xFFFDC3 | Read | 服务版本 |

**BLE GATT 接口**：

```c
// BLE GATT 初始化
int ble_gatt_init(void);

// BLE 广播开始
int ble_adv_start(void);

// BLE 广播停止
int ble_adv_stop(void);

// BLE 连接参数更新
int ble_update_conn_params(void);

// BLE 发送状态通知
int ble_send_status(uint8_t status);
```

### 4.4 用户接口

#### 4.4.1 物理触摸接口

| 项目 | 规格 |
|------|------|
| **触发方式** | 手指按压指纹传感器 |
| **触摸检测** | GPIO 中断或 FPM383C TOUCH 引脚 |
| **响应时间** | < 100 ms LED 亮起 |
| **采集时间** | < 500 ms |

#### 4.4.2 LED 状态指示接口

| LED 状态 | 颜色 | 含义 |
|----------|------|------|
| **熄灭** | 无 | 设备空闲 |
| **绿灯闪烁** | 绿色 | 等待用户触摸 |
| **绿灯常亮** | 绿色 | 操作成功 |
| **红灯闪烁** | 红色 | 操作失败 |
| **红灯常亮** | 红色 | 设备错误/锁定 |

**LED 接口**：

```c
// LED 初始化
int led_init(void);

// LED 设置状态
int led_set_state(led_state_t state);

// LED 熄灭
int led_off(void);
```

---

## 五、非功能需求

### 5.1 性能需求

| 需求编号 | 需求描述 | 目标值 | 验收方法 |
|----------|----------|--------|----------|
| **SR-NF01** | 完整认证时间（从触摸到完成） | < 2 秒 | 性能测试 |
| **SR-NF02** | 指纹验证时间 | < 1 秒 | 模块测试 |
| **SR-NF03** | 指纹采集时间 | < 500 ms | 模块测试 |
| **SR-NF04** | ES256 签名时间 | < 100 ms | 模块测试 |
| **SR-NF05** | USB HID 响应时间 | < 30 秒 | CTAP 规范 |
| **SR-NF06** | BLE GATT 响应时间 | < 30 秒 | CTAP 规范 |
| **SR-NF07** | LED 响应时间（触摸后亮起） | < 100 ms | 功能测试 |
| **SR-NF08** | BLE 连接间隔 | 15-30 ms | BLE 规范 |
| **SR-NF09** | 凭证存储容量 | ≥ 10 个 | 存储测试 |
| **SR-NF10** | 指纹模板容量 | ≥ 5 个（推荐 200） | FPM383C 规格 |

### 5.2 安全需求

| 需求编号 | 需求描述 | 要求级别 | 验收方法 |
|----------|----------|----------|----------|
| **SR-NF11** | 私钥不可导出 | 必须 | 安全审计 |
| **SR-NF12** | 私钥不可读取 | 必须 | 安全审计 |
| **SR-NF13** | 密钥隔离存储 | 必须 | 设计验证 |
| **SR-NF14** | 指纹本地比对 | 必须 | 功能测试 |
| **SR-NF15** | 指纹模板不可导出 | 必须 | FPM383C 规格 |
| **SR-NF16** | PIN 加密传输 | 必须 | 协议测试 |
| **SR-NF17** | 防钓鱼（域名验证） | 必须 | FIDO2 机制 |
| **SR-NF18** | 固件防篡改 | 推荐 | Secure Boot |
| **SR-NF19** | 指纹误识率（FAR） | < 0.001% | FPM383C 规格 |
| **SR-NF20** | 指纹拒识率（FRR） | < 2% | FPM383C 规格 |
| **SR-NF21** | PIN 重试限制 | 8 次 | 功能测试 |
| **SR-NF22** | UV 重试限制 | 3 次 | 功能测试 |

### 5.3 可靠性需求

| 需求编号 | 需求描述 | 目标值 | 验收方法 |
|----------|----------|--------|----------|
| **SR-NF23** | 认证成功率 | > 99% | 功能测试 |
| **SR-NF24** | USB 连接稳定性 | 无频繁断连 | 兼容性测试 |
| **SR-NF25** | BLE 连接稳定性 | 连接成功率 > 95% | 兼容性测试 |
| **SR-NF26** | 设备使用寿命 | > 5 年 | 可靠性评估 |
| **SR-NF27** | 指纹传感器按压寿命 | > 100 万次 | FPM383C 规格 |

### 5.4 可维护性需求

| 需求编号 | 需求描述 | 验收方法 |
|----------|----------|----------|
| **SR-NF28** | 支持固件 OTA 更新 | OTA 功能可用 |
| **SR-NF29** | 设备状态可诊断 | 配置工具可查看状态 |
| **SR-NF30** | 故障可简单排查 | 用户指南提供排查步骤 |
| **SR-NF31** | 代码注释完整 | 代码审查 |

### 5.5 可移植性需求

| 需求编号 | 需求描述 | 验收方法 |
|----------|----------|----------|
| **SR-NF32** | 兼容 Windows 10/11 | Windows Hello 测试 |
| **SR-NF33** | 兼容 macOS | Safari/Chrome 测试 |
| **SR-NF34** | 兼容 Linux | Chrome/Firefox 测试 |
| **SR-NF35** | 兼容 iOS 14+ | Safari BLE 测试 |
| **SR-NF36** | 兼容 Android 9+ | Chrome BLE 测试 |
| **SR-NF37** | 兼容主流浏览器 | Chrome/Safari/Firefox/Edge 测试 |

---

## 六、业务规则

### 6.1 FIDO2 标准合规规则

| 规则编号 | 规则描述 |
|----------|----------|
| **BR-01** | 必须支持 CTAP 2.0 核心命令（MakeCredential、GetAssertion、GetInfo、Reset） |
| **BR-02** | 必须支持 ES256 签名算法（COSE Value: -7） |
| **BR-03** | 必须支持用户验证（UV），提供指纹或 PIN 验证 |
| **BR-04** | 必须支持 resident credential（rk=true） |
| **BR-05** | 必须返回正确的认证器信息（GetInfo） |
| **BR-06** | 必须支持域名验证（rpIdHash），防止钓鱼攻击 |
| **BR-07** | 必须支持 ClientPIN 协议 v1 和 v2 |
| **BR-08** | 应支持 CTAP 2.1 增强特性（可选） |

### 6.2 密钥管理规则

| 规则编号 | 规则描述 |
|----------|----------|
| **BR-09** | 所有密钥对由 ATECC608A 硬件生成，私钥永不离开芯片 |
| **BR-10** | 每个凭证使用独立的密钥槽位，密钥隔离存储 |
| **BR-11** | 密钥槽位分配遵循 ATECC608A 配置方案 |
| **BR-12** | Reset 命令必须清除所有凭证密钥 |
| **BR-13** | Attestation 密钥用于证明认证器真实性 |

### 6.3 用户验证规则

| 规则编号 | 规则描述 |
|----------|----------|
| **BR-14** | 指纹验证优先，PIN 作为备用验证方式 |
| **BR-15** | 指纹验证在 FPM383C 本地完成，模板不可导出 |
| **BR-16** | PIN 验证通过 ECDH 加密通道传输，不明文传输 |
| **BR-17** | PIN 连续错误 8 次后锁定，Reset 后解锁 |
| **BR-18** | UV（指纹）连续失败 3 次后返回 UV Blocked |
| **BR-19** | 用户验证成功后设置 UV=1 标志 |

### 6.4 凭证管理规则

| 规则编号 | 规则描述 |
|----------|----------|
| **BR-20** | MakeCredential 时检查 excludeList，防止重复注册 |
| **BR-21** | GetAssertion 时检查 allowList 或查找 resident credential |
| **BR-22** | 凭证删除时释放密钥槽位和存储空间 |
| **BR-23** | CredentialManagement 需要用户验证（PIN） |
| **BR-24** | 凭证存储容量不足时返回错误 |

### 6.5 安全保护规则

| 规则编号 | 规则描述 |
|----------|----------|
| **BR-25** | 私钥不可导出、不可读取，只能用于签名 |
| **BR-26** | 所有敏感操作（签名、密钥生成）在 ATECC608A 内部完成 |
| **BR-27** | PIN 使用 ECDH + AES 加密传输 |
| **BR-28** | Reset 命令仅在设备启动后 10 秒内可执行 |
| **BR-29** | 固件应支持 Secure Boot 验证 |
| **BR-30** | 连续失败锁定保护，防止暴力破解 |

---

## 七、数据需求

### 7.1 凭证数据结构

**Credential 数据结构**：

```c
typedef struct {
    // 凭证 ID（credential ID），用于查找凭证
    uint8_t credential_id[32];
    
    // 密钥槽位索引（ATECC608A Slot 0-9）
    uint8_t key_slot;
    
    // RP ID 哈希（SHA-256，32 字节）
    uint8_t rp_id_hash[32];
    
    // 用户 ID（user.id，最多 64 字节）
    uint8_t user_id[64];
    uint8_t user_id_len;
    
    // 用户名称（user.name，可选）
    char user_name[64];
    
    // 用户显示名称（user.displayName，可选）
    char user_display_name[64];
    
    // 凭证创建时间
    uint32_t create_time;
    
    // 签名计数器
    uint32_t sign_count;
    
} credential_t;
```

**Credential ID 格式**：

| 字段 | 长度 | 说明 |
|------|------|------|
| **Random Nonce** | 16 bytes | 随机数，防止枚举 |
| **Key Slot Index** | 1 byte | 密钥槽位索引 |
| **Reserved** | 7 bytes | 保留字段 |
| **HMAC** | 8 bytes | HMAC 校验，防止伪造 |
| **总计** | 32 bytes | Credential ID 长度 |

### 7.2 指纹模板数据结构

**指纹模板存储在 FPM383C 模块内部，IBio 维护模板映射表**：

```c
typedef struct {
    // 指纹模板 ID（FPM383C 模板 ID，0-199）
    uint8_t template_id;
    
    // 模板友好名称
    char friendly_name[32];
    
    // 注册时间
    uint32_t create_time;
    
    // 是否已激活
    uint8_t active;
    
} fingerprint_template_t;
```

### 7.3 配置数据结构

**认证器配置**：

```c
typedef struct {
    // AAGUID（认证器 AAGUID，16 字节）
    uint8_t aaguid[16];
    
    // PIN 是否已设置
    uint8_t pin_set;
    
    // PIN 重试次数
    uint8_t pin_retries;
    
    // UV 重试次数
    uint8_t uv_retries;
    
    // 凭证数量
    uint8_t credential_count;
    
    // 指纹模板数量
    uint8_t fingerprint_count;
    
    // 设备序列号
    char serial_number[16];
    
    // 固件版本
    uint8_t firmware_major;
    uint8_t firmware_minor;
    uint8_t firmware_patch;
    
} authenticator_config_t;
```

### 7.4 密钥槽位分配

| 槽位 | 用途 | 密钥类型 | 权限 |
|------|------|----------|------|
| **Slot 0-9** | 凭证密钥 | ES256 私钥 | 不可读、可签名 |
| **Slot 14** | Attestation 密钥 | ES256 私钥 | 不可读、可签名 |
| **Slot 15** | PIN 密钥 | ECDH 密钥 | 不可读、可 ECDH |
| **Slot 10** | 临时 ECDH | ECDH 密钥 | 可读写、可 ECDH |

---

## 八、验收标准

### 8.1 功能验收标准

| 验收项 | 验收标准 | 验收方法 |
|--------|----------|----------|
| **MakeCredential** | WebAuthn Demo 注册成功 | https://webauthn.io/ |
| **GetAssertion** | WebAuthn Demo 登录成功 | https://webauthn.io/ |
| **指纹验证** | 指纹验证成功后才能执行 MakeCredential/GetAssertion | 功能测试 |
| **PIN 设置** | PIN 设置后可验证 | 功能测试 |
| **Reset** | Reset 后所有数据清空 | 功能测试 |
| **CredentialManagement** | 可枚举和删除凭证 | 功能测试 |
| **USB HID** | Windows/Mac/Linux USB 认证成功 | 兼容性测试 |
| **BLE GATT** | iOS/Android BLE 认证成功 | 兼容性测试 |

### 8.2 性能验收标准

| 验收项 | 目标值 | 验收方法 |
|--------|--------|----------|
| **认证时间** | < 2 秒 | 性能测试计时 |
| **指纹验证时间** | < 1 秒 | 模块测试 |
| **签名时间** | < 100 ms | 模块测试 |
| **USB 响应时间** | < 30 秒 | CTAP 规范 |
| **BLE 响应时间** | < 30 秒 | CTAP 规范 |

### 8.3 安全验收标准

| 验收项 | 验收标准 | 验收方法 |
|--------|----------|----------|
| **私钥保护** | 私钥不可导出、不可读取 | 安全审计 |
| **指纹安全** | 指纹模板不可导出 | FPM383C 规格 |
| **PIN 安全** | PIN 加密传输，不明文传输 | 协议分析 |
| **防钓鱼** | 不在错误域名工作 | FIDO2 测试 |
| **误识率（FAR）** | < 0.001% | FPM383C 规格 |
| **拒识率（FRR）** | < 2% | FPM383C 规格 |
| **锁定保护** | 连续错误后锁定 | 功能测试 |

### 8.4 兼容性验收标准

| 平台 | 验收标准 | 验收方法 |
|------|----------|----------|
| **Windows 10/11** | Windows Hello 认证成功 | 真机测试 |
| **macOS** | Safari/Chrome 认证成功 | 真机测试 |
| **Linux** | Chrome/Firefox 认证成功 | 真机测试 |
| **iOS 14+** | Safari BLE 认证成功 | 真机测试 |
| **Android 9+** | Chrome BLE 认证成功 | 真机测试 |
| **Chrome** | WebAuthn API 正常工作 | 浏览器测试 |
| **Safari** | WebAuthn API 正常工作 | 浏览器测试 |
| **Firefox** | WebAuthn API 正常工作 | 浏览器测试 |
| **Edge** | WebAuthn API 正常工作 | 浏览器测试 |

---

## 九、附录

### 9.1 CTAP 命令详细规格

| 命令 | 命令字节 | 必须 | 说明 |
|------|----------|------|------|
| **MakeCredential** | 0x01 | 必须 | 创建凭证 |
| **GetAssertion** | 0x02 | 必须 | 获取断言 |
| **GetInfo** | 0x04 | 必须 | 获取信息 |
| **ClientPIN** | 0x06 | 推荐 | PIN 管理 |
| **Reset** | 0x07 | 必须 | 重置认证器 |
| **GetNextAssertion** | 0x08 | 推荐 | 获取下一个断言 |
| **CredentialManagement** | 0x0A | 推荐 | 凭证管理 |
| **Selection** | 0x0B | 推荐 | 选择认证器 |

### 9.2 CBOR 编码规范

CTAP 使用 CBOR（Concise Binary Object Representation）编码。关键数据类型：

| 数据类型 | CBOR 类型 | 说明 |
|----------|-----------|------|
| **整数** | 0x00-0x17 (unsigned) | 正整数 |
| **字节串** | 0x40-0x5F | 二进制数据 |
| **文本串** | 0x60-0x7F | UTF-8 文本 |
| **数组** | 0x80-0x9F | 数组 |
| **映射** | 0xA0-0xBF | 键值对 |

### 9.3 状态码定义

| 状态码 | 名称 | 说明 |
|--------|------|------|
| **0x00** | CTAP_STATUS_OK | 成功 |
| **0x01** | CTAP_ERR_INVALID_COMMAND | 无效命令 |
| **0x02** | CTAP_ERR_INVALID_PARAMETER | 无效参数 |
| **0x03** | CTAP_ERR_INVALID_LENGTH | 无效长度 |
| **0x04** | CTAP_ERR_PIN_INVALID | PIN 无效 |
| **0x05** | CTAP_ERR_PIN_BLOCKED | PIN 锁定 |
| **0x06** | CTAP_ERR_UV_BLOCKED | UV 锁定 |
| **0x07** | CTAP_ERR_NO_CREDENTIALS | 无凭证 |
| **0x19** | CTAP_ERR_CREDENTIAL_EXCLUDED | 凭证已排除 |

### 9.4 与 URD 需求映射表

| SRS 需求编号 | URD 需求编号 | 需求描述 |
|--------------|--------------|----------|
| SR-F01 | UR-F02 | MakeCredential 创建凭证 |
| SR-F02 | UR-F01 | GetAssertion 指纹登录 |
| SR-F03 | UR-F03 | USB 连接 |
| SR-F04 | UR-F04 | BLE 连接 |
| SR-F09 | UR-F05 | 指纹验证 < 1 秒 |
| SR-F10 | UR-NF06, UR-NF07 | 指纹 FAR/FRR |
| SR-F11 | UR-F10 | 指纹注册 |
| SR-F05 | UR-F12 | PIN 设置 |
| SR-F07 | UR-F13, UR-F14 | 凭证管理 |
| SR-F06 | UR-F15 | Reset 重置 |
| SR-F13 | UR-F08 | Windows/Mac/Linux 兼容 |
| SR-F14 | UR-F09 | iOS/Android 兼容 |
| SR-F15 | UR-NF12 | 密钥不可导出 |
| SR-F18 | UR-NF11 | 指纹本地验证 |

---

## 十、文档演进链

根据 AGENTS.md 第三章文档演进链定义：

```
BRD → URD → SRS（本文档）→ HLD → LLD → 测试/部署/运维文档
```

**当前文档状态**：

| 文档 | 状态 | 说明 |
|------|------|------|
| **BRD.md** | 已完成 | 商业需求文档 |
| **URD.md** | 已完成 | 用户需求文档 |
| **SRS.md** | 初稿（本文档） | 软件需求规格文档 |
| **HLD.md** | 已完成 | 系统架构设计 |
| **LLD.md** | 待创建 | 详细设计文档 |

---

## 十一、文档修订历史

| 版本 | 修订日期 | 修订内容 | 修订人 |
|------|----------|----------|--------|
| **v1.0** | 2026-04-07 | 初稿创建 | IBio 项目团队 |

---

**文档结束**