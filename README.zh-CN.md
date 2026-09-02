# Home Assistant Voice PE 千问 Realtime 固件

中文 | [English](README.md)

这是 Home Assistant Voice: Preview Edition 的定制固件。Voice PE 负责唤醒词、
麦克风、扬声器和灯效；
[Qwen Realtime Voice Agent Add-on](https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent)
负责连接千问实时语音模型，并把 Home Assistant 中允许语音访问的实体自动转换成工具。

> 本仓库 fork 自
> [xandervanerven/home-assistant-voice-pe](https://github.com/xandervanerven/home-assistant-voice-pe)，
> 后者基于 Home Assistant 官方 Voice PE ESPHome 项目。

## 主要功能

- 实测版 `va_client` WebSocket 音频与状态客户端。
- 播放链路所需的修补版 `aic3204` 组件。
- 唤醒音真正播放结束后才开放麦克风，避免唤醒音被模型误识别。
- 显式 `conversation_end`、generation 隔离和可靠的会话关闭/重连。
- 常驻重采样与 I2S 播放链，降低回复被截断、断续和爆音的概率。
- 默认自动连接 `ws://homeassistant.local:8080/`。
- 通过 ESPHome 远程 package 安装，后续可在 ESPHome 面板更新。

## 必需的配套 Add-on

请先安装并配置
[HaipeiWang/ha-qwen-realtime-voice-agent](https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent)。
固件不保存千问 API Key，也不直接连接千问；所有云端鉴权均由 HAOS 中的 Add-on 完成。

## 首次安装

1. 在 Home Assistant 安装 **ESPHome Device Builder**。
2. 在 Add-on 商店加入仓库：
   `https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent`，安装并启动千问
   Realtime Add-on。
3. 在 Add-on 配置页填写 DashScope API Key 和模型所需配置，保持局域网可访问
   `8080` 端口。
4. 参照 [`secrets.yaml.example`](secrets.yaml.example)，在 ESPHome 的
   `secrets.yaml` 中填写 Wi-Fi、OTA 密码和 API 加密密钥。
5. 将 [`esphome-builder.dhcp.yaml`](esphome-builder.dhcp.yaml) 或
   [`esphome-builder.static-ip.yaml`](esphome-builder.static-ip.yaml) 复制为设备
   配置，只修改设备名和必要的 substitutions。
6. 第一次通过 USB 刷写；以后可以通过 Wi-Fi OTA 更新。

固件默认连接 `ws://homeassistant.local:8080/`。如果局域网不能解析该 mDNS 名称，
在设备 stub 中加入：

```yaml
substitutions:
  va_url: "ws://你的Home-Assistant地址:8080/"
```

## 自动发现与自动连接的边界

首次刷写成功后：

- ESPHome 会在局域网广播设备，Home Assistant 可自动发现 native API；首次发现时
  仍可能需要用户点击一次“配置”来接受设备。
- `va_client` 会解析 `homeassistant.local`、连接 Add-on，并在网络或 Add-on 短暂
  中断后自动重连。
- Add-on 会读取允许 Assist 访问的实体，自动生成能力工具，并在每个千问会话中注册。

Wi-Fi 密码、ESPHome API 加密密钥和 OTA 密码是每台设备独有的，第一次刷写时必须
由用户提供；公开固件无法也不应该自动发现这些秘密。

## 验证方法

1. 启动 Add-on，日志应显示 HA MCP 初始化、暴露实体数量和自动生成工具数量。
2. 启动 Voice PE，设备日志应显示已连接 Home Assistant 的 `8080` WebSocket。
3. 说“Alexa”，等待唤醒音结束后问一个纯聊天问题，应得到完整语音回复。
4. 控制一个已暴露给 Assist 的实体，确认实体状态改变且收到语音确认。
5. 连续对话窗口结束后，确认日志出现 `conversation_end`，并且播放器排空后才关闭
   千问会话。

更完整步骤见 [安装与排障指南](INSTALL.md)。当前版本为 `1.3.0-beta.1` 测试版。
