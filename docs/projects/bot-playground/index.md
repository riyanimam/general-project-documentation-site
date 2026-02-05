# Bot Playground Projects

This section covers bot playground projects for experimenting with messaging platforms.

## Project Overview

| Project | Description | Language | Platform |
|---------|-------------|----------|----------|
| [Discord Bot Playground](discord-bot-playground.md) | Discord bot with essential commands | Go | Discord |
| [Signal Bot Playground](signal-bot-playground.md) | Signal messaging bot automation | Go | Signal |

## Categories

### Discord Bots

Tools for building Discord bot functionality.

<div class="grid cards" markdown>

- :material-discord:{ .lg .middle } **Discord Bot Playground**

    ---

    A simple and extensible Discord bot written in Go using DiscordGo library.
    Perfect as a starter project or playground.

    [:octicons-arrow-right-24: Learn more](discord-bot-playground.md)

</div>

### Signal Bots

Projects for Signal messenger automation.

<div class="grid cards" markdown>

- :material-bell:{ .lg .middle } **Signal Bot Playground**

    ---

    A Signal bot playground for building and testing messaging automations
    on the Signal platform.

    [:octicons-arrow-right-24: Learn more](signal-bot-playground.md)

</div>

## Common Patterns

### Go Best Practices

All Go-based bots follow consistent patterns:

```go
// Structured logging
import "log"

logger := log.New(os.Stdout, "BOT: ", log.LstdFlags)

// Graceful shutdown
sigChan := make(chan os.Signal, 1)
signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
<-sigChan

// Environment-based configuration
token := os.Getenv("BOT_TOKEN")
if token == "" {
    log.Fatal("BOT_TOKEN is required")
}
```

### Error Handling

Consistent error handling across bots:

```go
type BotError struct {
    Op      string
    Err     error
    Retryable bool
}

func (e *BotError) Error() string {
    return fmt.Sprintf("%s: %v", e.Op, e.Err)
}

func handleMessage(m *Message) error {
    if err := processMessage(m); err != nil {
        return &BotError{
            Op:      "process_message",
            Err:     err,
            Retryable: true,
        }
    }
    return nil
}
```

## Quick Links

- [Discord Bot Playground :material-arrow-right:](discord-bot-playground.md)
- [Signal Bot Playground :material-arrow-right:](signal-bot-playground.md)
