# Connector API Specs — Dhyaan (Ken Round 3)

Source-of-truth for the connectors the Dhyaan agent calls on AgenticOrg. The
**Delhivery mock must mirror these endpoint paths and field names exactly**
(Round 3 rule). Tokens shown as placeholders — never commit real keys.

Docs: Pine Labs https://www.pinelabs.com/docs · Delhivery https://one.delhivery.com/developer-portal · Gnani https://docs.gnani.ai/api/introduction/quick-start

---

## RAIL 1 — GNANI (VOICE) — real connector
Base URL: `https://api.vachana.ai` · Auth header: `X-API-Key-ID: <key>`

### Speech-to-Text (STT)
- REST: `POST /stt/v3` · `Content-Type: multipart/form-data`
  - Request: `audio_file` (binary: WAV/MP3/OGG/FLAC/AAC/M4A), `language_code` (BCP-47, e.g. `hi-IN`)
  - Response: `success` (bool), `request_id` (str), `transcript` (str), `model` (str)
- Streaming: `wss://api.vachana.ai/stt/v3/stream` · headers `x-api-key-id`, `lang_code` · 16-bit mono PCM (8/16 kHz, 1024-byte frames) → JSON events `{type, text, segment_id, latency}`

### Text-to-Speech (TTS)
- REST: `POST /api/v1/tts/inference` · `Content-Type: application/json`
  - Request: `text` (str), `voice` (str), `model` (e.g. `timbre-v2.5`), `language` (e.g. `hi-IN`), `speed` (num, default 1.0), `audio_config` (obj: `sample_rate`, `num_channels`, `sample_width`, `encoding`, `container`)
  - Response: binary audio (`Content-Type: audio/wav`)
- Streaming: SSE `/api/v1/tts/sse` (base64 chunks) · WS `wss://api.vachana.ai/api/v1/tts` (binary PCM)

---

## RAIL 3 — DELHIVERY (LOGISTICS) — SELF-HOSTED MOCK (Vercel), custom connector
Auth header on all: `Authorization: Token <key>`. Prod base `https://track.delhivery.com`, staging `https://staging-express.delhivery.com`. Mock must return different responses per request incl. failures (no serviceability, no rider, timeout, malformed, balance). Detail URL pattern: `one.delhivery.com/developer-portal/document/b2c/detail/<slug>`.

### 1. Pincode Serviceability  (slug: pincode-serviceability)
`GET /c/api/pin-codes/json/?filter_codes={pin}` (one pincode at a time)
- Empty list ⇒ non-serviceable (NSZ). `remark:"Embargo"` ⇒ temporary NSZ; blank remark ⇒ serviceable. Omitting `filter_codes` returns serviceable + embargoed.
- (Heavy variant: `GET /api/dc/fetch/serviceability/pincode?product_type=Heavy&pincode={pin}`)

### 2. Shipment Creation / Manifestation  (slug: order-creation)
`POST /api/cmu/create.json` · body form-encoded: `format=json&data={JSON}`
- JSON = `{ "shipments": [ {...} ], "pickup_location": { "name": "<warehouse_name>" } }`
- Mandatory shipment fields: `name`, `order` (unique Order ID), `phone`, `add`, `pin`, `pickup_location` (exact registered WH name, case/space sensitive), `payment_mode` (`Prepaid`|`COD`|`Pickup`(reverse)|`REPL`)
- Common optional: `city`, `state`, `country`, `weight` (gms, float), `waybill` (blank = auto-assign; required per box for MPS), `products_desc`, `cod_amount`, `total_amount`, `shipping_mode` (Surface/Express), `transport_speed` (`F`=NDD/`D`=standard), `address_type`, `shipment_height/width/length`, `fragile_shipment`, `return_*` fields, `seller_*`, `hsn_code`, `ewbn`, `quantity`
- Body rejects special chars `& # % ; \` → URL-encode.
- MPS extra: `shipment_type:"MPS"`, `master_id`, `mps_children`, `mps_amount` (prefetched waybills mandatory).

### 3. Shipment Tracking  (slug: order-tracking)
`GET /api/v1/packages/json/?waybill={wb}&ref_ids={order_id}` (up to 50 comma-sep waybills)
- Input waybill or order ID → returns current status + full scan history. Statuses incl. In Transit, Delivered, RTO.

### 4. Calculate Shipping Cost  (slug: calculate-shipping-cost)
`GET /api/kinko/v1/invoice/charges/.json?md={E|S}&cgm={grams}&o_pin={origin}&d_pin={dest}&ss={Delivered|RTO|DTO}&pt={Pre-paid|COD}`
- Optional: `l`, `b`, `h` (dims), `ipkg_type` (box/flyer). Returns approximate charges.

### 5. NDR API (delivery failure)  (slug: ndr-api)  — asynchronous
`POST /api/p/update` · body `{ "data": [ { "waybill": "...", "act": "RE-ATTEMPT" | "PICKUP_RESCHEDULE" } ] }` → returns **UPL ID**.
- RE-ATTEMPT valid NSL codes: EOD-74/15/104/43/86/11/69/6. PICKUP_RESCHEDULE: EOD-777/21 (cancelled, non-OTP). Attempt count 1 or 2. Apply after 9 PM.
- Status: `GET /api/cmu/get_bulk_upl/{UPL_ID}?verbose=true`

### Other B2C endpoints available (grab if needed)
Expected TAT API · Fetch WayBill API (bulk pre-fetch) · Shipment Updation · Shipment Cancellation · Ewaybill Update · Generate Shipping Label · Pickup Request Creation · Client Warehouse Creation/Updation · Webhook Functionality · RVP QC 3.0 · Download Document API. (Delhivery also exposes its own MCP — "Try Our Integration MCP".)

---

## RAIL 2 — PINE LABS (PAYMENTS) — platform-provided connector; mock gaps
Use AgenticOrg's working Pine Labs connector where available. P3P agentic protocol: consumer authorises a UPI/Cards mandate once; agent pays within it; Grantex supplies agent identity + delegated auth + spend controls + audit; every completed txn returns a cryptographically verifiable receipt. (Full endpoint field names on developer.pinelabs.com — JS-gated; confirm in platform connector or mock per Round 3 rule.) Needed capabilities: delegated agent payment, spend-limit/permission-scope check, payment status + verifiable receipt.

---

## ≤3 EXTRA MOCK CAPABILITIES (allowed on mock server)
Reserve for healthcare-specific gaps the 3 rails don't cover, e.g.: pharmacy
availability/price lookup, prescription/record store, doctor-appointment slots.
For each submission answer: partner + endpoint + what data that partner already holds.

## REAL TOOLS (must not be mocked)
Telegram (primary user channel — voice note in, approval card out), WhatsApp
(optional), Google Sheets (visible Shared Care State / audit log), Gmail (route
external inputs like a bank SMS). A teammate plays "the user" through the real tool.
