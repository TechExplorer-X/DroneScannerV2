# 🛸 DroneScanner — ESP32-S3 无人机 Remote ID 探针（硬件版）

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![ESP32](https://img.shields.io/badge/Hardware-ESP32--S3-blue)](https://www.espressif.com/)
[![Open Drone ID](https://img.shields.io/badge/Protocol-ASTM_F3411--22a-green)](https://www.astm.org/f3411-22a.html)

> **开箱即用的无人机探测探针**：基于 ESP32-S3 的成品硬件，开启 WiFi 混杂模式监听无人机广播的 Remote ID 信号，机身屏幕实时显示，并通过蓝牙 / USB 将数据推送到手机、手表、网页看板等多端查看。

---

## 📸 界面一览

| 屏控图 | 手机 App 看板 |
|:---:|:---:|
| <img src="/img/屏控.png" width="350" height="120"> | <img src="/img/手机app.jpg" width="350" height="120"> |

| 小程序 | 手表 App 看板 |
|:---:|:---:|
| <img src="/img/侦测1.jpg" width="350" height="120"> | <img src="/img/手表.png" width="350" height="120"> |


---

## ✨ 功能特性

- **📡 无人机实时侦测** — WiFi 混杂模式捕获无人机广播的 Remote ID 数据包
- **🔍 Open Drone ID 解码** — 完整解析 ASTM F3411-22a Remote ID 等协议
- **🏷️ 机型自动识别** — 内置机型库（166 条前缀映射，覆盖 DJI 等主流品牌），自动匹配机型
- **🖥️ 机身屏幕实时列表** — 旋转无人机图标 + SN 序列号 + RSSI 信号，多机排序滚动，一目了然
- **📲 双模数据上报**
  - **蓝牙（Nordic UART）**：手机 / 手表 App 直连探针（BLE 名称 `RID-Live`）收 JSON 快照，无需局域网
  - **USB 串口（115200）**：电脑网页看板 / 调试工具直读 JSON
- **📱 多端查看**
  - Android App（蓝牙或 USB 串口直连）
  - 手表看板 App（BLE 直连，息屏震动 / 后台保活）
  - 电脑网页看板、微信小程序
- **🎮 专业扩展（需申请）** — 抓包记录 / 回放 / RID 伪造注入测试 / 离线记录
- **🔧 配网与升级** — 屏控款支持 AP 在线 OTA 升级 / 一键远程烧录工具

---

## 📦 版本说明

| 版本 | 说明 |
|------|------|
| **AP网页版** | 老项目固件，通过AP网页访问192.168.4.1侦测页查看数据，不再更新  |
| **小程序版** | 通过小程序实时侦测查看无人机飞行轨迹及回放等地图信息，因备案繁琐后续更新偏向app模式  |
| **屏控款** | 精简屏控界面 + AP 在线 OTA 固件升级，主力日常款 |
| **肩灯警闪款** | 侦测到无人机触发红蓝警闪（WS2812 双路）+ 警笛，适合布防场景 |

**核心硬件**：ESP32-S3 开发板 + ST7735S 128×128 1.44寸彩屏（可选）+ 2.4GHz 天线 + 按键模块 + 无源蜂鸣器（可选）

---

### 4️⃣ 开发编译烧录

源代码处于不公开状态，请前往Realeas下载烧录工具进行烧录，亦或者自行从项目文件取编译好的bin固件自行烧录，烧录工具支持远程固件更新，优先考虑使用一键烧录工具以便获取最新固件烧录

---

## ⚙️ 工作原理

```mermaid
flowchart LR
    A["🚁 无人机 DJI\n(广播 Remote ID)"] -->|WiFi Beacon / 探针帧| B["📡 ESP32-S3 探针\n(混杂模式 + odid_wifi 解码)"]
    B --> C["🖥️ 机身屏幕\n(实时列表)"]
    B --> D["📲 BLE 上报\n(RID-Live / Nordic UART JSON)"]
    B --> F["🔌 USB 串口\n(115200 JSON)"]
    D --> E["📱 手机 App / 手表 App"]
    F --> G["🖥️ 电脑网页看板"]
```

1. **混杂模式监听** — 不连接任何路由器，直接监听 2.4GHz 无线信道
2. **过滤 ODID 帧** — 筛选包含 ODID 协议头的 Beacon / 探针帧
3. **协议解码** — `odid_wifi` 提取载荷，`opendroneid` 解码为结构化数据
4. **机型识别** — SN 前缀匹配机型库（`drone_models.csv`166 条映射）
5. **实时展示** — 机身屏幕 + BLE / USB 双通道推送 JSON 快照（`\n` 分帧，最大 20 架）
6. **多端联动** — 手机 / 手表 / 网页看板任一端接入即用

---

## 🛠️ 配置说明

### 热点与蓝牙

```cpp
AP_SSID       "Remote_ID_Tool"   // 扩展功能 AP 热点
AP_PASS       "12345678"         // 热点密码
AP_CH         1/6/11             // WiFi 信道
BLE_DEV_NAME  "RID-Live"         // 蓝牙广播名（App 搜索自动连接）
MAX_DRONES    20                 // 同时跟踪无人机上限
TIMEOUT_MS    30000              // 30s 无信号判定离线
```

---

## 🔌 接线说明

#### 🖥️ 屏幕 (ST7735S 1.44寸) → ESP32-S3

| 屏幕 (ST7735S) | ESP32-S3 |
| --- | --- |
| GND | GND |
| VCC | 3V3 |
| SCL | GPIO 12 |
| SDA | GPIO 11 |
| RES | GPIO 9 |
| DC | GPIO 8 |
| CS | GPIO 10 |
| BL | GPIO 38 |

## 🎮 按键 → ESP32-S3

| 按键 | ESP32-S3 |
| --- | --- |
| 下拉接地 |
| （UP） | GPIO 4 |
| （DOWN） | GPIO 5 |
| （OK） | GPIO 6 |
| （BLACK） | GPIO 7 |

## 🔔 无源蜂鸣器 → ESP32-S3

| 无源蜂鸣器 | ESP32-S3 |
| --- | --- |
| VCC | 5V |
| I/O | IO17 |
| GND | GND |

## 🔌 数据接口

探针通过 **USB 串口** 与 **BLE** 输出同一份 JSON 快照，行尾 `\n` 分帧：

```json
{
  "t": "snap",
  "n": 1,
  "ch": 6,
  "bat": 75,
  "drones": [
    {
      "mac": "AA:BB:CC:DD:EE:FF",
      "rssi": -65,
      "id": "1581F8LQ1234",
      "model": "Mavic 3 Pro",
      "lat": 39.9042,
      "lon": 116.4074,
      "alt": 120.0,
      "ha": 80.0,
      "speed": 5.2,
      "heading": 180,
      "olat": 39.9051,
      "olon": 116.4061,
      "age": 3
    }
  ]
}
```

| 字段 | 说明 |
|------|------|
| `mac` | 无人机 MAC 地址 |
| `rssi` | 信号强度（dBm） |
| `id` | 无人机序列号（UAS ID） |
| `model` | 匹配到的机型名称 |
| `lat` / `lon` | 无人机经纬度（火星坐标） |
| `alt` / `ha` | 海拔 / 相对地面高度（m） |
| `speed` / `heading` | 速度（m/s）/ 航向（°） |
| `olat` / `olon` | 操作员位置（如广播中携带） |
| `age` | 距上次收到信号秒数 |

---

## 🔬 串口日志示例

```
RID v2.4 | AP:Remote_ID_Tool | BLE:RID-Live | 侦测中...
[1] AA:BB:CC:DD:EE:FF RSSI:-60 | 机型:Mavic 3 Pro | 39.9042,116.4074 alt:120m ha:80m
[state] 3 targets | 187 packets
```

---

## ❓ 常见问题

<details>
<summary><b>手机搜不到 RID-Live？</b></summary>

- 确认探针已上电且蓝牙开关开启（可在屏幕菜单查看）
- Android 12+ 需授予 App「附近设备」权限，且系统定位需开启
- 首次使用：连接时同时授权蓝牙 + 位置
</details>

<details>
<summary><b>探测不到无人机？</b></summary>

- 部分机型在地面不广播 Remote ID，需起飞后才有信号
- 无外接天线时探测距离约 100m，靠近无人机试试
- 检查天线是否接好；用屏幕 / 串口日志确认探针在正常收包
</details>

<details>
<summary><b>App 列表一直是空的？</b></summary>

- 确认 App 显示「已连接」（BLE 或 USB 串口 115200）
- 检查屏幕状态栏「总数」是否有数字；有数字但列表空请反馈截图
- USB 直连请确认插的是板载 USB 口（非 UART 转串口 COM 口）
</details>

<details>
<summary><b>如何升级固件？</b></summary>

- 屏控款：进入 OTA 页 → 手机连热点 `RID_Spoofer`（密码 12345678）→ 浏览器 `http://192.168.4.1` → 上传 bin
- 其它款：USB 线刷（一键在线烧录工具）
</details>

---

## 📄 License

本项目基于开源 ODID 库和 ESP32 Arduino 框架开发，**仅用于教育和研究目的**。

- [Open Drone ID](https://github.com/opendroneid) — 开源 ODID 协议实现
- [ASTM F3411-22a](https://www.astm.org/f3411-22a.html) — Remote ID 标准
- [ESP32 Arduino](https://docs.espressif.com/projects/arduino-esp32/en/latest/) — 官方文档

---

## 🙌 交流与支持

- 固件问题、使用操作、功能建议 → 交流群 `1102647907`
- 新机型识别映射（SN 前缀 + 型号）欢迎提供测试数据