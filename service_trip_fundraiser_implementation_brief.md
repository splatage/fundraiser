# Service Trip Fundraiser — Implementation Brief

## 1. Purpose

This document defines the v1 scope, economics assumptions, user flow, content, data model, and technical implementation plan for a **pickup-only pre-order Indian meal fundraiser** to support Lukas's Pacific Islands service trip.

This is designed for a **self-hosted implementation** with a simple custom order flow and an external hosted card checkout provider.

---

## 2. Event model

### Fundraiser format
- Pre-order only
- Pickup only
- Fixed menu
- Fixed prices
- Fixed pickup slots
- Online payment required before confirmation
- No on-site ordering
- No custom recipe modifications in v1

### Why this model
This model keeps the fundraiser operationally simple and economically strong:
- almost no food waste
- clear purchasing quantities after the order deadline
- simpler kitchen workflow for volunteer cooks
- easier admin and pickup handling
- predictable revenue and margin

---

## 3. v1 menu

## Individual meals
- Butter Chicken with Basmati Rice — **$18**
- Chana Masala with Basmati Rice — **$16**
- Dal with Basmati Rice — **$15**

## Family packs
- Butter Chicken Family Pack — **$55**
- Chana Masala Family Pack — **$47**
- Dal Family Pack — **$45**

## Optional add-ons
- Samosas — price configurable
- Dessert — price configurable

## v1 menu rules
- Rice included with all mains
- No spice-level selection
- No substitutions
- Notes field is for allergy alerts only
- All products have fixed SKUs and fixed pricing

---

## 4. Operational assumptions

## Kitchen and staffing
- Two volunteer cooks
- No kitchen rental cost
- Pickup handled by volunteers
- Food prepared only for paid orders

## Portion standardisation
These are planning standards for costing and batching.

### Individual meal
- 1 curry portion
- rice equivalent to approximately 100g dry rice

### Family pack
- equivalent of 4 individual portions

### Butter chicken individual
- approximately 150g raw chicken before cooking

### Chana individual
- approximately half a can chickpeas plus sauce

### Dal individual
- approximately 100g dry lentils plus sauce

## Pickup slots
Initial recommendation:
- 4:30–5:00 pm
- 5:00–5:30 pm
- 5:30–6:00 pm
- 6:00–6:30 pm

Slot capacity recommendation:
- 20–25 orders per slot to start

---

## 5. Economics model

## Cash cost categories
For this fundraiser, direct cash costs are mainly:
- ingredients
- packaging
- payment processing fees

There is no kitchen rental cost in scope.

## Economic principle
Each paid order should be positive-margin.

### Core formula
**Profit per item = sell price - ingredient cost - packaging cost - payment fee**

### Operational threshold
Because this is pre-order only, the more useful threshold is not financial break-even but:

**minimum total order volume worth running for the cooks and volunteers**

Recommended operational threshold:
- minimum proceed threshold: **40 individual-equivalent meals**
- strong target range: **70–100 individual-equivalent meals**

---

## 6. Customer-facing page copy

## Landing page hero
**Lukas Pacific Islands Service Trip Fundraiser**

Support Lukas by ordering a delicious homemade Indian meal.

Two talented volunteer cooks are generously preparing a limited menu of Indian dishes, with all proceeds going toward Lukas's upcoming Pacific Islands service trip.

This is a **pre-order pickup fundraiser**. Order online, choose your pickup slot, and collect your meal on the day.

## Key details block
- **Order deadline:** [insert date and time]
- **Pickup date:** [insert date]
- **Pickup location:** [insert location]
- **Pickup format:** pickup only
- **Cause:** supporting Lukas's service trip to the Pacific Islands

## Menu intro copy
We have kept the menu simple to help our volunteer cooks prepare everything well and keep this fundraiser efficient.

All main meals include basmati rice. Please note that menu items are fixed, and custom changes are not available for this fundraiser.

## Ordering notes copy
Please place your order before the deadline and complete payment online to confirm your order.

The notes field is for essential allergy alerts only. We are not able to offer custom spice levels or substitutions.

## Pickup info copy
Please arrive during your selected pickup window. Bring your confirmation email or order number to help us serve you quickly.

## Thank-you copy
Thank you for supporting Lukas and this trip. Every order helps, and sharing the fundraiser with others is also a huge help.

---

## 7. Product definitions

Suggested SKUs:
- `butter-chicken-ind`
- `chana-ind`
- `dal-ind`
- `butter-chicken-family`
- `chana-family`
- `dal-family`
- `samosa-addon`
- `dessert-addon`

