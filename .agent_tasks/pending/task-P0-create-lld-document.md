# 任务文档：创建 LLD.md 低级设计文档

**任务状态**：pending
**创建时间**：2026-04-07
**创建者**：Planner
**优先级**：P0（最高）

---

## 一、任务概述

### 1.1 任务目标

创建 IBio 项目的 LLD.md 低级设计文档，完成文档演进链的下一阶段。

**文档演进链**：`BRD → URD → SRS → HLD → LLD（本任务）→ 测试/部署/运维文档`

### 1.2 操作范围

**类型**：单子项目
**项目名称**：IBio
**项目路径**：/workspaces/agent-workspace/projects/IBio/

### 1.3 任务类型

**类型**：文档创建任务
**触发原因**：HLD.md 已完成，需要进行下一阶段的详细设计

---

## 二、文件信息

### 2.1 目标文件

| 项目 | 信息 |
|------|------|
| **文件路径** | `/workspaces/agent-workspace/projects/IBio/docs/LLD.md` |
| **文件类型** | Markdown 文档 |
| **操作类型** | 创建新文件 |

### 2.2 相关依赖文件

| 文件路径 | 用途 |
|----------|------|
| `/workspaces/agent-workspace/projects/IBio/docs/SRS.md` | 需求规格参考 |
| `/workspaces/agent-workspace/projects/IBio/docs/HLD.md` | 系统架构参考 |
| `/workspaces/agent-workspace/projects/IBio/docs/adr/*.md` | ADR 决策参考 |

---

## 三、LLD 文档内容规划

### 3.1 文档结构规划

LLD.md 文档必须包含以下章节：

```
# IBio 低级设计文档（LLD）

## 一、引言
- 1.1 目的
- 1.2 范围
- 1.3 参考资料

## 二、类图设计
- 2.1 HAL 层类图
- 2.2 安全模块类图
- 2.3 指纹模块类图
- 2.4 CTAP 协议栈类图
- 2.5 通信模块类图

## 三、Flash 分区设计
- 3.1 分区表定义
- 3.2 凭证存储分区
- 3.3 配置存储分区
- 3.4 指纹模板映射分区

## 四、数据结构定义
- 4.1 凭证数据结构
- 4.2 指纹模板映射数据结构
- 4.3 配置数据结构
- 4.4 认证器状态数据结构

## 五、API 详细定义
- 5.1 HAL 层 API
- 5.2 安全模块 API
- 5.3 指纹模块 API
- 5.4 CTAP 协议栈 API
- 5.5 通信模块 API

## 六、状态机设计
- 6.1 认证流程状态机
- 6.2 指纹验证流程状态机
- 6.3 PIN 验证流程状态机

## 七、错误处理机制
- 7.1 错误码定义表
- 7.2 错误恢复策略
- 7.3 错误日志记录

## 八、引脚分配解决方案
- 8.1 GPIO19 引脚冲突分析
- 8.2 解决方案对比
- 8.3 最终方案

## 九、性能预算
- 9.1 各模块性能指标
- 9.2 总体性能预算

## 十、附录
- 10.1 ATECC608A 密钥槽位详细配置
- 10.2 FPM383C 命令协议详细定义
- 10.3 CBOR 编码示例
```

### 3.2 各章节详细内容规划

#### 章节 1：引言

**内容**：
- 文档目的和范围
- 与 HLD.md 的关系
- 参考文档列表

**验收标准**：
- 包含完整的引言章节
- 参考文档列表正确引用 SRS.md、HLD.md、ADR 文档

---

#### 章节 2：类图设计

**内容**：

**2.1 HAL 层类图**：
- GPIO 驱动类：`gpio_driver_t`
- UART 驱动类：`uart_driver_t`
- I2C 驱动类：`i2c_driver_t`
- USB 驱动类：`usb_hid_driver_t`
- BLE 驱动类：`ble_gatt_driver_t`
- Flash 驱动类：`flash_driver_t`

