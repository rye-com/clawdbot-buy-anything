# buy-anything

A Clawdbot skill for purchasing products from Amazon. Like having Alexa in your chat app.

## Installation

### Via ClawdHub (recommended)

```bash
npx clawdhub@latest install buy-anything
```

### Manual

Copy this folder to your Clawdbot skills directory:

```bash
# Global installation
cp -r . ~/.clawdbot/skills/buy-anything

# Or workspace installation
cp -r . ./skills/buy-anything
```

## Setup

1. Get a Rye API key at https://rye.com
2. Set the environment variable:

```bash
export RYE_API_KEY="your-api-key"
```

## Usage

Just ask Clawdbot to buy something from Amazon:

```
You: Buy this for me https://amazon.com/dp/B0DJLKV4N9

Clawdbot: I found the Sony WH-1000XM5 for $348.00.
          Where should I ship it?

You: John Doe, 123 Main St, San Francisco CA 94102

Clawdbot: Ready to checkout! Pay securely here:
          https://pay.rye.com/xxx

          Total: $361.92 (includes $13.92 service fee)
```

## Pricing

A 4% service fee is applied to all orders to cover payment processing.

## Support

- Issues: https://github.com/rye-com/clawdbot-buy-anything/issues
- Rye API docs: https://docs.rye.com