Product fields:
- id
- sku
- name
- short_description
- long_description nullable
- price_cents
- category
- active
- max_available nullable
- display_order
- image_url nullable
- created_at
- updated_at

Suggested categories:
- main_individual
- main_family
- add_on

---

## 8. Customer form fields

## Required fields
- first_name
- last_name
- mobile
- email
- pickup_slot_id
- items[]
- terms_accepted

## Optional fields
- notes

## Notes field rule
Notes must be restricted in copy and backend validation to:
- allergy alert only
- short message length limit, for example 200 characters

## Item selection structure
Each selected item should contain:
- sku
- quantity

Quantities must be positive integers.

---

## 9. User flow

## Public flow
1. Customer lands on fundraiser page
2. Customer reads event details and menu
3. Customer selects one or more items
4. Customer selects a pickup slot
5. Customer enters contact details
6. Customer accepts terms
7. Frontend sends order request to backend
8. Backend validates products, slot availability, pricing, and totals
9. Backend creates order in `pending_payment`
10. Backend creates hosted checkout session with payment provider
11. Customer completes payment on hosted checkout page
12. Payment webhook marks order as `paid`
13. Confirmation email is sent
14. Customer arrives at pickup and staff marks order as `collected`

## Admin flow
1. Admin logs in
2. Admin views order list
3. Admin filters by paid/unpaid, collected/not collected, or pickup slot
4. Admin views item totals for kitchen prep
5. Admin views slot totals for pickup staffing
6. Admin marks orders as collected during pickup
7. Admin exports CSV if needed

---

## 10. Terms and policies

Suggested order policy text:

By placing an order, you confirm that:
- your order is for pickup only
- your selected pickup slot is your collection window
- your order is only confirmed once payment is successfully completed
- menu items are fixed and cannot be customised
- notes are for allergy alerts only
- refund requests after the order deadline may not be possible because food quantities are prepared to match confirmed orders

Suggested allergy text:

Our kitchen will be handling ingredients that may include dairy, gluten, nuts, and other allergens. We cannot guarantee an allergen-free environment.

---

## 11. Technical architecture

## Recommended approach
Use a **self-hosted order application** for everything except raw card handling.

### Recommended split
- self-hosted frontend and backend
- self-hosted database
- hosted card checkout provider for payments
- webhook callback to your backend
- SMTP email using self-hosted infrastructure

## Why this is the right split
This keeps the user experience controlled while avoiding the complexity and risk of building your own card-entry form and storing or proxying card details.

---

## 12. Suggested stack

### Frontend
Either of these is suitable:
- plain server-rendered pages with light JavaScript
- simple React/Vite frontend

### Backend
- Node.js with Express

### Database
- MariaDB or PostgreSQL

### Email
- SMTP via your own mail relay or transactional email service

### Payment
- Hosted payment checkout with webhook support

---

## 13. Database schema