**2.2 安全模块类图**：
- 密钥管理类：`key_manager_t`
- 加密引擎类：`crypto_engine_t`
- 用户验证类：`user_verification_t`
- ATECC608A 适配类：`atecc608a_adapter_t`

**2.3 指纹模块类图**：
- 指纹采集类：`fingerprint_capture_t`
- 指纹比对类：`fingerprint_match_t`
- 模板管理类：`template_manager_t`
- FPM383C 适配类：`fpm383c_adapter_t`

**2.4 CTAP 协议栈类图**：
- CTAP 处理器类：`ctap_processor_t`
- 凭证管理类：`credential_manager_t`
- 签名引擎类：`signature_engine_t`

**2.5 通信模块类图**：
- USB 传输类：`usb_transport_t`
- BLE 传输类：`ble_transport_t`
- 消息分片类：`message_fragmenter_t`

**验收标准**：
- 所有核心模块都有类图
- 类图使用 Mermaid 或 PlantUML 格式
- 类图包含属性、方法、关系

---

#### 章节 3：Flash 分区设计

**内容**：

**3.1 分区表定义**：
- ESP32-S3 Flash 分区表（CSV 格式）
- 总 Flash 大小：8 MB
- 分区类型：nvs、factory、ota、storage

**分区规划**：
| 分区名称 | 偏移地址 | 大小 | 类型 | 用途 |
|----------|----------|------|------|------|
| nvs | 0x9000 | 24 KB | data | NVS 配置存储 |
| otadata | 0x8000 | 8 KB | data | OTA 状态 |
| factory | 0x10000 | 1.5 MB | app | 固件主分区 |
| ota_0 | 0x190000 | 1.5 MB | app | OTA 分区 0 |
| ota_1 | 0x2E0000 | 1.5 MB | app | OTA 分区 1 |
| credential | 0x430000 | 128 KB | data | 凭证存储 |
| fingerprint | 0x450000 | 64 KB | data | 指纹模板映射 |
| config | 0x460000 | 32 KB | data | 配置存储 |

**3.2 凭证存储分区详细设计**：
- 凭证存储结构：链表或数组
- 凭证索引：credential_id → 存储位置映射
- 凭证容量计算：128 KB / 400 bytes ≈ 320 个凭证

**3.3 配置存储分区详细设计**：
- NVS 键值存储结构
- 配置项列表：AAGUID、PIN 状态、重试计数器等

**3.4 指纹模板映射分区详细设计**：
- 模板映射表：template_id → user_id 映射
- 模板容量：200 个模板映射

**验收标准**：
- 分区表完整，偏移地址和大小正确
- 分区总大小不超过 8 MB
- 分区类型和用途明确

---

#### 章节 4：数据结构定义

**内容**：

**4.1 凭证数据结构**：
```c
typedef struct {
    uint8_t credential_id[32];        // Credential ID
    uint8_t key_slot;                  // ATECC608A 密钥槽位
    uint8_t rp_id_hash[32];           // RP ID 哈希
    uint8_t user_id[64];              // 用户 ID
    uint8_t user_id_len;              // 用户 ID 长度
    char user_name[64];               // 用户名称
    char user_display_name[64];       // 用户显示名称
    uint32_t create_time;             // 创建时间
    uint32_t sign_count;              // 签名计数器
    uint8_t reserved[32];             // 保留字段
} credential_t;  // 总大小约 200-300 bytes
```

**4.2 指纹模板映射数据结构**：
```c
typedef struct {
    uint8_t template_id;              // FPM383C 模板 ID (0-199)
    char friendly_name[32];           // 模板友好名称
    uint32_t create_time;             // 注册时间
    uint8_t active;                   // 是否激活
    uint8_t reserved[8];              // 保留字段
} fingerprint_mapping_t;  // 总大小约 48 bytes
```

