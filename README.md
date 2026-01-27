# buy-anything

A Clawdbot skill for purchasing products from Amazon. Like having Alexa in your chat app.

## Installation

### Via ClawdHub (recommended)

```bash
# If you have clawdhub installed
clawdhub install buy-anything

# Or use npx
npx clawdhub install buy-anything
```

Don't have clawdhub? Install it:
```bash
npm install -g clawdhub
```

### Manual

Copy this folder to your Clawdbot skills directory:

```bash
# Global installation
cp -r . ~/.clawdbot/skills/buy-anything

# Or workspace installation
cp -r . ./skills/buy-anything
```

## Usage

Just ask Clawdbot to buy something from Amazon:

```
You: Buy this for me https://amazon.com/dp/B0DJLKV4N9

Clawdbot: I'll help you buy that! Where should I ship it?

You: John Doe, 123 Main St, San Francisco CA 94102, john@example.com, +14155551234

Clawdbot: Got it! Now I need your card details.
          Your card will be securely tokenized through Stripe.

You: 4242424242424242, 12/27, 123

Clawdbot: Order placed!
          Total: $361.92 (includes 4% fee)
          Confirmation: RYE-ABC123
```

## Pricing & Shipping

- A 4% fee is charged on all orders to cover transaction fees
- Orders under $15 have a $6.99 shipping charge
- Orders $15 and above get free 2-day Prime shipping

## Good to Know

- Orders are processed through a third-party Amazon account — you can't connect orders to your personal Amazon account at this time
- You'll receive an email with order confirmation and details after each purchase
- For returns or refunds, contact support@rye.com

## Legal

By using this skill, you agree to Rye's [Terms of Service](https://rye.com/terms-of-service) and [Privacy Policy](https://rye.com/privacy-policy).

## Support

- Issues: https://github.com/rye-com/clawdbot-buy-anything/issues
- Rye API docs: https://docs.rye.com
