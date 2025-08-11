# TradeBotPro Frontend

This repository contains the static frontend for the TradeBotPro website, including the payment success page that handles post-purchase fulfillment.

## Stripe Payment Link Configuration

To set up the "After payment" redirect URL in your Stripe Payment Link:

1. Go to your Stripe Dashboard
2. Navigate to **Payment Links**
3. Edit your existing payment link or create a new one
4. In the **After payment** section, set the redirect URL to:
   ```
   https://algotbp.com/success.html?session_id={CHECKOUT_SESSION_ID}
   ```

The `{CHECKOUT_SESSION_ID}` placeholder will be automatically replaced by Stripe with the actual session ID.

## Success Page Functionality

The `success.html` page handles post-purchase fulfillment by:

1. **Reading the session ID** from the query string parameter
2. **Polling the backend** at `GET /api/checkout-result?session_id=...` until fulfillment is ready
3. **Handling responses**:
   - `202` → Continues polling with modest exponential backoff
   - `200` → Displays the license key and automatically starts product download
4. **Displaying helpful status** text throughout the process
5. **Never exposing secrets** - only shows the license key returned by the backend
6. **Providing manual controls** like a copy button for the license key and manual download link

## API Configuration

The success page includes a configurable `API_BASE` setting at the top of the file:

### Same-Origin API (Default)
If you have a reverse proxy from `algotbp.com` to your Cloud Run backend, leave `API_BASE` empty:

```javascript
const API_BASE = '';
```

This will make requests to `/api/checkout-result` on the same domain.

### Direct Cloud Run URL
If there is no reverse proxy, set `API_BASE` to your Cloud Run URL:

```javascript
const API_BASE = 'https://your-service-name.run.app';
```

This will make requests to `https://your-service-name.run.app/api/checkout-result`.

## Backend Requirements

The backend must implement:

- **Endpoint**: `GET /api/checkout-result?session_id=...`
- **CORS**: Allow requests from `https://algotbp.com`
- **Response format**:
  - `202` for "still processing" (keep polling)
  - `200` with JSON containing `licenseKey` and `downloadUrl` when ready

Example successful response:
```json
{
  "licenseKey": "TBPRO-XXXX-YYYY-ZZZZ",
  "downloadUrl": "https://example.com/download/file.zip"
}
```