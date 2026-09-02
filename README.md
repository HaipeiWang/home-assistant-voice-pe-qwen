# Home Assistant Voice PE for Qwen Realtime

[中文](README.zh-CN.md) | English

Custom firmware that turns Home Assistant Voice: Preview Edition into a thin,
full-duplex client for the
[Qwen Realtime Voice Agent add-on](https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent).
The device performs the wake-word, microphone, speaker and LED work; Qwen handles
speech understanding and speech generation, while the add-on exposes Home Assistant
tools to the model.

> This repository is a fork of
> [xandervanerven/home-assistant-voice-pe](https://github.com/xandervanerven/home-assistant-voice-pe),
> which is based on the official Home Assistant Voice PE ESPHome project.

## What is included

- `va_client`: the tested WebSocket audio/state client used by Voice PE.
- Patched `aic3204` audio component required by the playback path.
- Wake chime gating: microphone capture starts only after the chime has really ended.
- Explicit `conversation_end`, generation-safe sessions and clean reconnect behavior.
- Warm resampler/I2S playback path and stricter echo rejection during replies.
- Automatic connection to `ws://homeassistant.local:8080/` by default.
- Remote ESPHome packages, so later firmware updates can be installed from the
  ESPHome dashboard.

## Required companion add-on

Install and configure
[HaipeiWang/ha-qwen-realtime-voice-agent](https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent)
before testing the device. The firmware does not contain a Qwen API key and does not
connect to Qwen directly.

## First installation

1. Install **ESPHome Device Builder** in Home Assistant.
2. Add the following repository to the Home Assistant Add-on Store and install its
   Qwen Realtime add-on:
   `https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent`
3. In the add-on configuration, provide your DashScope API key and the settings
   required for your Qwen model, then start the add-on. Keep port `8080` reachable
   from the Voice PE LAN.
4. In ESPHome Builder, put the values shown in
   [`secrets.yaml.example`](secrets.yaml.example) into your own `secrets.yaml`.
5. Copy either [`esphome-builder.dhcp.yaml`](esphome-builder.dhcp.yaml) or
   [`esphome-builder.static-ip.yaml`](esphome-builder.static-ip.yaml) into the
   device configuration. Change only the device name and any needed substitutions.
6. Perform the first flash over USB. Later builds can be installed over Wi-Fi.

The default backend address is `ws://homeassistant.local:8080/`. If that mDNS name
does not resolve on your network, add this substitution to the per-device stub:

```yaml
substitutions:
  va_url: "ws://YOUR_HOME_ASSISTANT_ADDRESS:8080/"
```

## Automatic discovery and connection

After a successful first flash:

- ESPHome advertises the device on the LAN and Home Assistant discovers its native
  API connection automatically. The user may still need to click **Configure** in
  Home Assistant to accept a newly discovered device.
- `va_client` resolves `homeassistant.local`, connects to the add-on WebSocket, and
  reconnects automatically after temporary network or add-on interruptions.
- The add-on independently reads the entities exposed to Assist, builds its Qwen
  tool definitions at startup, and registers those tools for each Qwen session.

Wi-Fi credentials, the ESPHome API encryption key and OTA password are necessarily
device-specific and must be supplied during first installation. No public firmware
can discover those secrets automatically.

## Verify

1. Start the Qwen add-on and confirm its log shows Home Assistant MCP initialization,
   exposed-entity discovery and generated tool counts.
2. Power on the Voice PE. Its log should show a WebSocket connection to port `8080`.
3. Say **Alexa**, wait for the wake chime, then ask a general question.
4. Ask it to control an entity that is exposed to Assist and confirm both the entity
   state and spoken reply.
5. End the follow-up window and confirm the log contains `conversation_end` and the
   Qwen session closes only after playback drains.

See the full [installation and troubleshooting guide](INSTALL.md).

## Version

This branch currently publishes firmware `1.3.0-beta.1`. It is a test release based
on physical Voice PE and HAOS add-on validation completed before publication.

## License and upstream

See [LICENSE](LICENSE). Hardware background and stock firmware documentation remain
available from [Home Assistant Voice PE](https://voice-pe.home-assistant.io/).