**4.3 配置数据结构**：
```c
typedef struct {
    uint8_t aaguid[16];               // 认证器 AAGUID
    uint8_t pin_set;                  // PIN 是否设置
    uint8_t pin_retries;              // PIN 重试次数
    uint8_t uv_retries;               // UV 重试次数
    uint8_t credential_count;         // 凭证数量
    uint8_t fingerprint_count;        // 指纹模板数量
    char serial_number[16];           // 设备序列号
    uint8_t firmware_version[3];      // 固件版本
} authenticator_config_t;
```

**4.4 认证器状态数据结构**：
```c
typedef struct {
    uint8_t state;                    // 当前状态
    uint8_t pending_command;          // 待处理命令
    uint8_t uv_method;                // 用户验证方式
    uint8_t uv_status;                // 用户验证状态
} authenticator_state_t;
```

**验收标准**：
- 所有数据结构有完整的字段定义
- 数据结构大小估算正确
- 字段类型和长度明确

---

#### 章节 5：API 详细定义

**内容**：

**5.1 HAL 层 API**（参考 SRS.md 第四章接口需求）：

**GPIO API**：
```c
/**
 * @brief 初始化 GPIO
 * @param pin GPIO 引脚号
 * @param mode GPIO 模式
 * @return 0 成功，其他失败
 */
int gpio_init(uint8_t pin, gpio_mode_t mode);

/**
 * @brief 读取 GPIO 值
 * @param pin GPIO 引脚号
 * @return GPIO 值 (0/1)，负数失败
 */
int gpio_read(uint8_t pin);

/**
 * @brief 写入 GPIO 值
 * @param pin GPIO 引脚号
 * @param value GPIO 值 (0/1)
 * @return 0 成功，其他失败
 */
int gpio_write(uint8_t pin, uint8_t value);
```

**UART API**：
```c
/**
 * @brief 初始化 UART
 * @param port UART 端口号
 * @param config UART 配置参数
 * @return 0 成功，其他失败
 */
int uart_init(uint8_t port, uart_config_t *config);

/**
 * @brief 从 UART 读取数据
 * @param port UART 端口号
 * @param data 数据缓冲区
 * @param len 数据长度
 * @param timeout 超时时间（ms）
 * @return 实际读取长度，负数失败
 */
int uart_read(uint8_t port, uint8_t *data, uint32_t len, uint32_t timeout);

/**
 * @brief 向 UART 写入数据
 * @param port UART 端口号
 * @param data 数据缓冲区
 * @param len 数据长度
 * @return 0 成功，其他失败
 */
int uart_write(uint8_t port, uint8_t *data, uint32_t len);
```

**I2C API**：
```c
/**
 * @brief 初始化 I2C
 * @param bus I2C 总线号
 * @param config I2C 配置参数
 * @return 0 成功，其他失败
 */
int i2c_init(uint8_t bus, i2c_config_t *config);

/**
 * @brief 从 I2C 设备读取数据
 * @param bus I2C 总线号
 * @param addr 设备地址
 * @param data 数据缓冲区
 * @param len 数据长度
 * @return 0 成功，其他失败
 */
int i2c_read(uint8_t bus, uint8_t addr, uint8_t *data, uint32_t len);

/**
 * @brief 向 I2C 设备写入数据
 * @param bus I2C 总线号
 * @param addr 设备地址
 * @param data 数据缓冲区
 * @param len 数据长度
 * @return 0 成功，其他失败
 */
int i2c_write(uint8_t bus, uint8_t addr, uint8_t *data, uint32_t len);
```

**5.2 安全模块 API**：

