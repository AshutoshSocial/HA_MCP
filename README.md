[![Releases](https://img.shields.io/github/v/release/AshutoshSocial/HA_MCP?label=Releases&style=for-the-badge)](https://github.com/AshutoshSocial/HA_MCP/releases)

# HA_MCP — Home Assistant Add-on for Claude MCP Integration

![Home Assistant + Claude](https://www.home-assistant.io/images/hass_logo.png)

Control smart devices, create automations, and manage your home with natural language through Claude Desktop using the Model Context Protocol (MCP).

- Topics: ai-assistant, anthropic, artificial-intelligence, automation, claude-ai, claude-desktop, docker, hassio, home-assistant, home-assistant-addon, home-automation, iot, llm, mcp, model-context-protocol, natural-language, smart-home, typescript, voice-assistant, websocket

Table of contents
- Features
- How it works
- Quick install
- Configuration
- Usage examples
- Automations and scripts
- WebSocket / MCP reference
- Development and build
- Troubleshooting
- Contributing
- License
- Releases

Features
- Natural language control via Claude Desktop MCP.
- Two-way WebSocket bridge between Home Assistant and Claude.
- Device control, scene and script triggers.
- Context-aware automation using MCP messages.
- Role and permission mapping for user accounts.
- TypeScript core with Docker image for Home Assistant Supervisor.
- Lightweight, low-latency design for local deployment.

How it works
- The add-on runs a small MCP server inside Home Assistant.
- Claude Desktop connects to that MCP endpoint via WebSocket.
- The add-on exposes a set of actions and intents as MCP messages.
- Home Assistant executes services based on MCP payloads.
- The add-on returns state and sensor data to Claude to keep context.

Quick install
1. Open the Releases page and download the packaged release file. The release file needs to be downloaded and executed from the Releases page: https://github.com/AshutoshSocial/HA_MCP/releases
2. Use the Home Assistant Supervisor UI to install the add-on from a local file or from a custom repository.
3. Start the add-on and open the Web UI or the ingress URL.
4. Point Claude Desktop to the add-on MCP endpoint (ws://YOUR_HA:PORT/mcp or wss when using SSL).

Releases link and file
- The releases page contains packaged build artifacts and an install script per release. Download the file for your platform and execute the included installer or run the provided Docker image.
- Visit the Releases page for the correct package and the release notes: https://github.com/AshutoshSocial/HA_MCP/releases

Configuration
Add-on config keys (config.json / add-on UI)
- listen_port: integer — TCP port for MCP WebSocket (default 8765).
- use_ssl: boolean — Enable TLS for WebSocket (default false).
- ssl_cert: path — Path to TLS certificate inside add-on (optional).
- ssl_key: path — Path to TLS key inside add-on (optional).
- allowed_users: list — Home Assistant user IDs allowed to control devices.
- log_level: string — "info", "debug", "warn", "error".
- mcp_protocol_version: string — Defaults to "1.0".

Home Assistant integration settings (configuration.yaml)
Example:
mcp:
  host: 127.0.0.1
  port: 8765
  token: YOUR_MCP_TOKEN
  enabled_contexts:
    - device_state
    - entity_attributes

Token management
- Generate a token in the add-on UI and paste it into Claude Desktop connection settings.
- Rotate tokens from the add-on UI or by editing the add-on config.

Usage examples

Manual control via Web UI
- Open the add-on Web UI.
- Use the test console to send MCP messages.
- Send a command payload such as:
{
  "intent": "turn_on",
  "entity_id": "light.living_room",
  "context": { "source": "claude" }
}

Sample MCP payloads
- Intent: device_control
  {
    "intent": "device_control",
    "action": "turn_on",
    "entity_id": "switch.coffee_maker"
  }

- Intent: ask_state
  {
    "intent": "ask_state",
    "entity_id": "sensor.temperature_living_room"
  }

Triggering HA services from MCP
- The add-on maps MCP intents to service calls.
- Example mapping:
  - device_control -> homeassistant.turn_on / turn_off / toggle
  - set_state -> homeassistant.update_entity
  - run_script -> script.turn_on

Automations and scripts
Example: Trigger automation when Claude issues an MCP scene command
automation:
  - alias: "Claude Scene Trigger"
    trigger:
      platform: event
      event_type: mcp_message
      event_data:
        intent: "activate_scene"
    action:
      - service: scene.turn_on
        data_template:
          entity_id: "{{ trigger.event.data.scene_id }}"

Example: Announce weather via Claude
automation:
  - alias: "Claude Weather Announce"
    trigger:
      platform: time
      at: "07:30:00"
    action:
      - service: mcp.send
        data:
          intent: "announce"
          message: "Morning update: {{ states('sensor.weather_summary') }}."

WebSocket / MCP reference
- Endpoint: ws://<home_assistant_host>:<port>/mcp
- Authentication: Bearer token in the initial WebSocket message or query string token parameter.
- Handshake message example:
  {
    "type": "mcp_init",
    "version": "1.0",
    "token": "YOUR_TOKEN",
    "client": "claude-desktop"
  }

Message types
- mcp_init — initial handshake
- mcp_event — events from Home Assistant (state changes, sensors)
- mcp_command — commands from Claude to Home Assistant
- mcp_ack — acknowledgement messages
- mcp_error — error messages

Message schema (high level)
- id: string — unique message id
- type: string — message type
- intent: string — action identifier
- payload: object — command payload
- context: object — conversation or session context

Security
- Use TLS (wss) when exposing the add-on outside your LAN.
- Use tokens and rotate them regularly.
- Limit allowed_users to the Home Assistant accounts that require access.
- Use Home Assistant user permission model to restrict service calls.

Development and build
Repository layout (example)
- src/ — TypeScript source
- dist/ — compiled JS
- docker/ — Dockerfile and build assets
- ui/ — web UI files and assets
- tests/ — unit and integration tests
- addon.json — Home Assistant add-on manifest
- README.md — this file

Build steps
1. Install Node.js 18+ and pnpm or npm.
2. Install dependencies:
   pnpm install
3. Build TypeScript:
   pnpm build
4. Build Docker image:
   docker build -t ha_mcp:local ./docker

Testing
- Run unit tests:
  pnpm test
- Run integration tests against a Home Assistant dev instance using the test harness.

Docker
- The add-on ships a Docker image that runs the MCP server.
- Use the Dockerfile in docker/ to build a local image.
- The add-on uses a small Node.js runtime and exposes the MCP WebSocket port.

Troubleshooting
- If Claude cannot connect, verify port and token.
- If commands fail, check the add-on logs for mcp_error messages.
- If entity mapping fails, verify entity_id and that the add-on has access to the entity via Home Assistant API.
- If you see TLS errors, ensure certificate and key paths are correct in the add-on config.

FAQ
Q: What is MCP?
A: MCP stands for Model Context Protocol. It is a JSON-based protocol to pass context and commands between a language model client and a local controller.

Q: Does this store voice data?
A: The add-on transfers text and JSON payloads. Store behavior depends on Home Assistant logging and the Claude Desktop client.

Q: Can I use multiple Claude clients?
A: Yes. The add-on supports multiple concurrent MCP connections. Each connection gets its own context namespace.

Q: Do I need the cloud?
A: No. You can run this setup on a local network using local endpoints and local models.

Contributing
- Fork the repo.
- Create a branch for your feature or fix.
- Follow the repo style guide (TypeScript, ESLint).
- Add tests for new behavior.
- Submit a pull request with a clear description and test results.

Code of conduct
- Follow respectful behavior and open discussion.
- Keep pull requests focused and small.

License
- The project uses the MIT License. See LICENSE file in the repo.

Credits and resources
- Home Assistant: https://www.home-assistant.io
- Claude Desktop / Anthropic: https://www.anthropic.com
- MCP spec: This repo implements a simple MCP variant to carry intents and context between Claude Desktop and Home Assistant.

Badges
[![Releases](https://img.shields.io/github/v/release/AshutoshSocial/HA_MCP?label=Releases&style=flat-square)](https://github.com/AshutoshSocial/HA_MCP/releases)  [![Docker Pulls](https://img.shields.io/docker/pulls/ashutoshsocial/ha_mcp?style=flat-square)](https://hub.docker.com/r/ashutoshsocial/ha_mcp)

Images and assets
- Home Assistant logo: https://www.home-assistant.io/images/hass_logo.png
- Use the add-on Web UI to view runtime graphs and message logs.

Releases and downloads
- The release bundle on the Releases page contains packaged artifacts and an install script. Download the release file from Releases and execute the included installer or use the Docker image provided on that page: https://github.com/AshutoshSocial/HA_MCP/releases

Contact
- Open issues in the repository for bugs and feature requests.
- Use pull requests for code changes.