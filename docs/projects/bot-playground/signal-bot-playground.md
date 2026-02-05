# Signal Bot Playground

[![Go](https://img.shields.io/badge/Go-1.23-00ADD8?logo=go)](https://golang.org/)
[![Signal](https://img.shields.io/badge/Platform-Signal-3A76F0?logo=signal)](https://signal.org/)
[![Automation](https://img.shields.io/badge/Type-Automation-orange)](https://github.com/riyanimam/signal-bot-playground)

A Signal bot playground for building and testing messaging automations on the Signal
platform. It serves as a sandbox for experimenting with Signal bot interactions,
building features like message responders or automated workflows for the Signal messenger.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/signal-bot-playground){ .md-button }

## Features

- **Signal Integration**: Connect to Signal messenger
- **Message Automation**: Respond to messages automatically
- **Group Support**: Handle individual and group conversations
- **Event Handling**: Process incoming messages and events
- **Go Implementation**: Built with Go for performance
- **Extensible**: Easy to add custom automation workflows

## Prerequisites

- **Go** >= 1.23
- **Signal CLI** or Signal API access
- **Signal Account** registered phone number

## Quick Start

```bash
# Clone the repository
git clone https://github.com/riyanimam/signal-bot-playground.git
cd signal-bot-playground

# Install dependencies
go mod download

# Set up Signal CLI (if not already done)
# Follow Signal CLI setup instructions

# Configure bot
export SIGNAL_PHONE_NUMBER="+1234567890"
export SIGNAL_DATA_PATH="./signal-data"

# Run the bot
go run main.go
```

## Project Structure

```text
signal-bot-playground/
├── cmd/
│   └── bot/
│       └── main.go           # Main entry point
├── internal/
│   ├── handlers/
│   │   ├── message.go       # Message handlers
│   │   └── group.go         # Group message handlers
│   ├── signal/
│   │   ├── client.go        # Signal client wrapper
│   │   └── events.go        # Event processing
│   └── config/
│       └── config.go        # Bot configuration
├── go.mod
├── go.sum
└── README.md
```

## Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `SIGNAL_PHONE_NUMBER` | Bot's Signal phone number | Yes |
| `SIGNAL_DATA_PATH` | Path to Signal data directory | Yes |
| `LOG_LEVEL` | Logging level | No (default: `info`) |
| `RESPOND_TO_GROUPS` | Enable group message responses | No (default: `true`) |

### Signal CLI Setup

```bash
# Install Signal CLI (Linux/macOS)
wget https://github.com/AsamK/signal-cli/releases/latest/download/signal-cli-*.tar.gz
tar xf signal-cli-*.tar.gz -C /opt
ln -sf /opt/signal-cli-*/bin/signal-cli /usr/local/bin/

# Register phone number
signal-cli -u +1234567890 register

# Verify with code received via SMS
signal-cli -u +1234567890 verify CODE

# Set up account
signal-cli -u +1234567890 receive
```

## Message Handlers

### Auto-Responder

```go
package handlers

import (
    "strings"
)

// HandleIncomingMessage processes incoming messages
func HandleIncomingMessage(msg *Message) (*Response, error) {
    // Convert to lowercase for case-insensitive matching
    content := strings.ToLower(msg.Content)
    
    switch {
    case strings.Contains(content, "hello"):
        return &Response{
            Text: "Hello! How can I help you today?",
        }, nil
    case strings.Contains(content, "help"):
        return &Response{
            Text: "Available commands:\n- hello\n- help\n- status",
        }, nil
    case strings.Contains(content, "status"):
        return &Response{
            Text: "Bot is running normally!",
        }, nil
    default:
        return nil, nil // No response
    }
}
```

### Keyword Triggers

```go
package handlers

// KeywordHandler handles keyword-based triggers
type KeywordHandler struct {
    keywords map[string]string
}

func NewKeywordHandler() *KeywordHandler {
    return &KeywordHandler{
        keywords: map[string]string{
            "weather": "I can't check weather yet, but it's probably nice!",
            "time":    "The current time is: %s",
            "joke":    "Why did the Go developer get lost? Because they didn't have a map!",
        },
    }
}

func (h *KeywordHandler) Handle(msg *Message) (*Response, error) {
    for keyword, response := range h.keywords {
        if strings.Contains(strings.ToLower(msg.Content), keyword) {
            return &Response{Text: response}, nil
        }
    }
    return nil, nil
}
```

## Development Workflow

### Running Locally

```bash
# Run with debugging
go run -v main.go

# Build binary
go build -o signal-bot cmd/bot/main.go
./signal-bot

# Run tests
go test ./...
```

### Testing Signal Integration

```bash
# Send test message to bot
signal-cli -u +1234567890 send -m "test message" +TARGET_NUMBER

# Receive messages
signal-cli -u +1234567890 receive

# Monitor logs
tail -f signal-bot.log
```

### Code Quality

```bash
# Format code
go fmt ./...

# Run linter
golangci-lint run

# Static analysis
go vet ./...

# Security check
gosec ./...
```

## Deployment

### Docker Deployment

```dockerfile
FROM golang:1.23-alpine AS builder

WORKDIR /app
COPY . .
RUN go mod download
RUN go build -o signal-bot cmd/bot/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates openjdk11-jre

# Install Signal CLI
RUN wget -O /tmp/signal-cli.tar.gz \
    https://github.com/AsamK/signal-cli/releases/latest/download/signal-cli-*.tar.gz && \
    tar xf /tmp/signal-cli.tar.gz -C /opt && \
    ln -sf /opt/signal-cli-*/bin/signal-cli /usr/local/bin/

WORKDIR /root/
COPY --from=builder /app/signal-bot .

CMD ["./signal-bot"]
```

```bash
# Build and run
docker build -t signal-bot .
docker run -v ./signal-data:/root/signal-data \
    -e SIGNAL_PHONE_NUMBER=+1234567890 \
    signal-bot
```

### systemd Service

```ini
[Unit]
Description=Signal Bot
After=network.target

[Service]
Type=simple
User=signalbot
WorkingDirectory=/opt/signal-bot
Environment="SIGNAL_PHONE_NUMBER=+1234567890"
Environment="SIGNAL_DATA_PATH=/var/lib/signal-bot"
ExecStart=/opt/signal-bot/signal-bot
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## Automation Examples

### Welcome Messages

```go
// Send welcome message to new group members
func HandleGroupJoin(event *GroupEvent) error {
    if event.Type == "member_joined" {
        welcomeMsg := fmt.Sprintf(
            "Welcome %s! Feel free to ask questions.",
            event.Member.Name,
        )
        return sendMessage(event.GroupID, welcomeMsg)
    }
    return nil
}
```

### Scheduled Messages

```go
// Send daily reminders
func StartScheduledMessages() {
    ticker := time.NewTicker(24 * time.Hour)
    defer ticker.Stop()
    
    for range ticker.C {
        msg := "Daily reminder: Don't forget to check your tasks!"
        sendMessageToAllGroups(msg)
    }
}
```

### Command Router

```go
// Route commands to handlers
func RouteCommand(msg *Message) (*Response, error) {
    if !strings.HasPrefix(msg.Content, "/") {
        return nil, nil
    }
    
    parts := strings.Split(msg.Content, " ")
    command := strings.TrimPrefix(parts[0], "/")
    
    switch command {
    case "echo":
        return &Response{Text: strings.Join(parts[1:], " ")}, nil
    case "time":
        return &Response{Text: time.Now().Format(time.RFC822)}, nil
    case "ping":
        return &Response{Text: "Pong!"}, nil
    default:
        return &Response{Text: "Unknown command"}, nil
    }
}
```

## Gotchas & Tips

!!! warning "Phone Number Verification"
    Your Signal phone number must be verified before the bot can work.
    Keep your verification codes secure.

!!! tip "Rate Limiting"
    Signal may rate-limit excessive message sending. Implement delays between
    bulk messages:
    ```go
    time.Sleep(1 * time.Second) // Wait between messages
    ```

!!! note "Data Persistence"
    Signal CLI stores data locally. Back up the data directory regularly:
    ```bash
    tar -czf signal-backup.tar.gz signal-data/
    ```

!!! warning "Group Permissions"
    Make sure your bot account has necessary permissions in Signal groups
    to send messages and receive notifications.

!!! tip "Message Queueing"
    For high-volume bots, implement a message queue to avoid overwhelming
    Signal's servers:
    ```go
    queue := make(chan *Message, 100)
    go processMessageQueue(queue)
    ```

!!! danger "Privacy & Security"
    Signal is an encrypted messaging platform. Respect user privacy and
    don't store sensitive message content unnecessarily.

## Troubleshooting

### Bot Not Receiving Messages

1. **Check Signal CLI**: Ensure Signal CLI is running and registered
   ```bash
   signal-cli -u +NUMBER receive
   ```

2. **Verify Permissions**: Check that bot has permissions in groups

3. **Check Logs**: Enable debug logging
   ```bash
   export LOG_LEVEL=debug
   ```

### Connection Issues

```bash
# Test Signal CLI connection
signal-cli -u +NUMBER send -m "test" +TARGET

# Check network
curl https://textsecure-service.whispersystems.org/

# Restart daemon
killall signal-cli
signal-cli -u +NUMBER daemon
```

### Messages Not Sending

1. **Check Rate Limits**: Wait between messages
2. **Verify Recipient**: Ensure recipient number is correct
3. **Check Group Membership**: Verify bot is still in group

## Enhancement Ideas

- [ ] Add natural language processing (NLP)
- [ ] Implement conversation context tracking
- [ ] Add database for user preferences
- [ ] Create admin commands for bot management
- [ ] Add multi-language support
- [ ] Implement attachment handling (images, files)
- [ ] Create webhook integrations
- [ ] Add analytics and usage tracking
- [ ] Implement conversation logging (with privacy controls)
- [ ] Add voice note support

## Resources

- [Signal Messenger](https://signal.org/)
- [Signal CLI GitHub](https://github.com/AsamK/signal-cli)
- [Signal Protocol](https://signal.org/docs/)
- [Go Signal Client Library](https://github.com/signal-golang/signal-golang)
- [Signal API Documentation](https://signal.org/docs/)
