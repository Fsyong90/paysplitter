# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PayNow Lunch Splitter is a single-page HTML application for splitting restaurant bills among teams and generating Singapore PayNow QR codes for payment. It features real-time team sync via Firebase, AI-powered receipt scanning, and receipt image sharing.

**Live URL**: https://fsyong90.github.io/paysplitter/

## Architecture

**Single-file structure**: `index.html` contains all HTML, CSS, and JavaScript (~2800 lines).

### State Management

Global state arrays:
- `people[]` - current session participants
- `items[]` - order items with `{id, name, price, people[]}` where `people` tracks who shares each item
- `savedTeam[]` - persistent teammate list (synced to Firebase)
- `splitHistory[]` - past split records (synced to Firebase)
- `paidStatus{}` - tracks who has paid in current session
- `currentTeamCode` - Firebase team sync identifier

### Calculation Flow (Order matters!)

1. Subtotal from all items
2. **Discount applied FIRST** (before service charge)
3. Service charge calculated on subtotal-after-discount
4. GST calculated on (subtotal - discount + service)
5. Each person's share of extras is proportional to their item subtotal
6. Optional rounding (nearest $0.10, $0.50, $1.00)

### Firebase Integration

- **Firestore**: Team data sync (teammates, history, receipt URLs)
- **Storage**: Receipt image uploads (2MB limit)
- **Real-time listeners**: `onSnapshot()` for live updates across team members

Team code acts as a simple access key - anyone with the code can read/write team data.

### External Services

| Service | Purpose |
|---------|---------|
| Firebase Firestore | Team sync, history storage |
| Firebase Storage | Receipt image sharing |
| ai.insea.io API | Receipt OCR (workflow 10902) |
| qrcodejs (CDN) | QR code rendering |

### PayNow QR Generation

Uses EMVCo QR specification:
- `generatePayNowString()` - builds TLV (tag-length-value) payload
- `calculateCRC()` - CRC-16/CCITT checksum
- Hardcoded PayNow number: `+6583399525`

## Key Functions

| Function | Purpose |
|----------|---------|
| `connectToTeam(code)` | Subscribes to Firebase real-time updates |
| `handleReceiptUpload()` | Uploads to Storage + sends to AI API |
| `uploadReceiptToFirebase()` | Stores image, updates Firestore with URL |
| `calculate()` | Main calculation, updates all displays |
| `splitAllEqually()` | Assigns all items to all people |
| `generateQR()` | Creates PayNow QR for a person |

## API Keys (in code)

- **ai.insea.io**: `yz1PZMqqNAQRVXeOmie0gkaDft9I2l0R`
- **Firebase**: Config object with project `paysplitter-16bd6`

## Development

Open `index.html` directly in browser. No build step required.

To deploy: Push to `main` branch, GitHub Pages auto-deploys.
