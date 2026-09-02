# Installation and troubleshooting

This solution has two parts:

```text
Voice PE firmware -- LAN WebSocket :8080 --> Qwen Realtime HAOS add-on
       wake/mic/speaker/LED              Qwen audio + HA MCP tools
```

## 1. Install the HAOS add-on

1. Open **Settings → Add-ons → Add-on Store → Repositories**.
2. Add `https://github.com/HaipeiWang/ha-qwen-realtime-voice-agent`.
3. Install **Qwen Realtime Voice Agent**.
4. Open its Configuration page. At minimum, enter a valid DashScope API key.
   Model-specific Workspace ID and endpoint fields may be required by your account
   and region. Do not place any API key in the firmware repository or device YAML.
5. Start the add-on and inspect its log. A healthy startup reports successful Home
   Assistant MCP initialization, the number of exposed entities and generated tools,
   and a WebSocket listener on port `8080`.

The Add-on uses Home Assistant's internal Supervisor token by default. Only entities
exposed to Assist are included in its automatic entity catalog. Use that exposure
list as the access-control boundary.

## 2. Prepare ESPHome secrets

Install **ESPHome Device Builder**, then add these keys to its `secrets.yaml`:

```yaml
wifi_ssid: "YOUR_WIFI"
wifi_password: "YOUR_WIFI_PASSWORD"
ota_password: "A_DEVICE_SPECIFIC_OTA_PASSWORD"
api_key: "A_32_BYTE_BASE64_ESPHOME_API_KEY"
```

For the fixed-IP template also add `static_ip`, `gateway`, `subnet`, `dns1`, and
`dns2`. The repository's [`secrets.yaml.example`](secrets.yaml.example) is safe to
copy as a schema, but its placeholder values must be replaced locally.

## 3. Flash Voice PE

1. Copy [`esphome-builder.dhcp.yaml`](esphome-builder.dhcp.yaml) into the adopted
   device configuration. Use
   [`esphome-builder.static-ip.yaml`](esphome-builder.static-ip.yaml) only if you
   intentionally need a fixed address.
2. Change `name` and `friendly_name`; keep the `packages` and `dashboard_import`
   sections unchanged.
3. If `homeassistant.local` does not resolve from the Voice PE network, add a
   `va_url` substitution containing the Home Assistant LAN hostname or address.
4. Validate the configuration, then use **Install → Plug into this computer** for
   the first flash. OTA is available after the device joins Wi-Fi.
5. Accept the ESPHome device if Home Assistant presents a discovered-integration
   notification.

## 4. End-to-end acceptance test

Run these in order:

1. **Connection:** boot the add-on, then Voice PE. Confirm a device WebSocket
   connection in the Add-on log.
2. **Conversation:** say “Alexa”, wait for the chime to finish, and ask for a
   three-sentence story. The complete audio should play without stale audio from a
   prior response.
3. **Tool discovery:** expose one test light to Assist and restart the Add-on. Its
   startup log should list a non-zero entity catalog and generated capability count.
4. **Tool execution:** ask to turn the test light on and off. Confirm the Qwen tool
   call, MCP execution/result logs, actual entity state, and spoken confirmation.
5. **Boundary:** let the follow-up window expire. Confirm `conversation_end`,
   `response.done`, paced-player drain, and then provider-session closure in that
   order.
6. **Recovery:** restart only the Add-on. Voice PE should reconnect without being
   reflashed or power-cycled.

## Troubleshooting

### Device stays disconnected/red

- Confirm the Add-on is running and host port `8080` is available on the LAN.
- From the device network, ensure `homeassistant.local` resolves. If not, override
  `va_url` with a stable LAN hostname or address.
- Check ESPHome logs for DNS, Wi-Fi and `va_client` reconnect messages.

### Wake works but there is no answer

- Check that the DashScope key/model/region configuration is valid.
- Look for Qwen session creation and `session.updated` in the Add-on log.
- Confirm microphone bytes are received after `wake` and that server VAD creates a
  response.

### Chat works but devices do not move

- Confirm the target entity is exposed to Assist.
- Restart the Add-on after changing the exposure list so the entity/tool cache is
  regenerated.
- Check the tool sequence: received → parsed → MCP executed → result returned.

### Speech is cut off

- Verify the current firmware and Add-on versions match this test release.
- A valid response must deliver the dedicated ordered `response.done` boundary only
  after all audio deltas have entered the same playback pipeline.
- An echo-triggered stop should be rare because the reply-time stop threshold is
  deliberately stricter than the idle threshold.

## Updating and rollback

Firmware updates are pulled from this repository's `main` branch by the remote
package. Add-on updates come from its own repository. Before a production rollout,
pin a known-good Git tag in the stub/package if you require deterministic rollback.
The original stock firmware remains available from the official Home Assistant Voice
PE project.
