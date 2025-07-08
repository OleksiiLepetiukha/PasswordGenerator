
# 🛠️ Shopify Custom Checkout Integration Guide

## 📑 Table of Contents

- [Prerequisites](#prerequisites)
- [Chapter 1: Creating Your Shopify App](#chapter-1-creating-your-shopify-app)
- [Chapter 2: Adding a Custom Checkout Domain (NGINX Setup)](#chapter-2-adding-a-custom-checkout-domain-nginx-setup)
- [Chapter 3: Registering Your Shop in the Checkout Backend](#chapter-3-registering-your-shop-in-the-checkout-backend)
- [Chapter 4: Implementing the Custom Checkout Button](#chapter-4-implementing-the-custom-checkout-button)
- [Testing the Integration](#testing-the-integration)

---

## ✅ Prerequisites

Before starting, make sure you have:

- Admin access to your Shopify store
- Access to the checkout backend API
- A Linux server (with NGINX) or frontend deployment on Vercel/Netlify
- DNS access to create subdomains (e.g., `checkout.yourstore.com`)
- Basic understanding of Shopify themes and Liquid templating

---

## 📦 Chapter 1: Creating Your Shopify App

### Step 1: Navigate to App Development

- In your Shopify admin panel:  
  `Settings → Apps and sales channels → Develop apps`  
- Click **Create an app**

### Step 2: Configure Basic App Settings

- App Name: Choose any name  
- App Developer: Your name or business  
- Click **Apply**

### Step 3: Set Up Admin API Scopes

Enable these:

```
✓ read_order_edits  
✓ write_orders  
✓ read_orders  
✓ read_products  
✓ read_price_rules  
✓ read_discounts  
✓ read_all_cart_transforms  
✓ write_cart_transforms  
✓ read_cart_transforms
```

### Step 4: Configure Storefront API Scopes

Enable these:

```
✓ unauthenticated_read_product_listings  
✓ unauthenticated_read_product_inventory  
✓ unauthenticated_read_product_pickup_locations  
✓ unauthenticated_read_product_tags  
✓ unauthenticated_write_checkouts  
✓ unauthenticated_read_checkouts  
✓ unauthenticated_read_content  
✓ unauthenticated_write_customers  
✓ unauthenticated_read_customers  
✓ unauthenticated_read_customer_tags  
✓ unauthenticated_read_metaobjects  
✓ unauthenticated_read_selling_plans  
✓ unauthenticated_read_bundles  
✓ unauthenticated_write_bulk_operations  
✓ unauthenticated_read_bulk_operations  
✓ unauthenticated_write_gates  
✓ unauthenticated_read_gates  
✓ unauthenticated_read_shop_pay_installments_pricing
```

### Step 5: Install and Save Access Tokens

- Click **Install app**
- Confirm installation
- Copy and save both:
  - Storefront API token
  - Admin API token

---

## 🌍 Chapter 2: Adding a Custom Checkout Domain (NGINX Setup)

If you're self-hosting the frontend (e.g., via VPS), follow these steps.

### Step 1: Create DNS Record

Point `checkout.clientstore.com` to your server IP.

Example:

```
Type: A  
Host: checkout.clientstore.com  
Value: 134.209.199.211
```

### Step 2: Create NGINX Config

```bash
sudo nano /etc/nginx/sites-available/checkout-frontend-clientstore.com
```

Paste config:

```nginx
server {
    listen 80;
    server_name checkout.clientstore.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;

        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization' always;
        add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range' always;

        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
        }
    }
}
```

### Step 3: Enable Config

```bash
sudo ln -sf /etc/nginx/sites-available/checkout-frontend-clientstore.com /etc/nginx/sites-enabled/
```

### Step 4: Test NGINX

```bash
nginx -t
```

Output should confirm: `syntax is ok` and `test is successful`

### Step 5: Generate SSL Certificate

```bash
sudo certbot --nginx -d checkout.clientstore.com
```

### Step 6: Restart NGINX

```bash
sudo systemctl restart nginx
```

---

## 🔧 Chapter 3: Registering Your Shop in the Checkout Backend

### API Configuration

Send a `POST` request to:

```
https://api.guillermo.click/api/shops/
```

### Example JSON Body:

Make sure `frontend_url` matches the subdomain from Chapter 2

```json
{
  "shop_domain": "clientstore.myshopify.com",
  "shop_name": "Client Store",
  "shopify_store_url": "https://clientstore.myshopify.com",
  "shopify_access_token": "{ADMIN_API_TOKEN}",
  "shopify_storefront_token": "{STOREFRONT_API_TOKEN}",
  "frontend_url": "https://checkout.clientstore.com",
  "currency": "EUR",
  "language_code": "en",
  "logo_url": "https://cdn.clientstore.com/logo.png",
  "is_active": true,
  "payment_systems": {
    "curo": {
      "enabled": true,
      "config": {
        "api_key": "{CURO_API_KEY}",
        "merchant_id": "{CURO_MERCHANT_ID}",
        "site_id": "{CURO_SITE_ID}",
        "enabled_methods": {
          "bancontact": true,
          "creditcard": false,
          "ideal": true,
          "klarna": false,
          "riverty": false,
          "paypal": true
        }
      }
    }
  },
  "colors": {
    "primary": "oklch(0.205 0 0)",
    "background": "oklch(1 0 0)",
    "foreground": "oklch(0.145 0 0)"
  }
}
```

### Verification

Ensure the response confirms the store is registered correctly.

---

## 🧩 Chapter 4: Implementing the Custom Checkout Button

### Step 1: Open Shopify Theme Editor

Navigate to `Sales Channels → Online Store` and click **Customize**

### Step 2: Edit Theme Code

Click the ⋯ menu → "Edit code"

### Step 3: Replace Checkout Button

Insert this snippet where the Shopify checkout button is rendered:

```html
<a href="#" id="external-checkout" class="cart__checkout-button button">Checkout</a>
```

Add the full `<script>` block to handle `POST /checkout/init` and redirect logic (provided earlier).

---

## ✅ Testing the Integration

1. Add products to cart
2. Click the custom checkout button
3. Confirm redirection to your custom domain
4. Complete test checkout flow (e.g., Curo sandbox, mock webhook)
5. Reach "Thank You" page
