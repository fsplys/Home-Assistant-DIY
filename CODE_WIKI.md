# HomeAssistant 智能家居实战篇 — Code Wiki

> 本仓库对应《HomeAssistant智能家居实战篇》系列视频课程，是一套围绕 **Home Assistant (HA)** 智能家居平台搭建、设备接入、自动化、语音交互、硬件 DIY 等主题编写的综合学习资料与配套源代码。
>
> 本文档（Code Wiki）对仓库的整体结构、主要模块职责、关键类与函数、依赖关系以及运行方式进行系统化梳理，便于后续维护与二次开发。

---

## 目录

1. [项目简介与定位](#1-项目简介与定位)
2. [仓库整体目录结构](#2-仓库整体目录结构)
3. [模块职责划分](#3-模块职责划分)
4. [核心 Python 模块详解](#4-核心-python-模块详解)
5. [HomeAssistant 自定义组件（custom_components）](#5-homeassistant-自定义组件custom_components)
6. [嵌入式 / 硬件代码](#6-嵌入式--硬件代码)
7. [YAML 配置示例](#7-yaml-配置示例)
8. [依赖关系分析](#8-依赖关系分析)
9. [项目运行方式](#9-项目运行方式)
10. [架构总览与数据流](#10-架构总览与数据流)
11. [扩展与维护建议](#11-扩展与维护建议)

---

## 1. 项目简介与定位

| 维度 | 说明 |
| --- | --- |
| 项目性质 | 课程配套资料 + 可运行示例代码 |
| 目标读者 | Home Assistant 学习者、智能家居 DIY 爱好者、Python/嵌入式开发者 |
| 核心平台 | Home Assistant（开源智能家居中枢） |
| 涵盖技术栈 | Python（HA 组件、语音助手）、YAML（HA 配置）、C/C++（Arduino/ESP8266）、Node-RED、AppDaemon、Docker、MQTT、Modbus/485、KNX、ONVIF 等 |
| 配套视频 | [网易云课堂 - HomeAssistant智能家居实战篇](https://study.163.com/course/introduction.htm?courseId=1006189053) |

仓库以"教程文档 + 可运行示例代码"的形式组织，覆盖从入门准备、组件接入、公网访问、设备定位、自动化、摄像头、人脸识别，到 DIY 智能硬件、485 总线、KNX、室外物联网、Docker 等 30+ 主题。

---

## 2. 仓库整体目录结构

```
/workspace
├── README.md                              # 仓库总入口，列出全部视频目录与课程追求
├── __init__.py                            # 空包标记，使根目录可作为 Python 包
├── ha_cli.py                              # Home Assistant REST API 客户端（语音助手使用）
├── ais_cli.py                             # 公共 AI（图灵机器人 tuling123）API 客户端
├── 参考文档（1-10）/                       # 第 1~10 篇：入门、组件、进阶、公网、定位、自动化等
│   ├── *.docx / *.pdf                     # 各篇章教程文档
│   └── example_*.yaml                     # 章节对应的 HA 配置示例
├── 参考文档（11-20）/                      # 第 11~20 篇：HomeKit、数据记录、AppDaemon、Node-RED、GPIO、MQTT、ESP8266 等
│   ├── *.docx / *.pdf
│   ├── example_*.yaml
│   ├── example_13_*.dash                  # AppDaemon DashBoard 配置
│   └── example_18_*.zip                   # 开发者篇打包示例
├── 参考文档（21-30）/                      # 第 21~30 篇：抓取、智能音箱、灯带、远程麦克风、485、KNX、室外物联网、Docker、红外/433
│   ├── *.docx / *.pdf
│   ├── custom_components/                 # HA 自定义组件示例
│   │   └── bus485_example/                # 485 总线 Modbus 继电器开关组件
│   ├── LocReport/                         # 室外物联网：Arduino + SIM7000 NB-IoT/GPS 定位上报
│   ├── example_24_audio_input/            # ESP32 I2S PDM 麦克风采集
│   └── example_24_audio_output/           # ESP32 I2S 音频输出
├── 补充篇章/                               # Linux 命令、文本编辑、YAML、Python 虚拟环境等基础补充
└── 加餐/                                   # 扩展专题：TensorFlow 物体识别、NFC、ESPHome 自动化等
```

---

## 3. 模块职责划分

仓库的"代码资产"可按职责分为以下几类：

| 模块 | 位置 | 职责 |
| --- | --- | --- |
| **HA 客户端** | [ha_cli.py](file:///workspace/ha_cli.py) | 封装 Home Assistant REST API，供语音助手调用服务、对话处理、TTS 播报、通知 |
| **AI 对话客户端** | [ais_cli.py](file:///workspace/ais_cli.py) | 调用图灵机器人（tuling123）开放 API，将语音文本转为智能应答文本 |
| **语音识别库（改写版）** | `__init__.py`（根目录） | 基于 `speech_recognition 3.8.1` 改造：增加 `recognize_google_cn`、修正 Snowboy 调用、增加热词回调与最小录音时长等 |
| **HA 自定义组件** | [参考文档（21-30）/custom_components/bus485_example/](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/custom_components/bus485_example) | 通过 485 串口 + Modbus RTU 协议接入多路继电器开关 |
| **嵌入式定位上报** | [参考文档（21-30）/LocReport/](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/LocReport) | Arduino + SIM7000（NB-IoT + GPS）通过 TCP 上报位置到 traccar |
| **ESP32 音频 IO** | `example_24_audio_input/`、`example_24_audio_output/` | 基于 ESP-IDF I2S 驱动实现 PDM 麦克风采集与音频播放 |
| **HA 配置示例** | 各 `参考文档` 目录内 `example_*.yaml` | 自动化、传感器、灯光、摄像头、MQTT、AppDaemon 等 YAML 配置样例 |
| **教程文档** | `*.docx` / `*.pdf` | 课程逐章讲解材料 |

---

## 4. 核心 Python 模块详解

### 4.1 `ha_cli.py` — Home Assistant REST 客户端

**文件**：[ha_cli.py](file:///workspace/ha_cli.py)

**职责**：为"DIY 智能音箱"语音助手提供对 Home Assistant 的 HTTP 访问能力，包括：
- 调用 HA 对话引擎（conversation）
- 调用任意 HA 服务（services）
- 发起 TTS 语音播报
- 发送持久化通知

#### 类 `ha_cli`

| 成员 | 类型 | 说明 |
| --- | --- | --- |
| `__init__(token, base_url='http://localhost:8123', verify=False)` | 构造方法 | 建立 `requests.Session`，注入 `Authorization: Bearer <token>` 与 `Content-Type: application/json` 头。`base_url` 末尾不能为 `/` |
| `process(input_sentence)` | 方法 | POST `/api/conversation/process`，把用户文本交给 HA 对话引擎，返回 `speech.plain.speech` 字段中的回复文本 |
| `call_service(domain, service, data=None)` | 方法 | POST `/api/services/<domain>/<service>`，调用 HA 任意服务。是其它方法的基础 |
| `speak(sentence, tts='google_translate_say', media_player='all')` | 方法 | 调用 `tts.<tts>` 服务，将 `sentence` 播报到指定媒体播放器 |
| `note(message='', title='不能理解的语音命令')` | 方法 | 调用 `persistent_notification.create`，对无法理解的命令在 HA 前端留痕 |

**关键依赖**：`requests`

**典型调用链**（来自第 22 篇 DIY 智能音箱）：
```
麦克风 → speech_recognition（ASR）→ 文本 → ha_cli.process() → HA 对话 → 回复文本 → ha_cli.speak() → TTS 播放
```

---

### 4.2 `ais_cli.py` — 公共 AI 对话客户端

**文件**：[ais_cli.py](file:///workspace/ais_cli.py)

**职责**：在语音助手不能被 HA 理解时，调用图灵机器人开放平台获取开放式问答回复，提升交互体验。

#### 类 `tuling123`

| 成员 | 类型 | 说明 |
| --- | --- | --- |
| `__init__(user_id, api_key, base_url='http://openapi.tuling123.com/openapi/api/v2')` | 构造方法 | 初始化请求会话和基础请求体结构（`reqType/userInfo/perception.inputText`） |
| `command(input_sentence)` | 方法 | 填充 `perception.inputText.text` 后 POST，遍历 `results` 取首个 `resultType == 'text'` 的 `values.text` 作为回复 |

**关键依赖**：`requests`

> 备注：图灵机器人 v2 接口需在官网申请 `apiKey`，且对调用频率有限制。当无法解析为 HA 命令时可用此模块兜底。

---

### 4.3 `__init__.py` — 改写版语音识别库

**文件**：[`__init__.py`](file:///workspace/__init__.py)（根目录，作为包被导入为 `speech_recognition` 的修改版）

**来源**：基于开源 `speech_recognition 3.8.1`（作者 Anthony Zhang）改造。

**改造目的**：适配国内网络环境（google.cn）、接入 Snowboy 离线唤醒词、为 DIY 智能音箱提供更稳健的"热词 + 指令"录音流程。

#### 主要改造点（文件头部注释列出）

1. 新增 `Recognizer.recognize_google_cn`：连接到 `google.cn`（国内可用）
2. 修正 `Recognizer.snowboy_wait_for_hot_word()` 中对 Snowboy 的错误调用方式
3. `snowboy_wait_for_hot_word()` 敏感度由 `0.4` 调整为 `0.6`
4. `snowboy_wait_for_hot_word()` 增加动态 `energy_threshold` 调整
5. `listen()` 不把 Snowboy 唤醒词包含在返回的语音中
6. `listen()` 增加 `hot_word_callback` 参数，唤醒后异步回调
7. `listen()` 增加 `min_time_after_hot_word` 参数（默认 3 秒），热词后保证最少录音时长
8. `dynamic_energy_adjustment_damping` 由 `0.15` → `0.6`
9. `dynamic_energy_ratio` 由 `1.5` → `0.8`

#### 核心类与方法

| 类 / 函数 | 职责 |
| --- | --- |
| `AudioSource`（抽象基类） | 音频源抽象，定义上下文管理协议 |
| `Microphone(AudioSource)` | 基于 PyAudio 的麦克风输入；支持设备枚举、默认采样率获取 |
| `Microphone.get_pyaudio()` | 静态方法，校验 PyAudio ≥ 0.2.11 |
| `Microphone.list_microphone_names()` | 列出所有麦克风设备名 |
| `Microphone.list_working_microphones()` | 实际发声测试，返回可用麦克风索引→名称映射 |
| `AudioFile(AudioSource)` | 读取 WAV/AIFF/FLAC 文件，内部转换为统一 PCM 流 |
| `AudioData` | 一段已录制音频，支持 `get_segment/get_raw_data/get_wav_data/get_aiff_data/get_flac_data` |
| `Recognizer(AudioSource)` | 识别器，包含录音、VAD、多引擎识别 |
| `Recognizer.record(source, duration, offset)` | 按时长/偏移录制原始音频 |
| `Recognizer.adjust_for_ambient_noise(source, duration=1)` | 环境噪声校准，动态调整 `energy_threshold` |
| `Recognizer.snowboy_wait_for_hot_word(...)` | 阻塞等待 Snowboy 唤醒词，期间持续调整能量阈值 |
| `Recognizer.listen(source, timeout, phrase_time_limit, snowboy_configuration, min_time_after_hot_word=3, hot_word_callback=None)` | **核心录音方法**：检测到语音（或热词）后录制一句话，返回 `AudioData` |
| `Recognizer.listen_in_background(source, callback, phrase_time_limit=None)` | 后台线程持续监听并回调 |
| `Recognizer.recognize_sphinx / recognize_google / recognize_google_cn / recognize_bing / ...` | 各 ASR 引擎封装（CMU Sphinx 离线、Google 在线、Bing、Wit.ai、Houndify、IBM、Snowboy 等） |

#### 自定义异常

| 异常 | 触发场景 |
| --- | --- |
| `WaitTimeoutError` | 等待说话/热词超时 |
| `RequestError` | 识别 API 请求失败 |
| `UnknownValueError` | 音频可识别但无法解析为文本 |

**关键依赖**：`pyaudio`（麦克风）、`pocketsphinx`（离线识别，可选）、`snowboydetect`（唤醒词，可选）、`flac` 命令行工具（FLAC 编码，Google 识别需要）、标准库 `wave/aifc/audioop/urllib`。

---

## 5. HomeAssistant 自定义组件（custom_components）

### 5.1 `bus485_example` — 485 总线 Modbus 继电器开关

**位置**：[参考文档（21-30）/custom_components/bus485_example/](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/custom_components/bus485_example)

**对应章节**：第 25 篇《485 总线》——接入 Modbus 设备。

#### 文件清单

| 文件 | 作用 |
| --- | --- |
| `manifest.json` | HA 组件元信息：域名 `bus485_example`、依赖 `pyserial>=3.4`、`crcmod>=1.7` |
| `__init__.py` | 空文件（组件入口由 `switch.py` 的 `setup_platform` 完成） |
| `switch.py` | 开关平台实现：串口初始化、Modbus 帧构造、继电器控制与状态读取 |

#### 关键函数 / 类

| 成员 | 说明 |
| --- | --- |
| `crc_modbus(input_buffer)` | 计算 Modbus RTU 的 CRC16（多项式 `0x18005`，`initCrc=0xFFFF`，小端输出 2 字节） |
| `setup_platform(hass, config, add_entities, discovery_info=None)` | HA 开关平台入口：打开 `/dev/ttyUSB0`（9600 8N1，超时 3s），为地址 1 的线圈 0~3 各创建一个 `my_switch` 实体并注册 |
| `class my_switch(SwitchDevice)` | 单个继电器实体，封装 Modbus 写单线圈（`0x05`）与读线圈状态（`0x01`） |

#### `my_switch` 关键方法

- `turn_on()`：下发 `[addr, 0x05, 0x00, coil, 0xFF, 0x00] + CRC16`，置位对应线圈
- `turn_off()`：下发 `[addr, 0x05, 0x00, coil, 0x00, 0x00] + CRC16`，复位对应线圈
- `update()`：下发读线圈命令 `[addr, 0x01, 0x00, 0x00, 0x00, 0x10] + CRC16`，读取 7 字节响应，按位解析指定线圈状态

**关键依赖**：`pyserial`、`crcmod`、`homeassistant.components.switch.SwitchDevice`

> 注：该组件使用 HA 旧式 `setup_platform` 接口，在较新版本 HA（≥ 2021.x）中需要迁移到 `async_setup_platform` 或 `SwitchEntity` 基类；仅作教学示例使用。

---

## 6. 嵌入式 / 硬件代码

### 6.1 `LocReport` — Arduino + SIM7000 定位上报

**位置**：[参考文档（21-30）/LocReport/](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/LocReport)

**对应章节**：第 27 篇《室外物联网》——车载定位/全球卫星定位。

| 文件 | 职责 |
| --- | --- |
| `SIM7000.h` / `SIM7000.cpp` | SIMCom SIM7000（NB-IoT + GNSS）模块的 AT 指令封装类 `SIM7000` |
| `LocReport.ino` | 主程序：初始化串口、配置 NB-IoT 网络、开启 GNSS、周期性上报经纬度到 traccar 服务器 |

#### `SIM7000` 类（C++）

| 方法 | 说明 |
| --- | --- |
| `bool begin(Stream &port)` | 绑定串口流（通常是 `SoftwareSerial`），发送 `AT` 并等待 `OK` |
| `void sendCmd(const char* cmd)` | 向模块发送 AT 命令 |
| `int readBuffer(char *buffer, int count, unsigned int timeout, unsigned int chartimeout)` | 读取模块响应，支持字符间超时 |
| `bool check_send_cmd(const char* cmd, const char* resp, ...)` | 发送命令并检查响应中是否包含期望字符串 `resp` |

#### `LocReport.ino` 关键流程

1. `Serial` 115200 打印日志，`SoftwareSerial(RXD2=10, TXD2=11)` 19200 连接 SIM7000
2. `sim7000.begin(mySS)` → 检测 SIM 卡（`AT+CPIN?`）
3. 选择 LTE/NB-IoT 模式（`AT+CNMP=38`、`AT+CMNB=2`、`AT+NBSC=1`）
4. 设置 APN（`AT+CSTT`）、激活 PDP（`AT+CIICR`）
5. 打开 GNSS 电源（`AT+CGNSPWR=1`）
6. 主循环：`AT+CGNSINF` 读取经纬度 → 通过 TCP 上报到 traccar（`ServerIP:ServerPort`，HTTP 协议）

**关键依赖**：Arduino 框架、`SoftwareSerial`、SIM7000 模块硬件。

---

### 6.2 ESP32 I2S 音频示例（第 24 篇 远程麦克风）

| 目录 | 作用 |
| --- | --- |
| `example_24_audio_input/` | ESP32 通过 I2S PDM 模式采集麦克风数据，16kHz/16bit，经串口输出原始采样值 |
| `example_24_audio_output/` | ESP32 通过 I2S 输出音频（DAC/外部功放） |

**关键技术点**：
- `i2s_config_t` 配置：`I2S_MODE_MASTER | I2S_MODE_RX | I2S_MODE_PDM`
- `i2s_pin_config_t` 指定 WS、DATA 引脚（示例中 `GPIO_NUM_19`、`GPIO_NUM_22`）
- `i2s_driver_install` / `i2s_set_pin` / `i2s_read` 完成采集循环

**关键依赖**：ESP-IDF 或 Arduino-ESP32 的 `driver/i2s.h`。

---

## 7. YAML 配置示例

各 `参考文档（N-M）/` 目录中包含的 `example_*.yaml` 是 Home Assistant `configuration.yaml` 的片段，演示不同功能。具有代表性的样例：

| 样例文件 | 所属篇 | 演示内容 |
| --- | --- | --- |
| `example_6_5_1.yaml` | 6. 自动化 | `input_number` + `automation` 实现"亮灯 N 秒"，含模板 delay 与分组视图 |
| `example_15_2_1.yaml` | 15. GPIO | 通过 `shell_command` 操作 `/sys/class/gpio` + `binary_sensor.command_line` + `light.template` 把树莓派 GPIO17 包装成灯 |
| `example_7_2_1 / 7_4_1 / 7_5_1` | 7. 设备接入 | 太阳/天气/红外遥控等传感器与开关接入 |
| `example_8_6_1` | 8. 摄像头 | 摄像头组件配置 |
| `example_9_2_1 ~ 9_6_1` | 9. 人脸识别 | DLib / 微软 / Facebox 人脸识别接入 |
| `example_10_2_1` | 10. 设备接入 | YeeLight / SystemMonitor / 百度 TTS |
| `example_11_4_1` | 11. HomeKit | HomeKit 桥接与 Siri 控制 |
| `example_13_2_1.dash`、`13_3_*.yaml`、`13_3_*.dash` | 13. AppDaemon | DashBoard 面板与 App 配置 |
| `example_16_*.yaml`（若有） | 16. MQTT | MQTT 灯、传感器、自动化 |
| `example_24_*.ino` 等非 yaml | 24. 远程麦克风 | 见第 6 节 |

**YAML 配置的通用运行方式**：将对应片段合并到 HA 的 `configuration.yaml`，或使用 `packages` 机制引入，然后在 HA 中执行"配置 → 通用 → 重新启动"或"重新加载 YAML 配置"。

---

## 8. 依赖关系分析

### 8.1 Python 模块依赖

```
ha_cli.py  ────────► requests
ais_cli.py ────────► requests

__init__.py (改写版 speech_recognition)
   ├─► pyaudio (Microphone)
   ├─► pocketsphinx (recognize_sphinx, 可选)
   ├─► snowboydetect (snowboy_wait_for_hot_word, 可选)
   ├─► flac (命令行, recognize_google 需要 FLAC 编码)
   └─► 标准库: wave, aifc, audioop, urllib, io, math, threading...

custom_components/bus485_example
   ├─► homeassistant.components.switch.SwitchDevice
   ├─► pyserial>=3.4
   └─► crcmod>=1.7
```

### 8.2 外部系统/服务依赖

| 模块 | 外部依赖 |
| --- | --- |
| `ha_cli.py` | 运行中的 Home Assistant 实例 + 长期访问令牌（Long-Lived Access Token） |
| `ais_cli.py` | 图灵机器人开放平台 `apiKey` + `userId` |
| `__init__.py` | Google 语音识别（需联网 / 代理）、Snowboy 模型文件（`.pmdl/.umdl`） |
| `bus485_example` | 485 转 USB 设备 `/dev/ttyUSB0` + Modbus RTU 继电器模块 |
| `LocReport.ino` | SIM7000 模块、NB-IoT SIM 卡、traccar 服务器 |
| ESP32 音频示例 | ESP32 开发板 + I2S/PDM 麦克风/功放硬件 |

### 8.3 模块间调用关系

DIY 智能音箱（第 22 篇）的主程序（未在仓库中显式给出，但由以下模块拼装）典型调用关系：

```
                  ┌──────────────────────────┐
                  │   改写版 speech_recognition │
                  │   (Microphone / Recognizer) │
                  └─────────────┬────────────┘
                                │ 语音文本
                                ▼
            ┌───────────────────────────────────────┐
            │            主程序（用户编写）            │
            └───┬───────────────────────────────┬───┘
                │ 可解析为 HA 命令               │ 不可解析
                ▼                                ▼
        ┌──────────────┐                ┌──────────────────┐
        │  ha_cli.py   │                │   ais_cli.py     │
        │ (HA REST API)│                │ (tuling123 对话) │
        └──────┬───────┘                └────────┬─────────┘
               │ 回复文本                          │ 回复文本
               └──────────────┬───────────────────┘
                              ▼
                    ha_cli.speak() → TTS → 音箱播放
```

---

## 9. 项目运行方式

仓库本身不是单一可执行项目，而是"教程 + 示例代码"集合。以下按模块说明运行前提与方法。

### 9.1 Home Assistant 环境准备

1. 按第 1 篇在树莓派 / x86 上安装 Home Assistant（虚拟环境或 Docker）。
2. 取得长期访问令牌：HA 头像 → 安全 → 长期访问令牌 → 创建。
3. 记录 HA 访问地址（默认 `http://<ip>:8123`）。

### 9.2 运行 `ha_cli.py` / `ais_cli.py`

```bash
# 安装依赖
pip install requests

# 在 Python 中使用
python3 -c "
from ha_cli import ha_cli
ha = ha_cli(token='<YOUR_LONG_LIVED_TOKEN>', base_url='http://localhost:8123')
print(ha.process('把客厅灯打开'))
ha.speak('你好，欢迎回家')
"
```

```python
from ais_cli import tuling123
ai = tuling123(user_id='user001', api_key='<YOUR_TULING_API_KEY>')
print(ai.command('今天天气怎么样'))
```

### 9.3 运行改写版语音识别库

将根目录 `__init__.py` 作为 `speech_recognition` 包使用（把包含该文件的目录加入 `PYTHONPATH` 并将其命名为 `speech_recognition`，或直接 `import __init__` 后取 `Recognizer`）：

```python
from speech_recognition import Recognizer, Microphone  # 即本仓库改写版

r = Recognizer()
with Microphone() as src:
    r.adjust_for_ambient_noise(src)
    audio = r.listen(src, snowboy_configuration=('/path/to/snowboy', ['model.pmdl']),
                     hot_word_callback=lambda: print('听到唤醒词'))
text = r.recognize_google_cn(audio, language='zh-CN')
```

前置依赖：
```bash
pip install pyaudio pocketsphinx
# Snowboy 需自行编译并提供 snowboydetect.py 与模型文件
# flac 命令：sudo apt install flac
```

### 9.4 安装 `bus485_example` 自定义组件

1. 将 `bus485_example/` 目录整个拷贝到 HA 配置目录下的 `custom_components/`：
   ```
   <HA_CONFIG>/custom_components/bus485_example/
   ```
2. 在 `configuration.yaml` 中启用开关平台：
   ```yaml
   switch:
     - platform: bus485_example
   ```
3. 重启 Home Assistant，前端将出现 `switch.my_relay_1_0` ~ `switch.my_relay_1_3` 四个实体。

> 需先安装运行时依赖：`pip install pyserial crcmod`（HA 环境内）。

### 9.5 烧录嵌入式示例

| 示例 | 工具链 | 步骤 |
| --- | --- | --- |
| `LocReport.ino` | Arduino IDE（安装 SIM7000 所需库，或直接使用自带 `SoftwareSerial`） | 配置 `NetworkAPN / ServerIP / ServerPort / DeviceID`，选择 Arduino 板型与串口，编译上传 |
| `example_24_audio_input.ino` / `_output.ino` | Arduino-ESP32 或 ESP-IDF | 选择 ESP32 板型，按接线连接 I2S 设备，编译上传 |

### 9.6 文档阅读

各 `参考文档（N-M）/*.pdf` 可直接阅读；`*.docx` 可用 Word/WPS 打开。对应章节的 YAML 片段可直接粘贴到 HA 配置中验证。

---

## 10. 架构总览与数据流

### 10.1 整体分层

```
┌──────────────────────────────────────────────────────────────┐
│                        教程文档层                              │
│   参考文档（1-10/11-20/21-30）/*.pdf, *.docx                   │
│   → 讲解概念、原理、操作步骤                                    │
├──────────────────────────────────────────────────────────────┤
│                      配置示例层（YAML）                        │
│   example_*.yaml / *.dash                                     │
│   → HA 自动化、传感器、灯光、MQTT、AppDaemon 面板              │
├──────────────────────────────────────────────────────────────┤
│                    Python 业务模块层                           │
│   ha_cli.py（HA 客户端）                                       │
│   ais_cli.py（AI 对话）                                        │
│   __init__.py（改写语音识别）                                   │
│   custom_components/bus485_example（HA 集成）                  │
├──────────────────────────────────────────────────────────────┤
│                    嵌入式 / 硬件层                             │
│   LocReport（Arduino + SIM7000）                               │
│   example_24_audio_input/output（ESP32 I2S）                  │
└──────────────────────────────────────────────────────────────┘
```

### 10.2 典型数据流（DIY 智能音箱）

```
[人说话]
   │
   ▼
[Microphone/PyAudio] ──PCM──► [Recognizer.listen + Snowboy 热词]
   │
   ▼ AudioData
[recognize_google_cn] ──HTTP──► [Google 语音识别]
   │
   ▼ 文本
[主程序] ──HA 命令──► [ha_cli.process/call_service] ──HTTP──► [Home Assistant]
   │                                                            │
   └─非 HA 命令─► [ais_cli.command] ──HTTP──► [图灵机器人]      │
                                                          │
                                                          ▼
[ha_cli.speak] ◄──── 回复文本 ──────────────────────────┘
   │
   ▼
[TTS 服务] ──► [媒体播放器/音箱] 播放语音
```

### 10.3 485 总线数据流

```
[HA 前端/自动化] ──turn_on/off──► [my_switch]
                                        │
                                        ▼
                              [Modbus 帧 + CRC16]
                                        │
                                        ▼
                              [pyserial /dev/ttyUSB0]
                                        │
                                        ▼
                              [485 继电器模块]
```

---

## 11. 扩展与维护建议

1. **`ha_cli.py` 健壮性**
   - 增加 `request` 超时与异常捕获（`requests.exceptions`），避免 HA 不可达时主程序崩溃。
   - `process()` 当前直接取 `r.json()['speech']['plain']['speech']`，应检查 HTTP 状态码与字段存在性。
   - 可补充 `get_state(entity_id)`、`fire_event` 等常用接口。

2. **`ais_cli.py`**
   - 图灵 v2 接口可能已下线或收费，可替换为百度智能云、讯飞、ChatGPT 等对话 API，保持 `command(text) -> str` 签名即可无缝替换。

3. **改写版 `speech_recognition`**
   - 该文件较大（单文件超 50KB），建议拆分为 `audio.py / recognizer.py / exceptions.py`。
   - 对 Snowboy 的依赖属于动态 `sys.path.append`，可在构造 `Recognizer` 时显式传入路径，避免全局副作用。
   - `Pool(processes=1)` 仅用于异步触发 `hot_word_callback`，可替换为 `threading.Thread` 以减少进程开销。

4. **`bus485_example`**
   - 升级到 HA 新版集成规范：使用 `SwitchEntity`、`async_setup_platform`、`config_flow`、`PLATFORMS = ["switch"]`。
   - 将串口设备、波特率、地址、线圈数量从硬编码改为 YAML 配置或 UI 配置项。
   - 对读取失败增加重试与日志分级。

5. **嵌入式代码**
   - `LocReport.ino` 中 `DeviceID`、`ServerIP` 等为硬编码宏，可迁移到 EEPROM / Preferences 存储，支持运行时配置。
   - 增加 GNSS 定位有效性判断（`AT+CGNSINF` 第 3 字段 `fix`），无效时不上报。

6. **文档同步**
   - Home Assistant 版本迭代快，旧式 `setup_platform`、`SwitchDevice` 等 API 在新版 HA 已移除；教程示例若要在新版运行需同步更新。
   - 建议在每篇文档中标注"最后验证的 HA 版本"。

---

## 附录 A：核心文件索引

| 文件 | 路径 |
| --- | --- |
| HA 客户端 | [ha_cli.py](file:///workspace/ha_cli.py) |
| AI 对话客户端 | [ais_cli.py](file:///workspace/ais_cli.py) |
| 改写语音识别库 | [`__init__.py`](file:///workspace/__init__.py) |
| 485 组件入口 | [switch.py](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/custom_components/bus485_example/switch.py) |
| 485 组件清单 | [manifest.json](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/custom_components/bus485_example/manifest.json) |
| SIM7000 驱动头文件 | [SIM7000.h](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/LocReport/SIM7000.h) |
| SIM7000 驱动实现 | [SIM7000.cpp](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/LocReport/SIM7000.cpp) |
| 定位上报主程序 | [LocReport.ino](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/LocReport/LocReport.ino) |
| ESP32 麦克风采集 | [example_24_audio_input.ino](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8821-30%EF%BC%89/example_24_audio_input/example_24_audio_input.ino) |
| GPIO 灯 YAML 示例 | [example_15_2_1.yaml](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%8811-20%EF%BC%89/example_15_2_1.yaml) |
| 自动化亮灯 YAML 示例 | [example_6_5_1.yaml](file:///workspace/%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3%EF%BC%881-10%EF%BC%89/example_6_5_1.yaml) |

---

> 本文档基于仓库当前状态生成。如后续新增章节或代码示例，建议在对应章节下补充条目并同步更新"核心文件索引"。
