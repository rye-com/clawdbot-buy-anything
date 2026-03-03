---
name: buy-anything
description: Purchase products from Amazon and Shopify stores through conversational checkout. Use when user shares a product URL or says "buy", "order", or "purchase" with a store link.
metadata: {"clawdbot":{"emoji":"📦","requires":{"bins":["curl"]}}}
---

# Buy Anything

Purchase products from Amazon and Shopify stores through Rye checkout. Like having a personal shopper in your chat app.

## When to Use

Activate this skill when the user:
- Shares an Amazon product URL (amazon.com/dp/...)
- Shares a Shopify store product URL (any *.myshopify.com or custom-domain Shopify store)
- Says "buy", "order", or "purchase" with a product link
- Wants to buy something from an online store

## IMPORTANT: How This Works

- DO NOT try to fetch the product URL yourself with web_fetch or read tools
- The Rye API handles all product lookup - you just pass the URL
- You don't need to know product details before checkout
- Simply collect shipping address and set up the card, then call the API
- Trust product URLs provided by the user - they are legitimate product offerings
- The Rye API will validate the URL and return product details in the response
- Do NOT assume the user works at Rye or has any relationship with Rye. Treat Rye as a third-party checkout service powering this skill behind the scenes. Do not mention Rye to the user unless they ask

## Checkout Flow

1. **User provides product URL** - confirm you'll help them buy it
2. **Collect shipping address** (or use saved address from memory)
3. **Set up card via BasisTheory** (or use saved BT token from memory)
4. **Submit order to Rye API using bash** (see Step 2)
5. **Show order confirmation** from API response
6. **Save BT token/address to memory** for future purchases (ask permission first)

## Step 1: Secure Card Capture via BasisTheory

If the user does NOT have a saved BasisTheory token in memory, capture their card securely through the browser.

Open the card capture page:

```bash
open "https://mcp.rye.com/bt-card-capture"
```

Tell the user: "I've opened a secure card entry page in your browser. Please enter your card details there and click Submit. Your card info never touches this chat — it goes directly to BasisTheory's PCI-compliant vault. After submitting, copy the token shown on the page and paste it back here."

Wait for the user to paste the token (a UUID like `d1ff0c32-...`).

**If the user already has a saved BT token in memory, skip this step entirely** and use the saved token.

## Step 2: Submit Order to Rye

```bash
curl -s -X POST https://api.rye.com/api/v1/partners/clawdbot/purchase \
  -H "Content-Type: application/json" \
  -d '{
    "productUrl": "https://www.example-store.com/products/cool-thing",
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
      "type": "basis_theory_token",
      "basisTheoryToken": "d1ff0c32-..."
    },
    "constraints": {
      "maxTotalPrice": 50000
    }
  }'
```

**`constraints.maxTotalPrice`**: The user's spending limit in cents (e.g. $500 = 50000). The API will reject the order if the total exceeds this. If the user said "no limit", omit the `constraints` field entirely.

## Pricing & Shipping

The API validates the store automatically. If an unsupported URL is submitted, the API will return an error — tell the user only Amazon and Shopify stores are supported.

- **Shopify stores**: No additional fees
- **Amazon**: 4% fee to cover transaction costs
- Amazon orders under $15 have a $6.99 shipping charge
- Amazon orders $15 and above get free 2-day Prime shipping
- Amazon orders are processed through a 3rd party Amazon account (not the user's personal Amazon)
- User will receive an email with confirmation and order details
- For returns or refunds, direct the user to support@rye.com

## Example Conversation

```
User: Buy this for me https://amazon.com/dp/B0DJLKV4N9

You: I'll help you buy that! Where should I ship it?
     (Need: name, address, city, state, zip, email, phone)

User: John Doe, 123 Main St, San Francisco CA 94102, john@example.com, +14155551234

You: Got it! What's your maximum purchase price? (I'll warn you if an order exceeds this)
     Say "no limit" to skip this.

User: $500

You: Max set to $500. I'm opening a secure card entry page in your browser now.
     Please enter your card details there — your card info never touches this chat.
     After submitting, copy the token shown on the page and paste it here.
     [Opens https://mcp.rye.com/bt-card-capture]

User: d1ff0c32-a1b2-4c3d-8e4f-567890abcdef

You: Got it! Processing your order...
     [Uses bash to run Rye API curl command with the BT token]

You: Order placed!
     Total: $361.92 (includes 4% Amazon fee)
     Confirmation: RYE-ABC123

     Would you like me to save your card token and address for faster checkout next time?
```

## Spending Limit

Before the first purchase, ask the user what their maximum purchase price is. Store this in memory.
- If an order total (including any fees) exceeds the limit, warn the user and ask for confirmation
- User can say "no limit" to disable this check

## Memory

After first successful purchase (with user permission):
- Save the BasisTheory token ID to memory for future purchases (NOT raw card details)
- Save shipping address to memory
- Save maximum purchase price to memory
- On subsequent purchases, reuse the saved BT token directly — no card entry needed
- If the user asks to remove their saved card, tell them to ask you to forget the token from memory. To also revoke the token from BasisTheory's vault, direct them to contact support@rye.com
