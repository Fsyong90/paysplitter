# PayNow Lunch Splitter

A single-page web application for splitting bills among friends with PayNow QR code generation.

## Project Structure

```
paysplitter/
└── index.html    # Complete application (HTML + CSS + JavaScript)
```

This is a **zero-dependency, client-side only** application. No build system, no npm, no backend required.

## Technologies

- **HTML5/CSS3/JavaScript (ES6+)** - Vanilla, no frameworks
- **QRCode.js** (CDN) - PayNow QR code generation
- **Tesseract.js** (CDN) - OCR fallback for receipt scanning
- **Google Vision API** (optional) - Better OCR accuracy

## Key Features

1. **Bill Splitting** - Split expenses among people
2. **Receipt Scanning** - Upload photo, extract items via OCR
3. **PayNow QR Codes** - Generate Singapore PayNow QR for payments
4. **Team Management** - Save/reuse team members
5. **Payment Tracking** - Mark who has paid
6. **History** - Save and reload previous splits
7. **Sharing** - WhatsApp & URL sharing

## Data Storage

Uses `localStorage` with these keys:
- `paynow_team` - Saved team members
- `paynow_history` - Split history (max 50 entries)
- `paynow_google_api_key` - Google Vision API key (optional)

## Development

Simply open `index.html` in a browser. No build step required.

## OCR Options

1. **Tesseract.js** (default) - Free, runs in browser, ~70-80% accuracy
2. **Google Vision API** (optional) - Requires API key, ~95%+ accuracy, 1,000 free scans/month

To enable Google Vision:
1. Get API key from [Google Cloud Console](https://console.cloud.google.com/apis/credentials)
2. Enable "Cloud Vision API"
3. Enter key in app settings (below scan button)
