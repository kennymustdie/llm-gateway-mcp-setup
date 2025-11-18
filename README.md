# LLM-Gateway MCP Setup Guide

Remote MCP Server für Ollama Turbo Cloud Gateway mit SSH-Zugriff.

## 🎯 Was ist das?

Ein Model Context Protocol (MCP) Server der auf einem Hetzner Server läuft und Zugriff auf verschiedene LLM-Modelle über Ollama Turbo Cloud bietet. Der Server wird remote via SSH in Claude Desktop, Claude Code und Goose eingebunden.

## ✨ Features

- **llm_models**: Liste aller verfügbaren Modelle
- **llm_chat**: Allgemeine Chat-Anfragen
- **llm_code**: Code-spezifische Anfragen

### Verfügbare Modelle

- `general-fast`: Schnelles Allzweck-Modell
- `general-deep`: Tiefgreifendes Reasoning
- `code-pro`: Optimiert für Code-Aufgaben
- Weitere Modelle über `llm_models` Tool

## 📋 Voraussetzungen

1. **SSH-Zugriff** zum Hetzner Server (135.181.128.98)
2. **SSH Key Authentication** (KEINE Passwort-Auth!)
3. **Claude Desktop** oder **Claude Code** oder **Goose**

## 🔐 Schritt 1: SSH Key Setup

### SSH Key prüfen

```bash
ls ~/.ssh/id_*
```

Wenn kein Key existiert:

```bash
# SSH Key generieren
ssh-keygen -t ed25519 -C "your_email@example.com"

# Key zum Server kopieren
ssh-copy-id thorsten@135.181.128.98
```

### Passwordless SSH testen

```bash
ssh thorsten@135.181.128.98 "echo 'SSH works!'"
```

✅ Wenn das OHNE Passwort-Eingabe funktioniert, bist du bereit!

## 🖥️ Schritt 2: Claude Desktop Konfiguration

### Config-Datei öffnen

**macOS:**
```bash
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Windows:**
```
%APPDATA%\Claude\claude_desktop_config.json
```

### MCP-Eintrag hinzufügen

```json
{
  "mcpServers": {
    "llm_gateway": {
      "command": "ssh",
      "args": [
        "thorsten@135.181.128.98",
        "node",
        "/opt/llm-gateway-mcp/dist/index.js"
      ],
      "env": {}
    }
  }
}
```

### Claude Desktop neu starten

1. **Cmd+Q** (komplett beenden)
2. Claude Desktop neu öffnen
3. Im MCP-Menü sollte **llm_gateway** erscheinen

### Testen

Prompt in Claude Desktop:
```
Nutze das MCP-Tool "llm_models" und zeige mir alle verfügbaren Modelle.
```

## 💻 Schritt 3: Claude Code Konfiguration

### Projekt-Config erstellen

In deinem Projekt-Root (z.B. `/Users/studio/dev/mein-projekt`):

```bash
nano .claude.json
```

### Config hinzufügen

```json
{
  "mcpServers": {
    "llm_gateway": {
      "command": "ssh",
      "args": [
        "thorsten@135.181.128.98",
        "node",
        "/opt/llm-gateway-mcp/dist/index.js"
      ],
      "env": {}
    }
  }
}
```

### Testen

```bash
cd /Users/studio/dev/mein-projekt
claude code
```

In Claude Code:
```
Liste alle MCP-Server. Rufe dann llm_models vom llm_gateway auf.
```

## 🦆 Schritt 4: Goose Konfiguration (Optional)

### Config-Datei öffnen

```bash
nano ~/.config/goose/config.yaml
```

### Extension hinzufügen

```yaml
extensions:
  llm_gateway:
    enabled: true
    type: stdio
    cmd: ssh
    args:
      - thorsten@135.181.128.98
      - node
      - /opt/llm-gateway-mcp/dist/index.js
    envs: {}
    timeout: 300
