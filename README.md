# zhengdian-esp32s3-homeassistant-panel
基于正点原子 ESP32-S3 Box 与 Arduino 框架开发的 Home Assistant 智能中控屏项目。支持通过 MQTT 动态更新 UI 页面，espnow离线控制，避免重复刷机

## 核心特性
- 🖐️**智能唤醒**：支持挥手唤醒与晃动唤醒。

- **组件支持**：支持将 Button、Switch、可变色灯接入中控屏（窗帘与 HA 自动化类型前端已就绪，待进一步验证）。

- 🌐**离线支持**：断网时自动切换至 ESP-NOW 发送控制指令。极大提升稳定性（注：需要接收设备端自行配置执行代码，可配合 ESPHome 节点设备使用）。

- 🖥️**动态 UI**：整体 UI 界面由 SquareLine Studio (SLS) 绘制。可通过 MQTT 向 panel/config 主题发送 JSON 数据，实现设备列表和控件的动态更新。

- 🌤️**主页显示**：内置天气、时间、WiFi信号强度与当前室温显示，包含 3 个子界面，并支持选择切换 3 种不同的壁纸。

- ⚡**UI状态更新**：通过 HTTP REST API 向 HA 发送控制命令，同时借助 WebSocket 实时监听并更新 UI 状态（注意：需要在 HA 中申请长期访问令牌）。

### 🛠️关于音频模块es8311
- 本项目对es8311进行了初始化

原本的计划是将其打造成带有语音助手的完全体，但鉴于在 Arduino 环境下实现本地语音唤醒的难度实在太高，我放弃了。而我又没有什么让中控屏发声的场景。但是正点原子的ESP32S3 BOX有两个版本，最直接的就是音频驱动，es8311的不同，所以保留了这段，好分清到底用的是哪版
## 🎨UI自定义与壁纸替换

为了规避开源项目的图片版权风险，源码中默认提供的 3 张壁纸已替换为纯色图片。

- **文件路径**：src/ui/images/

- **对应文件**：ui_img_miku_png.c、ui_img_nagisa_png.c、ui_img_aniime_png.c

- 可以挑选自己喜欢的图片（分辨率需裁剪为 320x240），使用 SLS 将其转换为 .c 格式的代码文件，直接替换掉上述三个同名文件

## ⚙️ 动态 UI 配置说明 (JSON 配置指南)

```
[
  {"name": "led灯", "id": "light.bedroom_light", "state": false, "type": "image", "room": "home", "mac": "94:54:C5:AA:A1:24"},
  {"name": "开灯", "id": "button.bedroom_light_on", "type": "button", "room": "home"},
  {"name": "关灯", "id": "button.bedroom_light_off", "type": "button", "room": "home"},
  {"name": "吊灯", "id": "light.ceiling", "state": false, "type": "switch", "room": "home"},
  {"name": "空调", "id": "ac.master", "state": true, "type": "switch", "room": "living"},
  {"name": "电视", "id": "tv.living", "state": false, "type": "image", "room": "bedroom"},
  {"name": "氛围灯", "id": "light.strip", "state": true, "type": "slider"}
]
```
### 字段参数详解

- name (显示名称)：显示在中控屏按钮上的文字。
  - 注意！中控屏的中文字体是在 SquareLine Studio (SLS) 中手动录入并打包编译的。如果你在这里输入的汉字没有包含在编译的字体库中，屏幕上就会显示为一个“方块”（缺失字符）。如果你要增加新的设备名，需要去 SLS 中重新导出一次字体文件。
  
- id （实体id）：该设备在 Home Assistant 中的唯一标识符（Entity ID）。你可以登录 HA 后台，在“开发者工具 -> 状态”中找到它。

- type (组件与 API 类型)：核心字段！它不仅决定了 UI 显示什么样式的组件，还决定了向 HA 发送什么类型的控制 API。必须准确填写。目前支持：
  - switch：基础开关组件
  - button：点按按钮
  - slider：窗帘组件，调用的是/api/services/cover/set_cover_position，但是我没有窗帘设备，所以没有试验过
  - image：**可变色灯光组件**(注：起这个名字是因为最初在 SLS 绘制 UI 时，我使用了一个灯泡的 Image 图片作为点击载体，而 Switch 类型用的是 SLS 原生的开关组件，为了区分底层逻辑，故将调光调色灯具命名为 image，请注意理解)
  - automation：HA自动化，但是我还没有试验过
  
- state：(初始状态)：UI 的初始状态展示（true 为开，false 为关）。此字段仅针对 switch 和 image 设置

- room (页面归属)：设置该按钮显示在中控屏的哪一页。中控屏共设计了 3 个页面，可通过手指横向滑动切换。有效值为：
    - home
    - living
    - bedroom

- mac (ESP-NOW 离线直连 MAC)：(选填) 用于断网容灾控制
    - 如果你填入了接收端设备的 MAC 地址，当 WiFi 断开时，中控屏会通过 ESP-NOW 协议直接向该 MAC 发送控制指令
    - 如果没有配置该字段，该按钮在断网时将不会发送指令
    - 注：使用此功能需要你的接收设备（如基于 ESPHome 的节点）自行编写并配置对应的 ESP-NOW 接收代码
