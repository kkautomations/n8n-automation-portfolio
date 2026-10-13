# Order Processing Automation (n8n)

Receives new orders via webhook, validates them, saves them to Google Sheets
and emails the business owner. A separate error workflow sends an alert if anything fails.

## Flow
Webhook → Edit Fields → If (validate) → Google Sheets → Gmail → Respond to Webhook
                                     ↘ Respond to Webhook (400, rejected)

## What it handles
- Valid orders: saved and owner notified, with an order ID returned
- Invalid orders (no name, bad email, quantity 0): rejected with HTTP 400
- Failures: Error Trigger workflow emails an alert with node, error and execution ID

## Setup
1. Import both JSON files into n8n
2. Connect Google Sheets and Gmail credentials
3. Create a sheet with the headers: order_id, received_at, customer_name, email, phone, product, quantity, total, status
4. Set Order Errors as the error workflow in Order Processing settings
5. Publish and send a test order to /webhook/new-order

## Known limits
- No duplicate-order check
- Prices come from the sender, not looked up from a product list
