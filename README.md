# 工业数据采集网关 v1.0.0

[![Docker Build](https://github.com/qinshihu/FactoryLink/actions/workflows/docker-build.yml/badge.svg)](https://github.com/qinshihu/FactoryLink/actions/workflows/docker-build.yml)
[![Release Build](https://github.com/qinshihu/FactoryLink/actions/workflows/release.yml/badge.svg)](https://github.com/qinshihu/FactoryLink/actions/workflows/release.yml)
[![License](https://img.shields.io/badge/license-MPL--2.0-blue.svg)](./LICENSE)

[English](./README.en.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [Deutsch](./README.de.md) | [Français](./README.fr.md) | [Español](./README.es.md) | [Русский](./README.ru.md) | [Português](./README.pt.md)

> 国内镜像：[Gitee](https://gitee.com/IT_Oline/FactoryLink) | [GitCode](https://gitcode.com/gcw_IM7Aihp/FactoryLink)

## 这是什么？

一个**单EXE、双击就能跑、零依赖、完全开源免费**的工业设备数据采集网关。

专门给国内制造业的一线工程师、工厂IT、小集成商使用。

## 为什么做这个？

- 某讯的采集网关卖1万块钱一台，我做个免费的给大家用
- 其他开源项目都要Docker，车间工程师根本不会用
- 所有国外项目都没有中文文档，出了问题找不到人问
- 引入了AI配置助手，支持用自然语言描述设备，AI自动生成配置

## 怎么用？

1. [下载 `工业数据采集网关.exe`](https://github.com/qinshihu/FactoryLink/releases/latest)

2. 双击运行（右下角会出现托盘图标）

![托盘图标](images/1(2).png)

3. 浏览器自动打开配置页面（默认 `http://localhost:8000`）

![配置页面](images/15.png)
![配置页面](images/1(1).png)
![配置页面](images/12.png)
![配置页面](images/qqq.png)

 4. 配置你的PLC IP、点位表、MQTT地址  

  ![设备配置](images/1(4).png)
  ![设备配置](images/qq.png)

5. 点"启动采集"，完事了

> 右键托盘图标可以：打开配置页面、启动/停止采集、退出程序。
> 
> 也可以用 Docker 部署：[Docker 部署说明](#docker-部署)

## 支持的协议

| 协议 | 支持型号 | 依赖库 |
|------|---------|--------|
| Modbus TCP | 所有标准Modbus TCP设备 | pymodbus 3.x |
| Modbus RTU | 所有标准Modbus RTU设备（串口） | pymodbus 3.x |
| 西门子S7 | S7-1200 / S7-1500 / S7-300 / S7-400 | python-snap7 3.0（纯Python） |
| 三菱MC | FX5U / Q系列 / L系列 | pymcprotocol |

## 核心功能

- **单EXE启动**：双击运行，不需要装任何运行环境
- **Web配置界面**：浏览器打开就能配置，不需要懂命令行
- **实时数据查看**：WebSocket推送，数据实时刷新
- **MQTT转发**：自动将采集数据转发到MQTT服务器
- **AI配置助手**：用自然语言描述设备，AI自动生成配置（支持 OpenAI / 通义千问 / DeepSeek 等）
- **Excel导入**：支持从Excel表格批量导入点位配置，导入前可预览确认
- **设备连接测试**：一键测试PLC连通性
- **断线重连**：网络断开自动重连，指数退避策略
- **开机自启动**：一键设置Windows开机自启动
- **系统托盘**：后台运行，右键托盘图标操作
- **日志查看**：Web界面直接查看采集日志，支持按级别筛选
- **配置热加载**：改完配置点"应用"，自动重启采集器
- **配置自动备份**：每次保存自动生成 `config.json.bak`
- **端口冲突处理**：8000被占用自动换8001、8002...

## 界面说明

| 页面 | 功能 |
|------|------|
| **首页** | 设备卡片列表、实时数据展示、采集启停按钮、设备在线状态、MQTT连接状态 |
| **设备配置** | 添加/编辑/删除设备、按协议显示配置项、点位表格增删改、Excel导入（预览确认）、模板下载、测试连接、**AI配置助手**（自然语言生成设备配置） |
| **系统设置** | MQTT配置、采集间隔、重连策略、开机自启动、**AI配置**（API地址/Key/模型）、日志查看（支持级别筛选） |

## 数据格式

### MQTT数据Topic

```
{topic_prefix}/{device_id}
```

示例：`factory/gateway-001/dev1`

### MQTT数据报文

```json
{
  "gateway": "车间1号网关",
  "device_id": "dev1",
  "device_name": "西门子S7-1200",
  "timestamp": 1719123456789,
  "values": {
    "温度1": {"value": 25.5, "unit": "℃", "quality": "good"},
    "压力1": {"value": 1.2, "unit": "MPa", "quality": "good"}
  }
}
```

### 状态Topic

```
{topic_prefix}/{device_id}/status
```

```json
{
  "device_id": "dev1",
  "status": "online",
  "message": "连接成功",
  "timestamp": 1719123456789
}
```

- status取值：`online`（正常采集）、`offline`（断线）、`error`（异常）
- quality取值：`good`（正常）、`bad`（读取失败）、`uncertain`（数据可疑）

## 点位数据类型

| 类型 | 说明 | 字节数 |
|------|------|--------|
| bool | 布尔量 | 1 bit |
| int16 | 16位有符号整数 | 2 |
| uint16 | 16位无符号整数 | 2 |
| int32 | 32位有符号整数 | 4 |
| uint32 | 32位无符号整数 | 4 |
| float | 32位浮点数 | 4 |
| double | 64位浮点数 | 8 |

> 实际值 = 原始值 × 倍率 + 偏移量。倍率和偏移量在点位配置中设置。

## 配置文件

所有配置保存在与EXE同目录的 `config.json` 文件中，修改后自动备份为 `config.json.bak`。

```json
{
  "gateway_name": "车间1号网关",
  "devices": [
    {
      "id": "dev1",
      "name": "西门子S7-1200",
      "protocol": "s7",
      "ip": "192.168.1.100",
      "rack": 0,
      "slot": 1,
      "enabled": true,
      "points": [
        {"name": "温度1", "address": "DB1.DBD0", "type": "float", "rate": 1.0, "offset": 0.0, "unit": "℃"}
      ]
    }
  ],
  "mqtt": {
    "host": "192.168.1.200",
    "port": 1883,
    "client_id": "gateway-001",
    "topic_prefix": "factory/gateway-001",
    "username": "",
    "password": "",
    "qos": 1,
    "enabled": true
  },
  "collect_interval": 1000,
  "reconnect": {
    "max_retries": 0,
    "base_delay": 1,
    "max_delay": 60
  },
  "ai": {
    "enabled": false,
    "api_url": "https://api.openai.com/v1",
    "api_key": "",
    "model": "gpt-3.5-turbo"
  }
}
```

- `collect_interval`：采集间隔，单位毫秒，1000 = 每秒采集一次
- `reconnect.max_retries`：0 = 无限重试
- 重试间隔：指数退避，1s → 2s → 4s → 8s → 16s → 32s → 60s（封顶）

## 协议地址格式

### Modbus

| 地址范围 | 区域 | 示例 |
|---------|------|------|
| 40001-49999 | 保持寄存器 | `40001` |
| 30001-39999 | 输入寄存器 | `30001` |
| 10001-19999 | 离散输入 | `10001` |
| 00001-09999 | 线圈 | `00001` |

### 西门子S7

| 格式 | 说明 | 示例 |
|------|------|------|
| DBx.DBDy | DB块双字（32位） | `DB1.DBD0` |
| DBx.DBXy.z | DB块位 | `DB1.DBX8.0` |
| DBx.DBWy | DB块字（16位） | `DB1.DBW0` |
| Mx.y | 内存位 | `M0.0` |
| Ix.y | 输入位 | `I0.0` |
| Qx.y | 输出位 | `Q0.0` |

> S7-1200/1500：rack=0, slot=1；S7-300/400：rack=0, slot=2

### 三菱MC

| 格式 | 说明 | 示例 |
|------|------|------|
| Dxxxx | 数据寄存器 | `D100` |
| Mxxxx | 内部继电器 | `M100` |
| Xx | 输入继电器 | `X0` |
| Yx | 输出继电器 | `Y0` |
| Wxxxx | 链接寄存器 | `W100` |

## Excel点位表格式

| 点位名称 | 地址 | 数据类型 | 倍率 | 偏移 | 单位 |
|---------|------|---------|------|------|------|
| 温度1 | DB1.DBD0 | float | 1.0 | 0 | ℃ |
| 压力1 | DB1.DBD4 | float | 1.0 | 0 | MPa |
| 运行状态 | DB1.DBX8.0 | bool | 1.0 | 0 | - |

- 第一行为表头（固定格式），第二行开始为点位数据
- 支持 `.xlsx` 和 `.xls` 格式
- 在设备配置页可下载模板
- 导入后会弹出预览对话框，确认后自动填入添加设备表单

## 从源码运行

```bash
# 1. 创建虚拟环境（推荐）
python -m venv venv
venv\Scripts\activate

# 2. 安装Python依赖
pip install -r requirements.txt

# 3. 编译前端
cd frontend
npm install
npm run build
cd ..

# 4. 启动后端
cd backend
python main.py
```

浏览器打开 `http://localhost:8000`。

## 打包成EXE

```bash
# 确保前端已编译
cd frontend && npm run build && cd ..

# 执行打包
build.bat
```

输出文件：`dist/工业数据采集网关.exe`（约25MB）

## Docker 部署

如果不想用 EXE，也可以用 Docker 方式运行（适合部署在服务器、工控机、树莓派上）。

### 快速启动

```bash
# 拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/huluwa666/tsq-images-hub:factorylink-latest

# 创建目录存放配置和日志
mkdir -p /opt/factorylink/{logs,config}

# 运行容器
docker run -d \
  --name factorylink \
  --restart always \
  -p 8000:8000 \
  -v /opt/factorylink/config:/app/config \
  -v /opt/factorylink/logs:/app/logs \
  registry.cn-hangzhou.aliyuncs.com/huluwa666/tsq-images-hub:factorylink-latest
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `-p 8000:8000` | 映射 Web 配置页面端口 |
| `-v /opt/factorylink/config:/app/config` | 挂载配置目录（config.json 保存在这里） |
| `-v /opt/factorylink/logs:/app/logs` | 挂载日志目录 |
| `--restart always` | 容器异常退出自动重启 |

### 首次使用

1. 容器启动后，浏览器打开 `http://你的服务器IP:8000`
2. 配置 PLC 设备和 MQTT
3. 点击"启动采集"

> 注意：Docker 版本**不支持**系统托盘图标和开机自启动功能（这两个是 Windows 专属）。

### 使用 docker-compose

创建 `docker-compose.yml`：

```yaml
version: '3'
services:
  factorylink:
    image: registry.cn-hangzhou.aliyuncs.com/huluwa666/tsq-images-hub:factorylink-latest
    container_name: factorylink
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
```

启动：

```bash
docker-compose up -d
```

## 技术栈

- **后端**：Python 3.11+ / FastAPI / WebSocket / uvicorn
- **前端**：Vue 3 / Vite / Element Plus / xlsx
- **协议库**：pymodbus 3.x / python-snap7 3.0 / pymcprotocol
- **MQTT**：paho-mqtt
- **AI**：httpx（OpenAI 兼容接口，支持通义千问、DeepSeek 等）
- **Excel**：openpyxl（服务端解析）
- **打包**：PyInstaller 6.x（`--onefile --windowed`）
- **托盘**：pystray + Pillow

## 项目结构

```
FactoryLink/
├── backend/
│   ├── main.py              # FastAPI入口（WebSocket、托盘、端口检测、Excel导入）
│   ├── config.py            # 配置管理（读写、备份、线程安全）
│   ├── logger.py            # 日志管理（10MB轮转×5、API读取）
│   ├── schemas.py           # Pydantic数据模型
│   ├── ai_service.py        # AI配置助手（OpenAI兼容接口、流式输出）
│   ├── collector/
│   │   ├── base.py          # 采集器基类（指数退避重连、后台重连线程）
│   │   ├── modbus.py        # Modbus TCP/RTU（pymodbus）
│   │   ├── s7.py            # 西门子S7（python-snap7 3.0）
│   │   └── mitsubishi.py    # 三菱MC（pymcprotocol）
│   └── forwarder/
│       └── mqtt.py          # MQTT转发（paho-mqtt）
├── frontend/
│   ├── src/
│   │   ├── App.vue          # 布局框架（导航、Logo、作者信息）
│   │   ├── router.js        # 路由配置
│   │   └── views/
│   │       ├── Home.vue          # 首页（设备卡片、实时数据、WebSocket）
│   │       ├── DeviceConfig.vue  # 设备配置（增删改、点位管理、Excel导入）
│   │       └── Settings.vue      # 系统设置（MQTT、采集、重连、自启动、日志）
│   └── dist/                # 编译后的静态文件
├── build.bat                # PyInstaller打包脚本
├── requirements.txt         # Python依赖清单
├── Dockerfile               # Docker镜像
└── README.md                # 本文件
```

## 架构说明与 EXE 打包机制

### 整体架构

FactoryLink 是面向工业现场的单机边缘数据采集网关，采用“Vue 单页应用 + FastAPI 单体后端 + 协议采集器插件 + MQTT 转发”的架构。前端负责设备、点位和系统配置；后端直接与 PLC 通信，将采集结果通过 WebSocket 推送到浏览器，并按设备转发至 MQTT。

```text
Vue 3 + Vite + Element Plus SPA
  ├─ REST API：配置、设备、采集控制、日志、Excel、AI
  ├─ WebSocket：实时数据与设备状态
  └─ SSE：AI 配置助手流式结果
                 │
                 ▼
FastAPI（backend/main.py）
  ├─ 配置管理：config.json / config.json.bak
  ├─ 采集调度：采集器生命周期、设备状态、WebSocket 广播
  ├─ MQTT 转发：数据主题与状态主题
  ├─ 运维能力：日志、Excel 导入、连接测试、系统托盘
  └─ AI 服务：OpenAI 兼容接口
                 │
                 ▼
BaseCollector 抽象层
  ├─ Modbus TCP / RTU
  ├─ Siemens S7
  └─ Mitsubishi MC
```

#### 前端层

- Vue 3 SPA 提供首页、设备配置和系统设置三个页面；开发环境由 Vite 将 `/api` 与 `/ws` 代理到本地后端，生产环境由 FastAPI 直接托管构建产物。
- 首页使用 REST 获取设备和运行状态，使用 WebSocket 接收实时点位和状态变化；AI 配置助手使用 SSE 接收模型的流式输出。

#### 后端与采集层

- `backend/main.py` 是应用组合入口，负责 FastAPI、REST API、WebSocket、采集调度、系统托盘、端口检测及静态文件服务。
- `BaseCollector` 定义连接、断开和单点读取等统一协议接口，并集中提供状态通知、指数退避和统一数据封装；新协议可通过新增 collector 子类并在采集器工厂中登记来扩展。
- Modbus、S7 和 Mitsubishi 采集器分别完成设备连接、地址解析、字节序/数据类型解码，以及倍率和偏移量处理。

#### 基础设施层

- 配置以 JSON 文件保存，每次保存前自动生成 `config.json.bak`；日志采用 10 MB × 5 份的轮转文件策略。
- MQTT 按 `{topic_prefix}/{device_id}` 发布采集数据，并按 `{topic_prefix}/{device_id}/status` 发布设备状态。

### 架构优点

1. **便于现场交付。** 单体程序把浏览器 UI、后端 API 和工业协议通信整合到一个部署单元，适合设备量有限、工程师需要快速安装和排障的车间场景。
2. **协议扩展边界清晰。** 公共连接、重连、状态和数据结构逻辑位于 `BaseCollector`，协议实现只需关注通信与解析逻辑。
3. **实时链路完整。** PLC 数据可同时进入 WebSocket 和 MQTT，既满足本地实时查看，也满足上游平台订阅。
4. **运维友好。** 提供配置备份、日志查看、Excel 点表导入、连通性测试、热应用配置和系统托盘等现场工具。

### 当前限制与改进方向

1. **优先修复断线重连控制流。** 采集循环当前会跳过未连接的采集器；应确保断线设备仍能触发后台重连，或将重连提升到独立设备 worker。
2. **采集是单线程串行的。** 一台 PLC 的慢响应会拖慢其他设备，且大量点位会产生大量单点请求。建议按设备隔离采集任务，并对连续地址实施批量读取。
3. **启停操作需状态机化。** 当前启停和应用配置使用后台线程，建议用 `stopped / starting / running / stopping / failed` 状态机及单一控制队列避免并发竞争。
4. **MQTT 可靠性仍可增强。** 建议增加连接/断连回调、退避重连、Last Will、发布结果检查及按需的本地离线队列。
5. **配置安全与校验需要加强。** MQTT 密码和 AI Key 当前保存在本地配置中；正式部署应对接口输出做脱敏，并考虑系统凭据库或 Docker Secret。协议、端口、地址和采集周期也应在保存前做更严格的结构化校验。
6. **补充自动化测试。** 建议优先覆盖地址解析、字节序解码、倍率偏移、重连状态机、配置备份和 FastAPI API 契约，再增加模拟 PLC/MQTT 的集成测试。

### 与 Fledge 的对比与可借鉴方向

[Fledge](https://github.com/fledge-iot/fledge) 是更偏向通用工业/物联网边缘平台的开源项目：它以服务和插件为主要扩展单元，将南向采集、过滤/处理、北向发送、通知和持久化能力解耦。FactoryLink 则以“Windows 单 EXE、浏览器配置、快速采集 PLC 并转发 MQTT”为重点。两者目标不同：FactoryLink 不应直接照搬 Fledge 的部署复杂度，但很值得吸收其稳定的扩展边界和运行治理思路。

| 维度 | FactoryLink 当前方式 | Fledge 可借鉴的思路 | 建议的调整 |
|------|---------------------|---------------------|------------|
| 产品定位 | 单机、单体、快速交付，优先服务 Windows 现场工程师。 | 面向长期运行、异构设备和多种北向系统的边缘平台。 | 保留单 EXE 产品形态；不要为了“微服务化”牺牲安装体验。 |
| 南向协议 | 以 `BaseCollector` 加三个协议实现扩展，采集器由主进程直接管理。 | 通过明确的南向插件契约接入协议和设备。 | 将 collector 工厂改为可注册的插件注册表；为协议实现定义能力、配置 schema、连接/健康检查和版本信息。 |
| 数据处理 | 读取后直接推送 WebSocket 和 MQTT，只有倍率/偏移等点位转换。 | 将采集、过滤/转换、北向发送拆为可组合的数据管道。 | 先在进程内增加轻量 pipeline：`采集 → 标准化 → 过滤/聚合/告警 → 输出`；不必立即拆成独立进程。 |
| 北向输出 | MQTT 写在单例 `MqttForwarder` 中。 | 北向连接器可按目标系统独立扩展与配置。 | 抽象 `Forwarder` 接口，保留 MQTT 默认实现；后续可增加 HTTP、OPC UA、数据库或其他工业平台适配器。 |
| 可靠性 | 单采集线程串行读取；断线和消息丢失的处理能力有限。 | 服务级健康检查、独立生命周期、持久化和失败隔离。 | 为每台设备建立独立 worker 与健康状态；加入输出队列、限流、失败重试、死信/落盘策略，并暴露运行指标。 |
| 运维与配置 | 本地 JSON 文件、日志文件和 Web UI。 | 统一配置、审计、服务状态和插件生命周期管理。 | 为配置加版本号与迁移机制；记录配置变更审计；在 UI 和 API 中展示设备最后成功时间、失败次数、读取耗时和队列积压。 |

#### 推荐演进顺序

1. **先修正运行可靠性，而不是拆服务。** 修复断线重连路径、按设备隔离采集任务、避免在锁内进行网络 I/O，并补充单元/集成测试。
2. **再抽象稳定接口。** 定义 `Collector`、`Processor`、`Forwarder` 三类内部接口；现有 Modbus/S7/Mitsubishi 和 MQTT 实现分别迁移到这些接口之后。这样可以获得 Fledge 式的可扩展性，同时仍保持单 EXE 部署。
3. **引入轻量的进程内管道。** 先支持点位过滤、死区、数据转换、聚合和简单规则告警；该阶段不要求插件进程或外部服务。
4. **最后才考虑可选插件包或多进程隔离。** 仅在需要第三方协议插件、设备数量明显增长、单个协议库不稳定，或需要隔离不同客户扩展时，再引入 Fledge 风格的独立插件进程和服务管理。

#### 不建议直接照搬的部分

- 不建议立即将当前应用拆成多个常驻服务或要求 Docker/容器编排；这会直接削弱 FactoryLink 的“下载后双击即可运行”优势。
- 不建议一开始就实现通用插件市场、复杂资产模型或全量时序数据库。应以现场真实需求驱动：优先保证采集不中断、数据可追溯、故障可定位。
- 不建议将 Fledge 当作“替代库”嵌入本项目；更合适的关系是把它作为架构参考，选择性借鉴其插件边界、数据管道和运行治理理念。

### 如何生成 Windows EXE

项目通过“先构建前端，再由 PyInstaller 打包 Python 后端和前端资源”的方式生成单文件 EXE：

```text
Vue 源码 --npm run build--> frontend/dist
                                │
Python 后端 + 第三方依赖 + frontend/dist
                                │
                         PyInstaller
                                │
                                ▼
                 dist/工业数据采集网关.exe
```

#### 构建步骤

应在 Windows 环境中完成构建：

```bat
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
pip install pyinstaller
build.bat
```

`build.bat` 会自动执行 `npm install` 和 `npm run build`，随后调用：

```bat
pyinstaller --onefile --windowed ^
    --add-data "frontend/dist;frontend/dist" ^
    --collect-all fastapi ^
    --collect-all uvicorn ^
    --collect-all pymodbus ^
    --collect-all paho.mqtt.client ^
    --collect-all websockets ^
    --collect-all pystray ^
    --collect-all httpx ^
    --collect-all openpyxl ^
    --collect-all python_multipart ^
    --hidden-import snap7 ^
    --hidden-import pymcprotocol ^
    --name "工业数据采集网关" ^
    --icon "icon.ico" ^
    backend/main.py
```

参数说明：

| 参数 | 作用 |
|------|------|
| `--onefile` | 生成单个 EXE 文件。 |
| `--windowed` | 以 Windows GUI 模式启动，不显示传统控制台窗口。 |
| `--add-data "frontend/dist;frontend/dist"` | 将 Vue 构建后的静态资源嵌入 EXE。 |
| `--collect-all` | 收集 FastAPI、Uvicorn、协议库等包的模块和资源，避免动态导入造成运行时缺包。 |
| `--hidden-import` | 强制包含 PyInstaller 静态分析可能遗漏的 S7、Mitsubishi 协议库。 |
| `--name` / `--icon` | 设置 EXE 文件名和 Windows 图标。 |

#### 双击 EXE 后的运行过程

1. PyInstaller 启动内置 Python 运行环境，并在临时目录中释放打包资源。
2. 程序执行 `backend/main.py` 的 `main()`：查找可用端口、创建系统托盘，并启动浏览器。
3. Uvicorn 在 `127.0.0.1` 启动 FastAPI 服务；若 8000 被占用，会自动尝试后续端口。
4. FastAPI 从 PyInstaller 的临时资源目录读取内置的 `frontend/dist`，向浏览器提供页面和 API。
5. 浏览器访问 `http://localhost:<port>`，页面再通过 REST、WebSocket 和 SSE 与本机服务交互。

因此，这个 EXE 的主界面本质上是“本机 FastAPI 服务 + 默认浏览器中的 Vue 页面”，而 Windows 原生界面部分主要是系统托盘菜单。

#### 运行时文件位置与注意事项

- 前端静态资源在单文件 EXE 启动时释放到 PyInstaller 临时目录；程序通过 `sys._MEIPASS` 定位这些资源。
- `config.json`、`config.json.bak` 和 `logs/gateway.log` 不写在临时目录，而是写在 EXE 所在目录，确保重启后配置和日志仍保留。
- 发布目录必须具有写权限；不建议直接放在受 UAC 保护的 `Program Files` 中。
- 单文件模式首次启动需要解压资源，启动时间通常会略长；未签名的 PyInstaller 程序也可能触发 Windows 安全软件告警，正式分发建议进行代码签名。
- README 中曾写出 `dist/FactoryLink.exe`，但实际 `build.bat` 配置的输出名是 `dist/工业数据采集网关.exe`，应以脚本为准。

## 常见问题

**Q: 双击EXE没反应？**

A: 查看右下角系统托盘是否有网关图标。如果端口被占用，程序会自动换端口，右键托盘图标选择"打开配置页面"。

**Q: 连接PLC失败？**

A: 先在设备配置页点"测试连接"按钮，确认IP、端口、机架号/插槽号是否正确。西门子S7-1200/1500用rack=0,slot=1，S7-300/400用rack=0,slot=2。

**Q: MQTT收不到数据？**

A: 检查MQTT服务器地址、端口是否正确，确认MQTT已启用。查看系统设置页的日志，排查连接错误。

**Q: 如何批量导入点位？**

A: 在设备配置页点击"导入Excel"，选择文件后会弹出预览对话框，确认点位无误后点击"确认并添加设备"，点位将自动填入添加设备表单。

**Q: AI配置助手怎么用？**

A: 在系统设置页启用AI助手并配置API Key（支持OpenAI、通义千问、DeepSeek等兼容接口），然后在设备配置页展开AI面板，用自然语言描述设备即可自动生成配置。例如："帮我添加一台192.168.1.100的西门子S7-1200，采集DB3偏移0开始的10个浮点数"。

**Q: Modbus RTU串口号怎么填？**

A: 直接填串口号，如 `COM3`、`COM4`。波特率默认9600，校验位默认无校验(N)。

**Q: 采集间隔设多少合适？**

A: 默认1000ms（1秒）。PLC响应快可以设500ms，慢的设备建议2000ms以上。太短可能导致读取超时。

---

因为90%的车间工程师，他们不需要什么云原生，不需要什么Docker，他们就想要一个简单、好用、能解决问题的工具。

---

## 许可证

本项目采用 **Mozilla Public License 2.0 (MPL-2.0)** 许可证开源。

## 作者

**谭策** — 独立开发者 | 工业物联网领域探索者

- 📝 博客：[https://www.zjzwfw.cloud/](https://www.zjzwfw.cloud/)
- 📧 邮箱：huawei_network@foxmail.com
- 💬 微信公众号：**IT Online**

<img src="images/公众号背面.png" width="300" alt="公众号背面">

---

[MPL-2.0](./LICENSE) © 谭策