**密钥管理 API**：
```c
/**
 * @brief 生成密钥对
 * @param key_type 密钥类型 (ES256, ECDH)
 * @param handle 密钥句柄输出
 * @return 0 成功，其他失败
 * @note 密钥由 ATECC608A 硬件生成
 */
int key_generate(uint8_t key_type, key_handle_t *handle);

/**
 * @brief 获取公钥
 * @param handle 密钥句柄
 * @param pubkey 公钥缓冲区
 * @param len 公钥长度输出
 * @return 0 成功，其他失败
 */
int key_get_public_key(key_handle_t *handle, uint8_t *pubkey, uint32_t *len);

/**
 * @brief 使用密钥签名
 * @param handle 密钥句柄
 * @param data 待签名数据
 * @param len 数据长度
 * @param sig 签名输出缓冲区
 * @param sig_len 签名长度输出
 * @return 0 成功，其他失败
 */
int key_sign(key_handle_t *handle, uint8_t *data, uint32_t len, uint8_t *sig, uint32_t *sig_len);

/**
 * @brief 执行 ECDH 密钥协商
 * @param handle 密钥句柄
 * @param peer_pubkey 对方公钥
 * @param shared_secret 共享密钥输出
 * @return 0 成功，其他失败
 */
int key_ecdh(key_handle_t *handle, uint8_t *peer_pubkey, uint8_t *shared_secret);

/**
 * @brief 删除密钥
 * @param handle 密钥句柄
 * @return 0 成功，其他失败
 */
int key_delete(key_handle_t *handle);
```

**用户验证 API**：
```c
/**
 * @brief 请求用户验证
 * @param method 验证方式 (UV_PIN, UV_FINGERPRINT)
 * @return 0 成功开始验证，其他失败
 */
int uv_request(uv_method_t method);

/**
 * @brief 获取验证结果
 * @param success 验证成功标志输出
 * @return 0 成功获取结果，其他失败
 */
int uv_get_result(uint8_t *success);

/**
 * @brief 设置 PIN
 * @param pin PIN 值
 * @param len PIN 长度
 * @return 0 成功，其他失败
 */
int uv_set_pin(uint8_t *pin, uint32_t len);

/**
 * @brief 验证 PIN
 * @param pin PIN 值
 * @param len PIN 镀度
 * @return 0 成功，其他失败
 */
int uv_verify_pin(uint8_t *pin, uint32_t len);
```

**5.3 指纹模块 API**：

**指纹采集 API**：
```c
/**
 * @brief 开始指纹采集
 * @return 0 成功开始采集，其他失败
 */
int fp_capture_start(void);

/**
 * @brief 等待指纹采集完成
 * @param image 图像缓冲区（可选）
 * @param len 图像长度输出（可选）
 * @param timeout 超时时间（ms）
 * @return 0 成功采集，其他失败
 */
int fp_capture_wait(uint8_t *image, uint32_t *len, uint32_t timeout);

/**
 * @brief 停止指纹采集
 * @return 0 成功停止，其他失败
 */
int fp_capture_stop(void);
```

**指纹比对 API**：
```c
/**
 * @brief 指纹 1:1 比对
 * @param template_id 模板 ID
 * @param match_score 匹配分数输出
 * @return 0 匹配成功，其他失败或不匹配
 */
int fp_match(uint8_t template_id, uint8_t *match_score);

/**
 * @brief 指纹 1:N 搜索
 * @param template_id 匹配模板 ID 输出
 * @param match_score 匹配分数输出
 * @return 0 找到匹配，其他失败或未找到
 */
int fp_search(uint8_t *template_id, uint8_t *match_score);
```

**模板管理 API**：
```c
/**
 * @brief 开始指纹注册
 * @return 0 成功开始注册，其他失败
 */
int fp_enroll_start(void);

/**
 * @brief 添加指纹图像到注册流程
 * @param image 图像数据（可选）
 * @param len 图像长度（可选）
 * @param quality 图像质量输出
 * @return 0 成功添加，其他失败
 */
int fp_enroll_add_image(uint8_t *image, uint32_t len, uint8_t *quality);

/**
 * @brief 完成指纹注册
 * @param template_id 模板 ID 输出
 * @return 0 成功注册，其他失败
 */
int fp_enroll_finish(uint8_t *template_id);

/**
 * @brief 删除指纹模板
 * @param template_id 模板 ID
 * @return 0 成功删除，其他失败
 */
int fp_template_delete(uint8_t template_id);

/**
 * @brief 清空所有指纹模板
 * @return 0 成功清空，其他失败
 */
int fp_template_clear_all(void);
```

