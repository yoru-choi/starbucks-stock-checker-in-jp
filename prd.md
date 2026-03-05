# Starbucks Japan Stock Checker

## Overview

A GitHub Actions bot that automatically monitors stock availability of limited Starbucks Japan goods and sends an email notification when an item is back in stock.

---

## Background

Limited Starbucks Japan items (cups, mugs, etc.) are hard to track for restocks, and manually refreshing the page is tedious. The goal is to catch restocks quickly so purchase opportunities are not missed.

---

## How It Works

### Stock Detection

No login required. The bot checks the public goods exchange page using a Playwright headless browser (full JS execution) and reads the actual DOM state.

**Target page:** `https://www.starbucks.co.jp/mystarbucks/reward/exchange/original_goods/`

Stock status is determined by the CSS class on the out-of-stock button whose `data-jan` attribute matches the target JAN code:

| CSS classes on outofstock button | Meaning |
|---|---|
| `js-cartform-outofstock hide` | ✅ In stock (button is hidden) |
| `js-cartform-outofstock` | ❌ Out of stock (button is visible) |

> Playwright is used because the `hide` class is applied dynamically by JavaScript; plain HTTP requests return pre-render HTML where the class is not yet set.

### Schedule

| Mode | Trigger |
|---|---|
| **Automatic** | Every 4 hours via cron (`0 */4 * * *`) — UTC 0, 4, 8, 12, 16, 20 / JST 9, 13, 17, 21, 1, 5 |
| **Manual** | `Run workflow` button in the GitHub Actions UI |

### Notifications

- Email is sent via Gmail SMTP when stock is detected.
- Set `force_notify: true` on a manual run to send a test email regardless of stock status.

---

## Configuration

### GitHub Repository Secrets

| Name | Description |
|---|---|
| `GMAIL_USER` | Gmail address used to send notifications |
| `GMAIL_APP_PASSWORD` | Gmail App Password (16 characters, no spaces) |
| `NOTIFY_EMAIL` | Email address to receive notifications |

### GitHub Repository Variables

| Name | Description | Default |
|---|---|---|
| `JAN_CODE` | JAN code of the product to monitor | `4524785613492` |

---

## Monitored Products

| JAN Code | Notes |
|---|---|
| `4524785613492` | Target limited item (currently monitored) |
| `4524785531086` | Test item (has stock; useful for verifying detection logic) |

---

## File Structure

```
.github/
  workflows/
    check-stock.yml   # GitHub Actions workflow
prd.md                # This document
```
