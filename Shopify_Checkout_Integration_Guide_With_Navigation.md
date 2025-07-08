
# 🛠️ Shopify Custom Checkout Integration Guide

## 📑 Table of Contents

- [Pre-Conditions](#pre-conditions)
- [Chapter 1: Creating Your Shopify App](#chapter-1-creating-your-shopify-app)
  - [Step 1: Navigate to App Development](#step-1-navigate-to-app-development)
  - [Step 2: Configure Basic App Settings](#step-2-configure-basic-app-settings)
  - [Step 3: Set Up Admin API Scopes](#step-3-set-up-admin-api-scopes)
  - [Step 4: Configure Storefront API Scopes](#step-4-configure-storefront-api-scopes)
  - [Step 5: Install and Save Access Tokens](#step-5-install-and-save-access-tokens)
- [Chapter 2: Adding a Custom Checkout Domain (NGINX Setup)](#chapter-2-adding-a-custom-checkout-domain-nginx-setup)
  - [Step 1: Create DNS Record](#step-1-create-dns-record)
  - [Step 2: Create NGINX Config](#step-2-create-nginx-config)
  - [Step 3: Enable Config](#step-3-enable-config)
  - [Step 4: Test NGINX](#step-4-test-nginx)
  - [Step 5: Generate SSL Certificate](#step-5-generate-ssl-certificate)
  - [Step 6: Restart NGINX](#step-6-restart-nginx)
- [Chapter 3: Registering Your Shop in the Checkout Backend](#chapter-3-registering-your-shop-in-the-checkout-backend)
  - [API Configuration](#api-configuration)
  - [Verification](#verification)
- [Chapter 4: Implementing the Custom Checkout Button](#chapter-4-implementing-the-custom-checkout-button)
  - [Step 1: Access Theme Editor](#step-1-access-theme-editor)
  - [Step 2: Edit Theme Code](#step-2-edit-theme-code)
  - [Step 3: Locate and Replace Checkout Button](#step-3-locate-and-replace-checkout-button)
- [Testing the Integration](#testing-the-integration)

---

## Pre-Conditions

Before starting, make sure you have:

- Admin access to your Shopify store
- Access to the checkout backend API
- A Linux server (with NGINX) or frontend deployment on Vercel/Netlify
- DNS access to create subdomains (e.g., `checkout.yourstore.com`)
- Basic understanding of Shopify themes and Liquid templating

---

## Chapter 1: Creating Your Shopify App

### Step 1: Navigate to App Development
Go to `Settings → Apps and sales channels → Develop apps` in your Shopify admin. Click **Create an App**.

### Step 2: Configure Basic App Settings
Name your app and provide developer information. Click **Apply**.

### Step 3: Set Up Admin API Scopes
Enable the following scopes:
- read_order_edits
- write_orders
- read_orders
- read_products
- read_price_rules
- read_discounts
- read_all_cart_transforms
- write_cart_transforms
- read_cart_transforms

### Step 4: Configure Storefront API Scopes
Enable:
- unauthenticated_read_product_listings
- unauthenticated_read_product_inventory
- unauthenticated_read_product_pickup_locations
- unauthenticated_read_product_tags
- unauthenticated_write_checkouts
- unauthenticated_read_checkouts
- unauthenticated_read_content
- unauthenticated_write_customers
- unauthenticated_read_customers
- unauthenticated_read_customer_tags
- unauthenticated_read_metaobjects
- unauthenticated_read_selling_plans
- unauthenticated_read_bundles
- unauthenticated_write_bulk_operations
- unauthenticated_read_bulk_operations
- unauthenticated_write_gates
- unauthenticated_read_gates
- unauthenticated_read_shop_pay_installments_pricing

### Step 5: Install and Save Access Tokens
Click **Install App**. Copy and save the Admin and Storefront tokens.

---

## Chapter 2: Adding a Custom Checkout Domain (NGINX Setup)

### Step 1: Create DNS Record
Add an A-record for your subdomain pointing to the server’s IP.

### Step 2: Create NGINX Config
Create file in `/etc/nginx/sites-available/checkout-frontend-clientstore.com` and insert the provided configuration block.

### Step 3: Enable Config
Create a symlink to `sites-enabled`.

```bash
sudo ln -sf /etc/nginx/sites-available/checkout-frontend-clientstore.com /etc/nginx/sites-enabled/
```

### Step 4: Test NGINX
Run:

```bash
nginx -t
```

### Step 5: Generate SSL Certificate
Use Certbot:

```bash
sudo certbot --nginx -d checkout.clientstore.com
```

### Step 6: Restart NGINX
```bash
sudo systemctl restart nginx
```

---

## Chapter 3: Registering Your Shop in the Checkout Backend

### API Configuration

Send `POST` request to:

```http
https://api.guillermo.click/api/shops/
```

With a valid JSON body containing your store credentials, branding, and payment method configuration.

### Verification

Ensure the response confirms successful creation and your domain matches what’s configured on frontend.

---

## Chapter 4: Implementing the Custom Checkout Button

### Step 1: Access Theme Editor
Go to `Sales Channels → Online Store → Customize` in Shopify admin.

### Step 2: Edit Theme Code
Click the three dots ⋯ → **Edit code**.

### Step 3: Locate and Replace Checkout Button
Replace default checkout button with the provided `<a>` element and add JavaScript code that triggers the `/checkout/init` API.

---

## Testing the Integration

1. Add products to cart
2. Go to your cart page
3. Click the custom checkout button
4. Verify redirect to custom domain
5. Complete checkout form and simulate payment
6. Confirm redirection to “Thank You” page