**5.4 CTAP 协议栈 API**：

**CTAP 处理 API**：
```c
/**
 * @brief 处理 CTAP 请求
 * @param req CTAP 请求结构
 * @param resp CTAP 响应结构输出
 * @return 0 成功处理，其他失败
 */
int ctap_process_request(ctap_request_t *req, ctap_response_t *resp);
```

**CTAP 命令常量定义**：
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

**5.5 通信模块 API**：

**USB HID API**：
```c
/**
 * @brief 初始化 USB HID
 * @return 0 成功，其他失败
 */
int usb_hid_init(void);

/**
 * @brief 发送 USB HID 数据
 * @param data 数据缓冲区
 * @param len 数据长度
 * @return 0 成功，其他失败
 */
int usb_hid_send(uint8_t *data, uint32_t len);

/**
 * @brief 接收 USB HID 数据
 * @param data 数据缓冲区
 * @param len 数据长度输出
 * @param timeout 超时时间（ms）
 * @return 0 成功，其他失败
 */
int usb_hid_recv(uint8_t *data, uint32_t *len, uint32_t timeout);
```

**BLE GATT API**：
```c
/**
 * @brief 初始化 BLE GATT
 * @return 0 成功，其他失败
 */
int ble_gatt_init(void);

/**
 * @brief 开始 BLE 广播
 * @return 0 成功，其他失败
 */
int ble_adv_start(void);

/**
 * @brief 停止 BLE 广播
 * @return 0 成功，其他失败
 */
int ble_adv_stop(void);
```

**验收标准**：
- 所有 API 有完整的函数签名
- 所有 API 有参数说明和返回值说明
- 所有 API 有注释说明用途

---

#### 章节 6：状态机设计

**内容**：

**6.1 认证流程状态机**：
- 状态定义：IDLE → WAITING_TOUCH → VERIFYING → SIGNING → COMPLETE
- 使用 Mermaid 状态图表示
- 定义状态转换条件和触发事件

**6.2 指纹验证流程状态机**：
- 状态定义：IDLE → CAPTURING → MATCHING → SUCCESS/FAIL
- 定义指纹采集、比对、结果处理流程

**6.3 PIN 验证流程状态机**：
- 状态定义：IDLE → WAITING_PIN → VERIFYING → SUCCESS/FAIL/BLOCKED
- 定义 PIN 输入、验证、锁定处理流程

**验收标准**：
- 所有状态机使用 Mermaid 状态图格式
- 状态定义清晰，转换条件明确
- 状态机覆盖所有核心流程

---

#### 章节 7：错误处理机制

**内容**：

**7.1 错误码定义表**：

参考 SRS.md 附录的 CTAP 状态码，补充内部错误码：

| 错误码 | 名称 | 说明 | 处理方式 |
|--------|------|------|----------|
| 0x00 | SUCCESS | 成功 | 正常返回 |
| 0x01 | ERR_INVALID_CMD | 无效命令 | 返回错误响应 |
| 0x02 | ERR_INVALID_PARAM | 无效参数 | 返回错误响应 |
| 0x03 | ERR_INVALID_LENGTH | 无效长度 | 返回错误响应 |
| 0x04 | ERR_PIN_INVALID | PIN 无效 | 减少重试次数 |
| 0x05 | ERR_PIN_BLOCKED | PIN 锁定 | 需 Reset 解锁 |
| 0x06 | ERR_UV_BLOCKED | UV 锁定 | 需 Reset 解锁 |
| 0x07 | ERR_NO_CREDENTIALS | 无凭证 | 返回错误响应 |
| 0x19 | ERR_CREDENTIAL_EXCLUDED | 凭证已排除 | 返回错误响应 |
| 0xE0 | ERR_HARDWARE | 硬件错误 | 记录日志，重试 |
| 0xE1 | ERR_UART_TIMEOUT | UART 超时 | 重试，超限报错 |
| 0xE2 | ERR_I2C_TIMEOUT | I2C 超时 | 重试，超限报错 |
| 0xE3 |_ERR_FLASH_WRITE | Flash 写入失败 | 重试，记录日志 |
| 0xE4 | ERR_CRYPTO | 加密操作失败 | 返回错误响应 |
| 0xE5 | ERR_FINGERPRINT | 指纹操作失败 | 返回错误响应 |

