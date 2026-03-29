
# TIMOR DEI — Backend Setup Guide
## "The cathedral does not build itself."

---

## FOLDER STRUCTURE

Copy everything to:  C:\xampp\htdocs\timor-dei\

```
timor-dei/
│
├── index.html                  ← Your main frontend (timordei_final.html renamed)
├── manifest.json               ← PWA manifest
│
├── js/
│   └── cart-api.js             ← Frontend JS bridge to the backend
│
├── api/
│   ├── .htaccess               ← Security — blocks direct access to config.php
│   ├── config.php              ← DB credentials + helper functions
│   ├── process_order.php       ← Main checkout endpoint
│   ├── check_stock.php         ← Stock levels endpoint
│   └── update_order_status.php ← Post-payment status updater
│
├── database/
│   └── timor_dei_schema.sql    ← Run this in phpMyAdmin ONCE
│
└── logs/                       ← Auto-created; order log written here
    └── orders.log              ← (created automatically)
```

---

## STEP-BY-STEP SETUP

### 1. Start XAMPP
- Open XAMPP Control Panel
- Start **Apache** → green light
- Start **MySQL** → green light

### 2. Create the Database
1. Open browser → `http://localhost/phpmyadmin`
2. Click **"New"** in the left sidebar
3. Database name: `timor_dei`
4. Collation: `utf8mb4_unicode_ci`
5. Click **Create**
6. Click the **SQL** tab (top menu)
7. Open `database/timor_dei_schema.sql` in a text editor
8. Copy the ENTIRE contents and paste into phpMyAdmin SQL tab
9. Click **Go**
10. You should see: `timor_dei > products (3 rows), orders (0), order_items (0)`

### 3. Copy Files
Copy this entire `timor-dei/` folder to:
```
C:\xampp\htdocs\timor-dei\
```

Rename `timordei_final.html` → `index.html`

### 4. Link cart-api.js in index.html
Add this line BEFORE the closing `</body>` tag in index.html:

```html
<script src="js/cart-api.js"></script>
```

### 5. Test the Setup
Open browser → `http://localhost/timor-dei/`

**Test 1 — Stock Check:**
Go to: `http://localhost/timor-dei/api/check_stock.php`
You should see a JSON response with all 3 products.

**Test 2 — Place an Order:**
1. Click "Add to Cart" on any product
2. Open cart drawer
3. Use the PayPal sandbox or card form
4. After payment, check phpMyAdmin → `orders` table
5. You should see a new row with status = 'pending'

**Test 3 — Sold Out:**
In phpMyAdmin, set `stock_quantity = 0` for product ID 1:
```sql
UPDATE products SET stock_quantity = 0 WHERE id = 1;
```
Refresh the page — the "Rosa Mystica Hoodie" button should say "Sold Out".

---

## GOING LIVE (Production Checklist)

- [ ] Change `APP_ENV` in `config.php` to `'production'`
- [ ] Set real DB credentials (never use root with no password on production)
- [ ] Replace `PAYPAL_CLIENT_ID = 'test'` with your Live PayPal Client ID
      → Get it at: https://developer.paypal.com → My Apps & Credentials → Live
- [ ] Replace `STRIPE_PK = 'pk_test_...'` with your Live Stripe Publishable Key
      → Get it at: https://dashboard.stripe.com → API Keys
- [ ] Uncomment the production Stripe code in `cart-api.js` and create `create_payment_intent.php`
- [ ] Change `ALLOWED_ORIGIN` in `config.php` to your actual domain
- [ ] Move DB password out of `config.php` into an environment variable
- [ ] Enable HTTPS (SSL certificate)
- [ ] Set up a cron job to clean up old 'pending' orders

---

## PAYPAL LIVE KEY — How to Get It

1. Go to https://developer.paypal.com
2. Log in with your PayPal business account
3. Dashboard → My Apps & Credentials
4. Switch from "Sandbox" to **"Live"** toggle
5. Create App or select existing
6. Copy the **Client ID** (the long string)
7. In `timordei_final.html` find:
   ```html
   <script src="https://www.paypal.com/sdk/js?client-id=test&...">
   ```
   Replace `test` with your Client ID.
8. In `js/cart-api.js` find:
   ```js
   const PAYPAL_CLIENT_ID = 'test';
   ```
   Replace with your Client ID.

---

## TROUBLESHOOTING

**"Could not connect to database"**
→ Make sure MySQL is running in XAMPP
→ Check DB_NAME = 'timor_dei' in config.php

**Blank page / PHP errors**
→ Open `http://localhost/timor-dei/api/check_stock.php` directly
→ Look at XAMPP error logs: `C:\xampp\apache\logs\error.log`

**Cart doesn't send to backend**
→ Open browser DevTools → Network tab
→ Add item to cart and try to checkout
→ Look for the POST to `process_order.php`
→ Check the response for error messages

**PayPal button doesn't appear**
→ Make sure you're on http:// (not file://)
→ The PayPal SDK blocks non-HTTP origins

---

*"Architecture is the will of an epoch translated into space." — Mies van der Rohe*
*TIMOR DEI — Est. MMXXV*
