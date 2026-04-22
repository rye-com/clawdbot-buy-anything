# buy-anything

A Clawdbot skill that helps the user complete purchases from supported Amazon and Shopify stores after explicit confirmation.

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

Ask Clawdbot to buy something from Amazon or any supported Shopify store:

```
You: Buy this for me https://amazon.com/dp/B0DJLKV4N9

Clawdbot: I'll help you buy that! Where should I ship it?

You: John Doe, 123 Main St, San Francisco CA 94102, john@example.com, +14155551234

Clawdbot: Got it! What's your max purchase price? (I'll warn you before exceeding it)

You: $500

Clawdbot: Open this secure card entry page in your browser:
          https://mcp.rye.com/bt-card-capture
          Enter your card details there — they never touch this chat.
          After submitting, copy the token and paste it here.

You: d1ff0c32-a1b2-4c3d-8e4f-567890abcdef

Clawdbot: Order placed!
          Total: $358.44 (includes 3% service fee)
          Confirmation: RYE-ABC123

          Want me to save your card token for next time?
```

After saving, future purchases are one message, but Clawdbot should still confirm before ordering:

```
You: Buy this https://cool-store.com/products/gadget

Clawdbot: I can place that order using your saved shipping info and tokenized card reference.
          Confirm yes/no.
```

## How Payments Work

### The short version

When you make your first purchase, Clawdbot sends you to a secure card entry page in your browser. Your card details go directly to [BasisTheory](https://basistheory.com/) — a PCI-certified token vault — and are converted into a reusable token. Your card number never appears in chat. Only the token ID is used for purchases.

### Under the hood

```
Clawdbot provides card capture page
        |
        v
You enter card details
(BasisTheory secure iframes — card goes directly to BT vault)
        |
        v
BasisTheory returns token ID
You copy it back into chat
        |
        v
+----------------+
|   Rye API      | <-- Only sees the BT token ID, never your card
|   Purchase     |
+--------+-------+
         |
         v
Shopify: card forwarded directly to store
Amazon: token converted to payment server-side (3% fee)
```

1. **Clawdbot provides a secure page** — a hosted web page with BasisTheory's PCI-compliant card elements loads in your browser
2. **You enter your card** — card details are captured in BasisTheory's secure iframes and sent directly to their vault. The card never appears in chat or in any Clawdbot process
3. **BasisTheory returns a token** — a reusable token ID (e.g. `d1ff0c32-...`) that represents your card
4. **You paste the token** — copy the token from the page and paste it into chat. This is an opaque ID, not your card number
5. **Token is used to pay** — the Rye API receives only the token to place the order. For Shopify stores, the card is forwarded directly to the store's payment processor. For Amazon, it's converted server-side

### Saved cards

When you save your card for future purchases (opt-in only — Clawdbot will ask first), only the **BasisTheory token ID** is stored in local memory on your device — not your card number, expiry, or CVC. The token is a reusable payment reference, not the raw card number. Future purchases can reuse this token directly with no full card entry needed, although CVC may need to be refreshed later. Clawdbot should always ask for confirmation before placing any order with a saved token.

To remove your saved card:
- **Ask Clawdbot to forget your card token** — this deletes it from local memory and prevents future purchases through this skill
- **To also revoke the token from BasisTheory's vault** (so it can't be used by any system), contact [orders@rye.com](mailto:orders@rye.com)
- You can delete all saved data (card, address, spending limit) at any time by asking Clawdbot to forget it

## Guardrails

- Clawdbot should only place an order after an explicit yes/no confirmation in the same conversation
- Clawdbot should restate the item URL, quantity, shipping recipient, and max approved total before purchase
- Clawdbot should never store raw card details, expiry, or CVC
- Clawdbot should delete saved data immediately if the user asks

### Spending limit

On your first purchase, Clawdbot will ask you to set a maximum purchase price. If any order (including fees) would exceed your limit, Clawdbot will warn you and ask for confirmation before proceeding. You can say "no limit" to skip this.

You can update your limit at any time by telling Clawdbot:

```
You: Set my max purchase to $200
You: Remove my spending limit
```

### Order confirmations

Every order sends an email confirmation to the address you provide, so you always have a receipt and tracking info.

## Pricing & Shipping

- **Shopify stores**: Standard store pricing — no markup from us
- **Amazon**: 3% fee to cover transaction costs
- Amazon orders under $15 have a $6.99 shipping charge
- Amazon orders $15 and above get free 2-day Prime shipping

## Good to Know

- Shopify orders go through the store's normal checkout — the store handles fulfillment directly
- Amazon orders are processed through a third-party Amazon account — you can't connect orders to your personal Amazon account at this time
- You'll receive an email with order confirmation and details after each purchase
- For returns or refunds, contact orders@rye.com

## Legal

By using this skill, you agree to Rye's [Terms of Service](https://rye.com/terms-of-service) and [Privacy Policy](https://rye.com/privacy-policy).

## Support

- Issues: https://github.com/rye-com/clawdbot-buy-anything/issues
- Rye API docs: https://docs.rye.com