**7.2 错误恢复策略**：

| 错误类型 | 恢复策略 | 说明 |
|----------|----------|------|
| **硬件错误** | 重试机制 | 超过 3 次重试后记录日志并返回错误 |
| **通信超时** | 重试机制 | UART/I2C 超时后重试，超过 3 次报错 |
| **Flash 错误** | 重试机制 | Flash 写入失败后重试，严重错误记录 |
| **PIN 锁定** | Reset 解锁 | PIN 锁定后需要执行 Reset 命令解锁 |
| **UV 锁定** | Reset 解锁 | UV 锁定后需要执行 Reset 命令解锁 |

**7.3 错误日志记录**：

定义错误日志格式：
```c
typedef struct {
    uint32_t timestamp;    // 时间戳
    uint8_t error_code;    // 错误码
    uint8_t module;        // 模块 ID
    uint16_t details;      // 详细信息
} error_log_t;
```

**验收标准**：
- 错误码覆盖所有可能的错误场景
- 错误恢复策略明确可行
- 错误日志格式定义清晰

---

#### 章节 8：引脚分配解决方案

**内容**：

**8.1 GPIO19 引脚冲突分析**：

根据 HLD.md 和 SRS.md 的硬件接口定义：
- GPIO19 原规划：FPM383C TOUCH（指纹传感器触摸检测）
- GPIO19 也用于：USB-C CC2（USB-C 配置通道 2）

**冲突原因**：
- ESP32-S3 内置 USB PHY 使用 GPIO19 (USB_DN) 和 GPIO20 (USB_DP)
- USB-C CC2 通常连接到 GPIO19 或 GPIO20
- 但 USB-C CC2 实际应使用独立 GPIO，不应与 USB_DN/DP 冲突
- HLD.md 原设计中 GPIO19 用于 FPM383C TOUCH，但 SRS.md 显示 USB_DN 使用 GPIO19

**实际分析**：
ESP32-S3 USB 引脚：
- GPIO19: USB_DN (USB 数据负)
- GPIO20: USB_DP (USB 数据正)

USB-C CC1/CC2 引脚（配置通道）：
- CC1: GPIO25（HLD.md 定义）
- CC2: GPIO26（HLD.md 定义）

**结论**：GPIO19 不应用于 USB-C CC2，GPIO19 是 USB_DN 数据引脚。原设计可能是文档错误或规划不当。

**8.2 解决方案对比**：

| 方案 | FPM383C TOUCH 引脚 | 优点 | 缺点 |
|------|---------------------|------|------|
| **方案 A** | GPIO18（原 EN 引脚改为 TOUCH） | GPIO18 附近，引脚合理 | 需要重新设计 EN 控制方式 |
| **方案 B** | GPIO4/GPIO5（未使用 GPIO） | 无冲突 | 引脚位置可能不理想 |
| **方案 C** | FPM383C 内部触摸检测（UART 命令） | 无需 GPIO | 依赖模块响应时间 |
| **方案 D** | 轮询检测（软件方案） | 无需 GPIO | 响应延迟 |

**推荐方案**：
- 方案 A：GPIO18 用于 FPM383C TOUCH
- FPM383C EN 控制使用软件控制或常使能
- GPIO19 保持 USB_DN 功能

**8.3 最终方案**：

| 引脚 | 功能 | 说明 |
|------|------|------|
| GPIO19 | USB_DN | USB 数据负（内置 USB PHY） |
| GPIO20 | USB_DP | USB 数据正（内置 USB PHY） |
| GPIO18 | FPM383C TOUCH | 指纹传感器触摸检测 |
| GPIO25 | USB-C CC1 | USB-C 配置通道 1 |
| GPIO26 | USB-C CC2 | USB-C 配置通道 2 |

