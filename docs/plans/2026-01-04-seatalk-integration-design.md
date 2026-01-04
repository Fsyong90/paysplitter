# Seatalk Bot Integration Design

**Date:** 4 Jan 2026
**Status:** Approved

## Overview

Integrate PaySplitter with Seatalk bot to enable receipt scanning and payment reminder posting directly from work chat.

## User Flow

```
1. User sends receipt photo to Seatalk bot
2. Bot scans with AI, extracts items
3. Bot replies with summary + paysplitter link
4. User opens link, assigns people, finalizes split
5. User clicks "Send to Seatalk" button
6. Bot posts one message per person with their amount + PayNow QR
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      SEATALK BOT                            │
└─────────────────┬───────────────────────────┬───────────────┘
                  │                           │
                  ▼                           ▼
┌─────────────────────────────┐   ┌───────────────────────────┐
│  WORKFLOW 1: Receipt Scan   │   │  WORKFLOW 2: Post Results │
│  n8n workflow               │   │  n8n workflow             │
└─────────────────────────────┘   └───────────────────────────┘
                                              ▲
┌─────────────────────────────────────────────┴───────────────┐
│                    PAYSPLITTER APP                          │
│  New: "Send to Seatalk" button                              │
└─────────────────────────────────────────────────────────────┘
```

## Workflow 1: Receipt Scan

**Trigger:** Seatalk webhook (image message)

**Steps:**
1. Receive POST from Seatalk, extract image_url and chat_id
2. Validate message contains image, else reply "Please send a receipt photo!"
3. Download image from Seatalk CDN
4. POST to ai.insea.io/api/workflows/10902/run (existing OCR API)
5. Parse response: items[], subtotal, service_charge, gst, total
6. Generate paysplitter URL with encoded data + chat_id parameter
7. Reply to Seatalk with summary + link

**Example Reply:**
```
🍜 Receipt Scanned!

Items found:
• Chicken Rice - $5.50
• Laksa - $8.00
• Teh Tarik x2 - $4.00

Subtotal: $17.50
GST (9%): $1.58
Total: $19.08

👉 Assign & split: https://fsyong90.github.io/paysplitter/?d=eyJp...&chat_id=xxx
```

## Workflow 2: Post Final Results

**Trigger:** HTTP webhook from paysplitter app

**Payload:**
```json
{
  "chat_id": "seatalk_chat_id",
  "breakdown": [
    { "name": "John", "amount": 8.50, "items": ["Chicken Rice"] },
    { "name": "Mary", "amount": 10.58, "items": ["Laksa", "Teh Tarik"] }
  ],
  "paynow_number": "+6583399525",
  "date": "4 Jan 2026"
}
```

**Steps:**
1. Receive POST with breakdown data
2. For each person:
   a. Generate PayNow QR image (using quickchart.io or similar)
   b. Post message to Seatalk with amount + QR image
3. Post final summary message

**Example Messages (one per person):**
```
💰 John owes S$8.50

[PayNow QR Image]

Scan to pay → +65 8339 9525
```

## Paysplitter App Changes

### 1. Detect Seatalk Source
- Check URL for `chat_id` parameter
- Store in variable: `seatalkChatId`
- URL format: `paysplitter/?d=<data>&chat_id=<seatalk_id>`

### 2. "Send to Seatalk" Button
- Only visible when `seatalkChatId` is present
- Placed in Share Actions row (next to WhatsApp)
- Button text: "💬 Seatalk"

### 3. Send Breakdown Function
```javascript
async function sendToSeatalk() {
    const payload = {
        chat_id: seatalkChatId,
        breakdown: getPersonData(),
        paynow_number: getPayNowNumber(),
        date: new Date().toLocaleDateString('en-GB', { day: 'numeric', month: 'short', year: 'numeric' })
    };

    await fetch(N8N_SEATALK_WEBHOOK_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
    });

    showToast('Sent to Seatalk! ✅');
}
```

## Implementation Order

1. **Workflow 1** - Receipt scan bot (n8n)
2. **App changes** - Add Seatalk button + send function
3. **Workflow 2** - Post results bot (n8n)
4. **Testing** - End-to-end flow

## Technical Notes

- Reuse existing ai.insea.io API (workflow 10902) for OCR
- QR generation: Use quickchart.io API or generate server-side
- Seatalk API: Reference existing reminder-bot implementation
- n8n webhook URLs will need to be configured in app

## Dependencies

- Seatalk bot token (existing)
- ai.insea.io API key (existing)
- n8n running on Raspberry Pi (existing)
- ngrok tunnel for webhook access (existing)