```

### Goose neu starten

Komplett beenden und neu öffnen.

## 🧪 Vollständiger Test

### 1. Modelle auflisten

```
Rufe llm_models auf.
```

Erwartetes Ergebnis:
```json
{
  "models": [
    {"id": "general-fast", "name": "General Fast", "type": "general"},
    {"id": "general-deep", "name": "General Deep", "type": "general"},
    {"id": "code-pro", "name": "Code Pro", "type": "code"}
  ]
}
```

### 2. Chat-Anfrage

```
Nutze llm_chat mit model "general-fast" und frage: "Was ist Quantencomputing?"
```

### 3. Code-Anfrage

```
Nutze llm_code mit model "code-pro" und frage: "Erkläre Python Decorators"
```

## 🔧 Troubleshooting

### Problem: MCP erscheint nicht in Claude

**Lösung:**
1. Claude Desktop **komplett beenden** (Cmd+Q)
2. Config-Datei prüfen (gültiges JSON?)
3. SSH-Verbindung testen: `ssh thorsten@135.181.128.98 "echo OK"`
4. Claude neu starten

### Problem: "Permission denied"

**Lösung:**
```bash
# SSH Key nochmal zum Server kopieren
ssh-copy-id thorsten@135.181.128.98

# Testen
ssh thorsten@135.181.128.98 "echo OK"
```

### Problem: "Connection timeout"

**Lösung:**
1. Netzwerk-Verbindung prüfen
2. Server erreichbar? `ping 135.181.128.98`
3. SSH Port offen? `nc -zv 135.181.128.98 22`

### Problem: "node: command not found"

**Kontaktiere Server-Admin** - Node.js muss auf dem Server installiert sein.

## 📚 MCP Tool Referenz

### llm_models

Liste alle verfügbaren Modelle.

**Parameter:** Keine

**Beispiel:**
```
Zeige mir alle verfügbaren Modelle via llm_models.
```

### llm_chat

Allgemeine Chat-Anfrage.

**Parameter:**
- `model` (string): Model ID (z.B. "general-fast")
- `message` (string): Deine Frage

**Beispiel:**
```
Nutze llm_chat mit model "general-deep" und message "Erkläre Blockchain"
```

### llm_code

Code-spezifische Anfrage.

**Parameter:**
- `model` (string): Model ID (z.B. "code-pro")
- `query` (string): Code-Frage

**Beispiel:**
```
Nutze llm_code mit model "code-pro" und query "Wie funktioniert async/await?"
```

## ⚙️ Erweiterte Konfiguration

### SSH Config für einfacheren Zugriff

Erstelle `~/.ssh/config`:

```
Host hetzner
    HostName 135.181.128.98
    User thorsten
    IdentityFile ~/.ssh/id_ed25519
```

Dann kannst du die MCP-Config vereinfachen:

```json
{
  "command": "ssh",
  "args": [
    "hetzner",
    "node",
    "/opt/llm-gateway-mcp/dist/index.js"
  ]
}
```

### Connection Keepalive

In `~/.ssh/config`:

```
Host hetzner
    HostName 135.181.128.98
    User thorsten
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Verhindert Timeout bei längeren MCP-Sessions.

## 🚀 Nächste Schritte

1. ✅ SSH Key Setup
2. ✅ Claude Desktop/Code konfiguriert
3. ✅ MCP funktioniert
4. 🎉 Nutze die LLM-Tools in deinen Projekten!

## 📞 Support

Bei Problemen:
1. Config-Dateien prüfen (gültiges JSON/YAML?)
2. SSH-Verbindung testen
3. Claude komplett neu starten
4. Server-Admin kontaktieren wenn Node.js Probleme auftreten

## 📝 Lizenz

Private Setup Guide - Not for redistribution

---

**Version:** 1.0.0
**Last Updated:** November 2025
**Server:** Hetzner (135.181.128.98)
**MCP Path:** `/opt/llm-gateway-mcp/dist/index.js`