**验收标准**：
- 引脚冲突分析正确
- 解决方案可行
- 最终引脚分配无冲突

---

#### 章节 9：性能预算

**内容**：

**9.1 各模块性能指标**：

| 模块 | 操作 | 目标时间 | 验收标准 |
|------|------|----------|----------|
| **指纹采集** | 单次采集 | < 500 ms | FPM383C 规格 |
| **指纹比对** | 1:N 搜索 | < 1000 ms | FPM383C 规格 |
| **ES256 签名** | 签名操作 | < 100 ms | ATECC608A 规格 |
| **ECDH** | 密钥协商 | < 100 ms | ATECC608A 规格 |
| **SHA-256** | 哈希计算 | < 50 ms | ESP32 硬件加速 |
| **CBOR 编码** | 编码/解码 | < 20 ms | 代码效率 |
| **Flash 读写** | 单次读写 | < 10 ms | ESP32 Flash 性能 |
| **USB 响应** | 单次传输 | < 50 ms | USB HID 性能 |

**9.2 总体性能预算**：

| 流程 | 操作分解 | 总时间预算 |
|------|----------|------------|
| **MakeCredential** | UV + 密钥生成 + 存储 + 签名 | < 2000 ms |
| **GetAssertion** | UV + 凭证检索 + 签名 | < 1500 ms |
| **指纹验证** | 采集 + 比对 | < 1000 ms |

**验收标准**：
- 各模块性能指标明确
- 总体性能预算符合 SRS.md 非功能需求

---

#### 章节 10：附录

**内容**：

**10.1 ATECC608A 密钥槽位详细配置**：
- Slot 0-9：凭证密钥配置详情
- Slot 14：Attestation 密钥配置详情
- Slot 15：PIN 密钥配置详情

**10.2 FPM383C 命令协议详细定义**：
- 常用命令列表和格式
- 响应格式定义

**10.3 CBOR 编码示例**：
- MakeCredential 请求编码示例
- GetAssertion 响应编码示例

**验收标准**：
- 附录内容补充完整
- 附录内容与正文一致

---

## 四、执行步骤规划

### 4.1 执行步骤

| 步骤 | 内容 | 时间预估 |
|------|------|----------|
| **1** | 创建 LLD.md 文件框架（章节结构） | 10 分钟 |
| **2** | 编写引言章节（章节 1） | 5 分钟 |
| **3** | 编写类图设计章节（章节 2） | 30 分钟 |
| **4** | 编写 Flash 分区设计章节（章节 3） | 20 分钟 |
| **5** | 编写数据结构定义章节（章节 4） | 20 分钟 |
| **6** | 编写 API 详细定义章节（章节 5） | 30 分钟 |
| **7** | 编写状态机设计章节（章节 6） | 20 分钟 |
| **8** | 编写错误处理机制章节（章节 7） | 15 分钟 |
| **9** | 编写引脚分配解决方案章节（章节 8） | 15 分钟 |
| **10** | 编写性能预算章节（章节 9） | 10 分钟 |
| **11** | 编写附录章节（章节 10） | 15 分钟 |
| **12** | 文档完整性检查和格式调整 | 10 分钟 |

### 4.2 步骤依赖关系

```
步骤 1（创建框架）
    ↓
步骤 2-11（并行编写各章节）
    ↓
步骤 12（完整性检查）
```

---

## 五、验收标准

### 5.1 文档完整性验收标准

| 验收项 | 验收标准 | 验收方法 |
|--------|----------|----------|
| **章节完整性** | 所有 10 个章节全部完成 | 检查文档结构 |
| **类图完整性** | 所有核心模块有类图 | 检查类图章节 |
| **分区表完整性** | 分区表完整无冲突 | 检查分区章节 |
| **API 完整性** | 所有核心 API 定义完整 | 检查 API 章节 |
| **状态机完整性** | 核心流程状态机完整 | 检查状态机章节 |
| **错误码完整性** | 错误码覆盖所有场景 | 检查错误处理章节 |

