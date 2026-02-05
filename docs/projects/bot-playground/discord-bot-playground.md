# Discord Bot Playground

[![Go](https://img.shields.io/badge/Go-1.23-00ADD8?logo=go)](https://golang.org/)
[![DiscordGo](https://img.shields.io/badge/DiscordGo-Library-5865F2?logo=discord)](https://github.com/bwmarrin/discordgo)
[![Discord](https://img.shields.io/badge/Platform-Discord-5865F2?logo=discord)](https://discord.com/)

A simple and extensible Discord bot written in Go (Golang) using the DiscordGo library.
It includes essential commands like !ping, !help, !info, and utilities to fetch server
and user info — perfect as a starter project or playground for experimenting with Discord
bot development in Go.

[:fontawesome-brands-github: View on GitHub](https://github.com/riyanimam/discord-bot-playground){ .md-button }

## Features

- **Essential Commands**: Pre-built commands for ping, help, server info, and user info
- **DiscordGo Library**: Built on the popular Go Discord library
- **Extensible Architecture**: Easy to add new commands and features
- **Slash Commands**: Modern Discord slash command support
- **Event Handling**: Message and interaction event processing
- **Go Best Practices**: Clean code structure following Go conventions

## Prerequisites

- **Go** >= 1.23
- **Discord Bot Token** from [Discord Developer Portal](https://discord.com/developers/applications)
- **Discord Server** with permissions to add bots

## Quick Start

```bash
# Clone the repository
git clone https://github.com/riyanimam/discord-bot-playground.git
cd discord-bot-playground

# Install dependencies
go mod download

# Set up environment variables
export DISCORD_BOT_TOKEN="your-bot-token-here"

# Run the bot
go run main.go
```

## Project Structure

```text
discord-bot-playground/
├── cmd/
│   └── bot/
│       └── main.go           # Main entry point
├── internal/
│   ├── commands/
│   │   ├── ping.go          # Ping command
│   │   ├── help.go          # Help command
│   │   └── info.go          # Server/user info commands
│   ├── handlers/
│   │   └── message.go       # Message event handlers
│   └── config/
│       └── config.go        # Bot configuration
├── go.mod
├── go.sum
└── README.md
```

## Available Commands

### Prefix Commands

| Command | Description | Example |
|---------|-------------|---------|
| `!ping` | Check bot latency | `!ping` |
| `!help` | Show available commands | `!help` |
| `!serverinfo` | Display server information | `!serverinfo` |
| `!userinfo` | Display user information | `!userinfo @user` |

### Slash Commands

| Command | Description | Usage |
|---------|-------------|-------|
| `/ping` | Check bot latency | `/ping` |
| `/help` | Show available commands | `/help` |
| `/info` | Get server or user info | `/info server` or `/info user @mention` |

## Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DISCORD_BOT_TOKEN` | Your Discord bot token | Yes |
| `COMMAND_PREFIX` | Command prefix for text commands | No (default: `!`) |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) | No (default: `info`) |

### Bot Permissions

Required Discord bot permissions:

- **Send Messages**: Reply to commands
- **Read Message History**: Process messages
- **Use Slash Commands**: Register and respond to slash commands
- **View Channels**: Access channel information

## Adding Custom Commands

### Text Command

```go
package commands

import (
    "github.com/bwmarrin/discordgo"
)

// CustomCommand handles a custom command
func CustomCommand(s *discordgo.Session, m *discordgo.MessageCreate) {
    if m.Content == "!custom" {
        s.ChannelMessageSend(m.ChannelID, "This is a custom command!")
    }
}
```

### Slash Command

```go
package commands

import (
    "github.com/bwmarrin/discordgo"
)

var CustomSlashCommand = &discordgo.ApplicationCommand{
    Name:        "custom",
    Description: "A custom slash command",
}

func HandleCustomSlashCommand(s *discordgo.Session, i *discordgo.InteractionCreate) {
    s.InteractionRespond(i.Interaction, &discordgo.InteractionResponse{
        Type: discordgo.InteractionResponseChannelMessageWithSource,
        Data: &discordgo.InteractionResponseData{
            Content: "Custom slash command executed!",
        },
    })
}
```

## Development Workflow

### Running Locally

```bash
# Run with hot reload (using air)
air

# Run normally
go run cmd/bot/main.go

# Build binary
go build -o discord-bot cmd/bot/main.go
./discord-bot
```

### Testing

```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run tests with verbose output
go test -v ./...
```

### Code Quality

```bash
# Format code
go fmt ./...

# Run linter
golangci-lint run

# Vet code
go vet ./...
```

## Deployment

### Docker Deployment

```dockerfile
FROM golang:1.23-alpine AS builder

WORKDIR /app
COPY . .
RUN go mod download
RUN go build -o bot cmd/bot/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/bot .

CMD ["./bot"]
```

```bash
# Build and run with Docker
docker build -t discord-bot .
docker run -e DISCORD_BOT_TOKEN=$DISCORD_BOT_TOKEN discord-bot
```

### systemd Service

```ini
[Unit]
Description=Discord Bot
After=network.target

[Service]
Type=simple
User=botuser
WorkingDirectory=/opt/discord-bot
Environment="DISCORD_BOT_TOKEN=your-token"
ExecStart=/opt/discord-bot/bot
Restart=always

[Install]
WantedBy=multi-user.target
```

## Gotchas & Tips

!!! warning "Bot Token Security"
    Never commit your bot token to version control. Use environment variables
    or a secrets management system.

!!! tip "Rate Limiting"
    Discord has rate limits. Implement rate limiting in your bot to avoid
    being temporarily banned:
    ```go
    import "time"

    var lastCommand = make(map[string]time.Time)

    func checkRateLimit(userID string) bool {
        if last, ok := lastCommand[userID]; ok {
            if time.Since(last) < 3*time.Second {
                return false
            }
        }
        lastCommand[userID] = time.Now()
        return true
    }
    ```

!!! note "Slash Command Registration"
    Slash commands must be registered with Discord. This is done automatically
    when the bot starts, but can take a few minutes to propagate.

!!! warning "Gateway Intents"
    Make sure to enable required intents in the Discord Developer Portal:
    - Message Content Intent (for reading message content)
    - Guild Members Intent (for member information)

!!! tip "Testing Without Production"
    Create a test server for development to avoid disrupting your production
    Discord server.

## Troubleshooting

### Bot Doesn't Respond

1. **Check Token**: Verify your bot token is correct

   ```bash
   echo $DISCORD_BOT_TOKEN
   ```

2. **Check Permissions**: Ensure bot has required permissions in Discord server

3. **Check Intents**: Enable required intents in Developer Portal

### Slash Commands Not Appearing

1. **Wait for Propagation**: Can take up to 1 hour for global commands
2. **Use Guild Commands**: For instant updates during development:

   ```go
   _, err := s.ApplicationCommandCreate(appID, guildID, cmd)
   ```

3. **Clear Old Commands**: Remove outdated commands:

   ```go
   commands, _ := s.ApplicationCommands(appID, guildID)
   for _, cmd := range commands {
       s.ApplicationCommandDelete(appID, guildID, cmd.ID)
   }
   ```

### Connection Issues

```bash
# Enable debug logging
export LOG_LEVEL=debug
go run cmd/bot/main.go

# Check network connectivity
ping discord.com
```

## Enhancement Ideas

- [ ] Add database integration for persistent data
- [ ] Implement moderation commands (kick, ban, mute)
- [ ] Add music playback functionality
- [ ] Create admin dashboard
- [ ] Add multi-language support
- [ ] Implement custom emoji reactions
- [ ] Add role management commands
- [ ] Create scheduled message functionality
- [ ] Add integration with external APIs
- [ ] Implement custom embeds and rich messages

## Resources

- [Discord Developer Portal](https://discord.com/developers/docs/intro)
- [DiscordGo Documentation](https://pkg.go.dev/github.com/bwmarrin/discordgo)
- [Discord.js Guide](https://discordjs.guide/) (concepts apply to all bots)
- [Discord API Discord Server](https://discord.gg/discord-api)
- [Go Documentation](https://go.dev/doc/)
