# DFRNT API Explorer

A self-contained developer tool for integrating with the Urgent Couriers / DFRNT API. Built for external developers integrating on behalf of mutual customers.

---

## What It Is

A browser-based API client (like Postman, but DFRNT-specific) with:
- All endpoints pre-loaded with correct sample payloads
- Live request sending directly to the API
- An AI assistant (GPT-4o) that auto-analyzes errors and suggests fixes
- No backend required — single HTML file

---

## Quick Start

### Option 1 — Open locally
```bash
open dfrnt-api-explorer/index.html
```
Works in any modern browser. No server needed.

### Option 2 — Host on Cloudflare Pages / Netlify / S3
Upload `index.html` to any static host. Share the URL with external devs.

---

## How to Get a Bearer Token

Developers need a token from the customer's DFRNT account:

1. Log into the customer's DFRNT account
2. Go to **Settings → API Tokens**
3. Click **Generate Token**
4. Copy the token and paste it into the API Explorer's token field (top right)

> ⚠️ Tokens are scoped to the customer account. Each integration partner should use a token from the specific customer they're building for.

---

## AI Assistant Setup

The AI assistant uses OpenAI (GPT-4o). Developers need their own OpenAI API key:

1. Get a key at [platform.openai.com](https://platform.openai.com)
2. On first load, paste the key into the setup modal
3. The key is stored in the browser's `localStorage` — it never leaves the browser

The AI will:
- Automatically analyze any error response and explain what went wrong
- Suggest the corrected JSON snippet
- Answer follow-up questions about the API with full request/response context

---

## Endpoints Covered

### Rates
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/Rates` | Get available services with pricing |
| POST | `/api/Rates/GetSchedulerRates` | Get scheduled services with pricing |
| POST | `/api/Rates/GetRerateAmount` | Rerate a job by speed |

### Jobs
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/Jobs/{id}` | Get job by ID |
| GET | `/api/Jobs/{reference}/{date}` | Get job by reference + date |
| POST | `/api/Jobs` | Create a new pickup job |
| POST | `/api/Jobs/Topup` | Create a one-off topup job |
| PUT | `/api/Jobs/{id}` | Update a job |
| DELETE | `/api/Jobs/{id}` | Delete a job |
| POST | `/api/Jobs/Release/{id}` | Release an OnHold/prebook job |
| POST | `/api/Jobs/SavedBooking` | Create a saved booking |
| DELETE | `/api/Jobs/SavedBooking/{referenceB}` | Delete a saved booking |

### Address Validation
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/Jobs/ValidateUTAddress/{address}` | Validate address is in UT area |
| POST | `/api/Jobs/ValidateUTAddress` | Validate geolocation is in UT area |

### Webhooks
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/Webhook` | List all subscriptions |
| POST | `/api/Webhook` | Create a subscription |
| DELETE | `/api/Webhook/{id}` | Delete a subscription |

### Labels
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/JobLabel/{id}` | Get printable label for a job |

---

## Typical Integration Flow

```
1. POST /api/Rates          → get SpeedId + QuoteId for the route
2. POST /api/Jobs           → create job using SpeedId + QuoteId from step 1
3. GET  /api/Jobs/{id}      → poll for job status
4. POST /api/Webhook        → (optional) subscribe to JobStatus events
```

---

## Address Requirements

### NZ Addresses (`CountryCode: "NZ"`)
| Field | Required |
|-------|----------|
| StreetAddress | ✅ |
| Suburb | ✅ |
| City | ✅ |
| PostCode | ✅ |
| CountryCode | ✅ |
| Latitude / Longitude | Recommended |
| VehicleSizeId | ❌ Not required |

### US Addresses (`CountryCode: "US"`)
| Field | Required |
|-------|----------|
| StreetAddress | ✅ |
| City | ✅ |
| State | ✅ |
| ZipCode | ✅ |
| CountryCode | ✅ |
| VehicleSizeId | ✅ Required at request level |

> **Package weight field:** Use `Kg` for NZ, `Lb` for US.

---

## Key Field Reference

### JobType
| Value | Meaning |
|-------|---------|
| 1 | Pickup (from client, deliver to address) |
| 2 | Deliver to us |
| 3 | Third party |

### JobNotificationType
| Value | Meaning |
|-------|---------|
| `EMAIL` | Email notification (default) |
| `TEXT` | SMS notification |
| `WEBSITE` | Web notification |
| `EMAILANDTEXT` | Both email and SMS |

### StorageState
`null` (default) · `Ambient` · `Chilled` · `Frozen`

### Webhook EventType
Currently only `JobStatus` is supported.

---

## Common Errors & Fixes

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| 400 — Missing required field | `Suburb`, `PostCode`, or `StreetAddress` missing | Check NZ address requirements above |
| 400 — No Rates Found | Route not serviceable, or bad coordinates | Verify addresses and lat/lng |
| 401 — Unauthorized | Token missing, expired, or invalid | Regenerate token in customer settings |
| 404 — Not Found | Wrong Job ID or reference | Double-check the ID |
| 400 — VehicleSizeId required | US address without VehicleSizeId | Add `"VehicleSizeId": 1` to request |

---

## Files

```
dfrnt-api-explorer/
├── index.html     # The entire tool — single self-contained file
└── README.md      # This document
```

---

## Hosting Recommendations

| Option | Notes |
|--------|-------|
| **Cloudflare Pages** | Free, fast, deploys from GitHub |
| **Netlify** | Free tier, drag-and-drop deploy |
| **AWS S3 + CloudFront** | If already in AWS ecosystem |
| **Internal server** | Any static file server works |

For public developer access, recommend hosting behind the DFRNT developer docs subdomain (e.g. `tools.urgent.co.nz` or `developer.urgent.co.nz`).

---

## Maintenance

The tool is built directly from the API source code in `git.customd.com/urgent-couriers/api`. When new endpoints are added:

1. Add the endpoint definition to the `ENDPOINTS` object in `index.html`
2. Add a sidebar entry with the correct method tag
3. Update this README

---

*Generated by Clawdia 🦞 — 2026-03-05*