### 5.2 内容正确性验收标准

| 验收项 | 验收标准 | 验收方法 |
|--------|----------|----------|
| **与 SRS 一致** | API 定义与 SRS.md 接口需求一致 | 对比检查 |
| **与 HLD 一致** | 类图设计与 HLD.md 模块划分一致 | 对比检查 |
| **与 ADR 一致** | 技术选型与 ADR 文档一致 | 对比检查 |
| **引脚分配正确** | GPIO19 引脚冲突已解决 | 检查解决方案章节 |

### 5.3 格式规范验收标准

| 验收项 | 验收标准 | 验收方法 |
|--------|----------|----------|
| **Markdown 格式** | 符合 Markdown 规范 | 格式检查 |
| **Mermaid 图表** | 类图和状态图格式正确 | 图表渲染检查 |
| **代码块格式** | 代码块语法正确 | 语法检查 |

---

## 六、回滚方案

### 6.1 回滚条件

| 条件 | 说明 |
|------|------|
| **文档创建失败** | 文件写入失败或格式错误 |
| **验收不通过** | 内容不完整或不正确 |
| **用户要求撤销** | 用户明确要求撤销任务 |

### 6.2 回滚操作

```bash
# 删除 LLD.md 文件
rm /workspaces/agent-workspace/projects/IBio/docs/LLD.md

# 如果已提交 Git，执行回滚
git checkout -- /workspaces/agent-workspace/projects/IBio/docs/LLD.md
```

---

## 七、分支规划

### 7.1 分支信息

| 项目 | 信息 |
|------|------|
| **基础分支** | main 或 develop（根据项目状态确定） |
| **任务分支** | task/P0-create-lld |
| **合并目标** | 基础分支 |

### 7.2 Git 提交信息

```
docs: 创建 IBio LLD.md 低级设计文档

- 完成类图设计（HAL、安全、指纹、CTAP、通信模块）
- 完成 Flash 分区表设计（8MB 分区规划）
- 完成 API 详细定义（所有核心接口）
- 完成状态机设计（认证、指纹验证、PIN 验证流程）
- 完成错误处理机制（错误码定义和恢复策略）
- 解决 GPIO19 引脚冲突问题
- 完成 ATECC608A 密钥槽位详细配置
```

---

## 八、规则引用记录

| 规则文件 | 相关性 | 应用方式 | 影响内容 |
|----------|--------|----------|----------|
| AGENTS.md（第三章） | 强相关 | 约束 | LLD 文档内容要求（类图、数据库表结构、API定义、状态机、错误处理） |
| AGENTS.md（第十章） | 强相关 | 约束 | 任务文档命名规范、状态管理 |
| AGENTS_PLANNER.md（第六章） | 强相关 | 约束 | 任务文档创建流程、提交要求 |
| build-rule.md | 弱相关 | 参考 | Windows 编译要求（不直接相关） |
| e2e-test-console-rule.md | 弱相关 | 参考 | 端到端测试要求（后续测试阶段参考） |

---

## 九、实时更新区域

### 9.1 执行进度

| 步骤 | 状态 | 开始时间 | 完成时间 | 备注 |
|------|------|----------|----------|------|
| **规划完成** | ✓ completed | 2026-04-07 | 2026-04-07 | 任务文档创建完成 |
| **待执行** | pending | - | - | 等待 Coder 执行 |

### 9.2 问题记录

| 时间 | 问题描述 | 处理方式 | 状态 |
|------|----------|----------|------|
| - | 无问题 | - | - |

### 9.3 审核记录

| 时间 | 审核者 | 审核结果 | 备注 |
|------|--------|----------|------|
| - | - | 待审核 | - |

---

## 十、任务状态历史

| 状态 | 时间 | 说明 |
|------|------|------|
| **pending** | 2026-04-07 | 任务文档创建，等待执行 |

---

**任务状态**：pending
**下次更新时间**：Coder 开始执行后