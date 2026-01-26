---
name: buy-anything
description: Buy anything from Amazon. Just send a product link and I'll handle the rest.
---

# Buy Anything

Purchase products from Amazon through conversational checkout. Like having Alexa in your chat app.

## When to Use

- User wants to buy something from Amazon
- User shares an Amazon product URL
- User says "buy", "order", or "purchase" with an Amazon link

## Setup (Optional)

Add to your `~/.clawdbot/clawdbot.json`:

```json
{
  "skills": {
    "entries": {
      "buy-anything": {
        "config": {
          "maxPurchasePrice": 500
        }
      }
    }
  }
}
```

| Config | Description | Default |
|--------|-------------|---------|
| `maxPurchasePrice` | Maximum allowed purchase price in USD | No limit |

## Pricing

A 4% service fee is applied to all orders to cover payment processing.

## Flow

1. User provides Amazon product URL
2. Collect shipping address (or use saved address if available)
3. **Check price against maxPurchasePrice** - reject if over limit
4. Use saved card if available, otherwise ask user for card details
5. Tokenize card with Stripe
6. Submit order to Rye API
7. **Save card details to memory** for future purchases (ask user permission first)

## Tokenize Card with Stripe

Before submitting an order, tokenize the card with Stripe. Use this publishable key: `pk_live_xxx`

```
POST https://api.stripe.com/v1/tokens
Authorization: Bearer pk_live_xxx
Content-Type: application/x-www-form-urlencoded

card[number]=4242424242424242&card[exp_month]=12&card[exp_year]=2027&card[cvc]=123
```

Response contains `id` field - this is the token (e.g., `tok_xxx`). Use this token in the Rye API request.

## Submit Order to Rye

```
POST https://api.rye.com/api/v1/partners/clawdbot/purchase
Content-Type: application/json

{
  "productUrl": "https://amazon.com/dp/B0xxx",
  "quantity": 1,
  "buyer": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "phone": "+14155551234",
    "address1": "123 Main St",
    "city": "San Francisco",
    "province": "CA",
    "postalCode": "94102",
    "country": "US"
  },
  "paymentMethod": {
    "type": "STRIPE_TOKEN",
    "token": "tok_xxx"
  }
}
```

## Memory

- **Save card details to memory** after first successful purchase (with user permission)
- On subsequent purchases, use saved card - tokenize it fresh each time
- Also save shipping address for faster checkout

## Example Conversation (First Purchase)

```
User: Buy this for me https://amazon.com/dp/B0DJLKV4N9

Clawdbot: I found the Sony WH-1000XM5 Wireless Headphones for $348.00.
          Where should I ship it?

User: John Doe, 123 Main St, San Francisco CA 94102, john@example.com

Clawdbot: Got it! I'll need your card info to complete the purchase.

User: 4242424242424242, 12/27, 123

Clawdbot: Order placed!
          Sony WH-1000XM5 - $348.00 + $13.92 fee = $361.92
          Confirmation: RYE-ABC123
          Delivery: Jan 30 - Feb 1

          Would you like me to save your card and address for faster checkout next time?

User: Yes

Clawdbot: Saved! Next time you can just send me a link and I'll handle the rest.
```

## Example Conversation (Repeat Purchase)

```
User: Buy this https://amazon.com/dp/B0xxx

Clawdbot: I found the USB-C Cable for $12.99.
          Ship to John Doe, 123 Main St, San Francisco?

User: Yes

Clawdbot: Order placed!
          USB-C Cable - $12.99 + $0.52 fee = $13.51
          Confirmation: RYE-DEF456
          Delivery: Jan 29 - Jan 31
```

### Over Limit Example

```
User: Buy this https://amazon.com/dp/B0xxx (price: $750)

Clawdbot: This item costs $750, which exceeds your max purchase limit of $500.
          Would you like to update your limit or choose a different item?
```

## Supported Marketplaces

- Amazon (amazon.com, amazon.co.uk, etc.)

More marketplaces coming soon.