## `products`
```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sku VARCHAR(64) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    short_description VARCHAR(255) NOT NULL,
    long_description TEXT NULL,
    category VARCHAR(64) NOT NULL,
    price_cents INT NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    max_available INT NULL,
    display_order INT NOT NULL DEFAULT 0,
    image_url VARCHAR(1024) NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## `pickup_slots`
```sql
CREATE TABLE pickup_slots (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    label VARCHAR(64) NOT NULL,
    starts_at DATETIME NOT NULL,
    ends_at DATETIME NOT NULL,
    capacity INT NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## `orders`
```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    public_order_number VARCHAR(32) NOT NULL UNIQUE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    mobile VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    pickup_slot_id BIGINT NOT NULL,
    notes VARCHAR(500) NULL,
    subtotal_cents INT NOT NULL,
    fee_cents INT NOT NULL,
    total_cents INT NOT NULL,
    payment_status VARCHAR(32) NOT NULL,
    fulfillment_status VARCHAR(32) NOT NULL,
    payment_provider VARCHAR(32) NULL,
    payment_provider_session_id VARCHAR(255) NULL,
    payment_provider_intent_id VARCHAR(255) NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    paid_at DATETIME NULL,
    collected_at DATETIME NULL,
    CONSTRAINT fk_orders_pickup_slot FOREIGN KEY (pickup_slot_id) REFERENCES pickup_slots(id)
);
```

## `order_items`
```sql
CREATE TABLE order_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    sku_snapshot VARCHAR(64) NOT NULL,
    name_snapshot VARCHAR(255) NOT NULL,
    unit_price_cents INT NOT NULL,
    quantity INT NOT NULL,
    line_total_cents INT NOT NULL,
    CONSTRAINT fk_order_items_order FOREIGN KEY (order_id) REFERENCES orders(id),
    CONSTRAINT fk_order_items_product FOREIGN KEY (product_id) REFERENCES products(id)
);
```

## `admin_users`
```sql
CREATE TABLE admin_users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(32) NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## `webhook_events`
```sql
CREATE TABLE webhook_events (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    provider VARCHAR(32) NOT NULL,
    event_id VARCHAR(255) NOT NULL,
    event_type VARCHAR(255) NOT NULL,
    processed BOOLEAN NOT NULL DEFAULT FALSE,
    received_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uq_provider_event (provider, event_id)
);
```

---

## 14. Status definitions

### `payment_status`
- `pending_payment`
- `paid`
- `failed`
- `expired`
- `refunded`

### `fulfillment_status`
- `not_ready`
- `ready_for_pickup`
- `collected`
- `cancelled`

Recommended v1 default:
- set `fulfillment_status = not_ready` when order is created
- optionally bulk-move all paid orders to `ready_for_pickup` on the event day

---

## 15. API contract

## Public endpoints

### `GET /api/public/products`
Returns active products in display order.

#### Response
```json
{
  "products": [
    {
      "sku": "butter-chicken-ind",
      "name": "Butter Chicken with Basmati Rice",
      "description": "Individual meal",
      "priceCents": 1800,
      "category": "main_individual"
    }
  ]
}
```

### `GET /api/public/pickup-slots`
Returns active pickup slots with remaining capacity.

#### Response
```json
{
  "pickupSlots": [
    {
      "id": 1,
      "label": "4:30–5:00 pm",
      "startsAt": "2026-05-15T16:30:00",
      "endsAt": "2026-05-15T17:00:00",
      "remainingCapacity": 18
    }
  ]
}
```

### `POST /api/public/orders/create-checkout-session`
Creates an order and returns hosted checkout URL.

#### Request
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "mobile": "0211234567",
  "email": "john@example.com",
  "pickupSlotId": 2,
  "notes": "Dairy allergy alert",
  "items": [
    { "sku": "butter-chicken-ind", "quantity": 2 },
    { "sku": "samosa-addon", "quantity": 1 }
  ],
  "termsAccepted": true
}
```

#### Success response
```json
{
  "publicOrderNumber": "LST-10027",
  "checkoutUrl": "https://payments.example/checkout/..."
}
```

#### Validation failure response
```json
{
  "error": "pickup_slot_full",
  "message": "The selected pickup slot is no longer available."
}
```

### `GET /api/public/orders/:publicOrderNumber`
Returns the paid order summary for customer success page.

---

## Payment webhook endpoint

### `POST /api/payments/webhook`
Responsibilities:
- verify webhook signature
- deduplicate by event id
- find internal order using provider metadata
- mark order paid if event indicates successful completion
- set `paid_at`
- trigger confirmation email
- record webhook event as processed

---

## Admin endpoints

### `POST /api/admin/login`
Authenticates admin and issues session or token.

### `GET /api/admin/orders`
Supports filters:
- `paymentStatus`
- `fulfillmentStatus`
- `pickupSlotId`
- `search`

### `GET /api/admin/orders/:id`
Returns full order details.

### `POST /api/admin/orders/:id/mark-collected`
Marks a paid order as collected.

### `GET /api/admin/reports/item-totals`
Returns kitchen prep counts.

### `GET /api/admin/reports/pickup-slot-summary`
Returns pickup staffing summary.

### `GET /api/admin/export/orders.csv`
Returns CSV export of orders.

---

## 16. Backend validation rules

The backend must enforce all of the following:
- order deadline has not passed
- at least one item has been selected
- all selected products exist and are active
- quantities are positive integers within configured limits
- pickup slot exists and is active
- pickup slot has remaining capacity
- subtotal is computed server-side
- any payment fee is computed server-side
- total is computed server-side
- notes length is limited
- terms must be accepted
- duplicate webhook events must be harmless

## Capacity handling
Slot capacity should be enforced using a transaction or equivalent concurrency-safe method so that two final orders cannot overfill a slot at the same time.

---

## 17. Admin UI requirements

## Orders list columns
- order number
- customer name
- mobile
- email
- pickup slot
- payment status
- fulfillment status
- total
- created at

## Orders list actions
- view details
- mark collected
- resend confirmation email

## Kitchen summary page
Show aggregated counts by SKU, for example:
- butter chicken individual: 32
- chana individual: 18
- dal individual: 12
- butter chicken family: 14
- samosa add-on: 21

## Pickup summary page
Show for each slot:
- slot label
- paid order count
- customer names
- mobile numbers
- collected count

---

## 18. Confirmation email template

### Subject
Your Lukas Service Trip Fundraiser order is confirmed

### Body
Hi [First Name],

Thank you for supporting Lukas's Pacific Islands service trip.

Your order has been received and paid successfully.

**Order number:** [Order Number]
**Pickup slot:** [Slot Label]
**Pickup location:** [Pickup Location]

**Your order:**
- [Item] x [Qty]
- [Item] x [Qty]

**Total paid:** $[Amount]

Please arrive during your selected pickup window and bring this email or your order number with you.

If you included an allergy note, please remind the team at pickup.

Thank you again for your support.

---

## 19. Security requirements

### Payment security
- do not collect raw card details on your own frontend
- use hosted checkout
- verify webhook signatures
- store provider references only

### App security
- admin authentication required for admin pages
- password hashes only, never plain-text passwords
- CSRF protection for admin session flows
- input validation on all public endpoints
- rate limit public order creation endpoint
- rate limit admin login endpoint
- server-side audit logging for admin actions

### Privacy
Store only the data needed to run the fundraiser:
- name
- mobile
- email
- order data
- allergy note if supplied

Do not request unnecessary personal information.

---

## 20. Accessibility and usability requirements

- mobile-first layout
- large tap targets for quantities and slot selection
- clear running total
- visible order deadline
- visible pickup location and date
- clear allergy text
- confirmation state after successful payment
- plain language throughout

---

## 21. Suggested page structure

## Public fundraiser page
1. Hero heading
2. Brief story and purpose
3. Date/location/deadline details
4. Menu cards
5. Pickup info
6. Order form
7. FAQ / policy section

## Success page
1. Thank-you message
2. Order number
3. Pickup slot
4. Pickup location
5. Item summary
6. Contact info for questions

## Admin pages
- login
- dashboard
- orders list
- order detail
- kitchen totals
- pickup summary

---

## 22. Recommended implementation phases

## Phase 1 — foundation
- products table
- pickup_slots table
- admin_users table
- initial seed data
- public menu endpoint
- pickup slots endpoint

## Phase 2 — order creation
- create order endpoint
- validation logic
- server-side price calculation
- order persistence
- hosted checkout session creation

## Phase 3 — payment confirmation
- webhook endpoint
- idempotent event handling
- mark orders paid
- send confirmation email

## Phase 4 — admin tools
- admin login
- orders list
- order detail page
- mark collected action
- kitchen totals report
- pickup slot report
- CSV export

## Phase 5 — hardening
- rate limiting
- deadline enforcement
- product caps
- resend confirmation email
- refund/admin override tooling if needed

---

## 23. Seed data example

### Products
```sql
INSERT INTO products (sku, name, short_description, category, price_cents, active, display_order)
VALUES
('butter-chicken-ind', 'Butter Chicken with Basmati Rice', 'Individual meal', 'main_individual', 1800, TRUE, 10),
('chana-ind', 'Chana Masala with Basmati Rice', 'Individual meal', 'main_individual', 1600, TRUE, 20),
('dal-ind', 'Dal with Basmati Rice', 'Individual meal', 'main_individual', 1500, TRUE, 30),
('butter-chicken-family', 'Butter Chicken Family Pack', 'Serves approximately 4', 'main_family', 5500, TRUE, 40),
('chana-family', 'Chana Masala Family Pack', 'Serves approximately 4', 'main_family', 4700, TRUE, 50),
('dal-family', 'Dal Family Pack', 'Serves approximately 4', 'main_family', 4500, TRUE, 60);
```

---

## 24. Open configuration values

These should be environment or admin-configurable:
- event name
- order deadline
- pickup location
- pickup date
- pickup slot capacities
- product prices
- product availability
- payment provider keys
- SMTP settings
- admin bootstrap credentials

---

## 25. Key decisions locked for v1

The following decisions should remain fixed unless there is a strong reason to change them:
- pickup only
- pre-order only
- prepaid only
- fixed menu
- fixed prices
- no customisations
- hosted checkout for payments
- simple admin dashboard
- minimal data collection

---

## 26. Recommended immediate next step

Build the project as a small MVP in this order:
1. database schema and seed data
2. public menu page and slot endpoint
3. order creation flow and hosted checkout redirect
4. webhook payment confirmation
5. confirmation email
6. admin order list and kitchen reports

This is enough to launch the fundraiser cleanly without overbuilding.

