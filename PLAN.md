# PayNow Lunch Splitter - Improvement Plan

## Overview
Enhance the app for a fixed team (~5-10 people) doing regular lunch splits, with saved teammates, receipt scanning, history tracking, and sharing capabilities.

---

## Phase 1: Core Team Features

### 1.1 Save Teammates (LocalStorage)
- Store teammate list in browser localStorage
- Quick toggle to include/exclude teammates for each session
- Add/edit/remove teammates from saved list
- Default "select all" or "select none" buttons

### 1.2 Rounding Options
- Add rounding setting: nearest $0.10, $0.50, $1.00, or exact
- Option to round up (generous) or nearest
- Show original vs rounded amount

---

## Phase 2: Receipt Scanning (OCR)

### 2.1 Tesseract.js Integration
- Add camera/upload button for receipt photo
- Process image with Tesseract.js (runs in browser, free)
- Extract item names and prices automatically
- Review & edit extracted items before confirming
- Works offline after initial load

---

## Phase 3: History & Tracking

### 3.1 Split History Log
- Save each completed split to localStorage
- Show date, restaurant/description, total, participants
- View past split details
- Clear history option
- Export history as CSV (optional)

### 3.2 Quick Presets
- Save frequent restaurants/items as presets
- One-tap to load preset items
- Edit/delete presets

---

## Phase 4: Sharing

### 4.1 Share via Link
- Generate shareable URL with bill data encoded
- Recipients see read-only breakdown (no QR generation for them)
- Copy link or share via Web Share API (mobile)

### 4.2 WhatsApp/Telegram Quick Share
- Format breakdown as text message
- One-tap share to messaging apps

---

## Implementation Priority

| Priority | Feature | Effort |
|----------|---------|--------|
| 1 | Save Teammates | Low |
| 2 | Rounding Options | Low |
| 3 | Share via Link/Text | Medium |
| 4 | Split History | Medium |
| 5 | Quick Presets | Medium |
| 6 | Receipt OCR | High |

---

## Technical Notes

- All data stored in localStorage (no backend needed)
- Tesseract.js loaded on-demand (~2MB) only when OCR is used
- Share links use URL parameters or base64 encoding
- Stays as single HTML file for simplicity
