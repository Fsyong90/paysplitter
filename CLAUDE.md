# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PayNow Lunch Splitter is a single-page HTML application for splitting restaurant bills among multiple people and generating Singapore PayNow QR codes for payment. It runs entirely in the browser with no build system or server required.

## Architecture

**Single-file structure**: `paynow-lunch-splitter.html` contains all HTML, CSS, and JavaScript.

**State management**: Two global arrays manage application state:
- `people[]` - list of participant names
- `items[]` - list of order items with `{id, name, price, people[]}` where `people` tracks who shares each item

**Calculation flow**:
1. Per-item costs are split equally among assigned participants
2. Service charge applied to subtotal (before GST)
3. Discount applied after service charge
4. GST calculated on (subtotal + service - discount)
5. Each person's share of service/discount/GST is proportional to their item subtotal

**PayNow QR generation**: Uses EMVCo QR specification with:
- `generatePayNowString()` - builds the payload with TLV (tag-length-value) encoding
- `calculateCRC()` - computes CRC-16/CCITT checksum required by spec
- Hardcoded phone number: `+6583399525`

## External Dependencies

- `qrcodejs` (CDN): QR code rendering library loaded from cdnjs

## Development

Open the HTML file directly in a browser. No build step required.
