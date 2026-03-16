# Deployment Guide

Develooper runs as a long-lived Node.js process. Below are options for running it as a system service.

## macOS (launchd)

Create a plist file at `~/Library/LaunchAgents/com.develooper.bot.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.develooper.bot</string>
    <key>ProgramArguments</key>
    <array>
        <string>node</string>
        <string>--env-file=.env</string>
        <string>--import</string>
        <string>tsx</string>
        <string>src/index.ts</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/path/to/develooper</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>ThrottleInterval</key>
    <integer>30</integer>
    <key>StandardOutPath</key>
    <string>/tmp/develooper.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/develooper.err</string>
</dict>
</plist>
```

Replace `/path/to/develooper` with the actual directory path.

### Service management

```bash
# Load and start
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.develooper.bot.plist

# Status
launchctl list | grep develooper

# Restart
launchctl kickstart -k gui/$(id -u)/com.develooper.bot

# Stop
launchctl bootout gui/$(id -u)/com.develooper.bot
```

## Linux (systemd)

Create a unit file at `~/.config/systemd/user/develooper.service`:

```ini
[Unit]
Description=Develooper Telegram Bot
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/path/to/develooper
ExecStart=/usr/bin/node --env-file=.env --import tsx src/index.ts
Restart=always
RestartSec=30
StandardOutput=append:/tmp/develooper.log
StandardError=append:/tmp/develooper.err

[Install]
WantedBy=default.target
```

Replace `/path/to/develooper` with the actual directory path.

### Service management

```bash
# Reload after editing the unit file
systemctl --user daemon-reload

# Enable (start on boot)
systemctl --user enable develooper

# Start
systemctl --user start develooper

# Status
systemctl --user status develooper

# Restart
systemctl --user restart develooper

# Stop
systemctl --user stop develooper

# Logs
journalctl --user -u develooper -f
```

## Health Endpoint

Develooper exposes an HTTP health endpoint at `http://localhost:3847/health` (configurable via `HEALTH_PORT`). Use it for monitoring:

```bash
curl http://localhost:3847/health | jq
```

Returns JSON with Telegram status, SDK status, uptime, and active message count.

## Logs

By default, Develooper logs to stdout/stderr using [pino](https://github.com/pinojs/pino). Set `LOG_LEVEL` in `.env` to control verbosity.

For pretty-printed development logs:
```bash
npm run dev | npx pino-pretty
```
