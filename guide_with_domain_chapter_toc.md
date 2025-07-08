# Shopify Custom Checkout Integration Guide


## Table of Contents

1. [Prerequisites](#pre-conditions)
2. [Chapter 1: Creating Your Shopify App](#chapter-1-creating-your-shopify-app)
   - [Step 1: Navigate to App Development](#step-1-navigate-to-app-development)
   - [Step 2: Configure Basic App Settings](#step-2-configure-basic-app-settings)
   - [Step 3: Set Up Admin API Scopes](#step-3-set-up-admin-api-scopes)
   - [Step 4: Configure Storefront API Scopes](#step-4-configure-storefront-api-scopes)
   - [Step 5: Install and Save Access Tokens](#step-5-install-and-save-access-tokens)
3. [Chapter 2: Adding a Custom Checkout Domain (NGINX Setup)](#chapter-2-adding-a-custom-checkout-domain-nginx-setup)
   - [Step 1: Create DNS Record](#step-1-create-dns-record)
   - [Step 2: Create NGINX Config](#step-2-create-nginx-config)
   - [Step 3: Enable Config](#step-3-enable-config)
   - [Step 4: Test NGINX](#step-4-test-nginx)
   - [Step 5: Generate SSL Certificate](#step-5-generate-ssl-certificate)
   - [Step 6: Restart NGINX](#step-6-restart-nginx)
4. [Chapter 3: Registering Your Shop in the Checkout Backend](#chapter-3-registering-your-shop-in-the-checkout-backend)
   - [API Configuration](#api-configuration)
   - [Verification](#verification)
5. [Chapter 3: Implementing the Custom Checkout Button](#chapter-3-implementing-the-custom-checkout-button)
   - [Step 1: Access Theme Editor](#step-1-access-theme-editor)
   - [Step 2: Edit Theme Code](#step-2-edit-theme-code)
   - [Step 3: Locate and Replace Checkout Button](#step-3-locate-and-replace-checkout-button)
6. [Testing the Integration](#testing-the-integration)
   - [Flow](#flow)


## Pre-Conditions

Before starting, verify that you have:
1. Admin access to your Shopify store
2. Access to the checkout backend API
3. Basic understanding of Shopify themes and Liquid templating

---

## Chapter 1: Creating Your Shopify App

### Step 1: Navigate to App Development
1. In your Shopify admin panel, go to **Settings** -> **Apps and sales channels** -> **Develop apps**
2. Click the **Create an APP** button

### Step 2: Configure Basic App Settings
1. Fill in the required information:
   - **App Name**: Enter your desired app name
   - **App Developer**: Specify the developer information
2. Click **"Apply"** to save the basic settings

### Step 3: Set Up Admin API Scopes
Configure the following Admin API scopes for your app:

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
Add these Storefront API scopes:

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
1. Click the **"Install app"** button
2. Confirm installation by clicking **Install**
3. **Important**: Copy and securely save both tokens:
   - **Storefront API access token**
   - **Admin API access token**

> **Note**: Store these tokens securely as they provide access to your store's data.

---


## Chapter 2: Adding a Custom Checkout Domain (NGINX Setup)

### Step 1: Create DNS Record

Create an A-record in your domain's DNS settings to point the desired subdomain (e.g. `checkout.clientstore.com`) to the server IP address.

Example:
```
Type: A
Host: checkout.clientstore.com
Value: 134.209.199.211
```

### Step 2: Create NGINX Configuration

Create a new file in `/etc/nginx/sites-available/`:

```bash
sudo nano /etc/nginx/sites-available/checkout-frontend-clientstore.com
```

Paste the following configuration into the file:

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

### Step 3: Enable Configuration

Create a symlink to activate the site:

```bash
sudo ln -sf /etc/nginx/sites-available/checkout-frontend-clientstore.com /etc/nginx/sites-enabled/
```

### Step 4: Test NGINX Configuration

Run:

```bash
sudo nginx -t
```

Expected output:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Step 5: Generate SSL Certificate

Run the following command to generate and configure SSL using Certbot:

```bash
sudo certbot --nginx -d checkout.clientstore.com
```

### Step 6: Restart NGINX

```bash
sudo systemctl restart nginx
```


## Chapter 3: Registering Your Shop in the Checkout Backend

### API Configuration
Make a POST request to create your shop configuration:

**Endpoint**: `https://api.guillermo.click/api/shops/`

**Request Body**:
```json
{
    "colors":
    {
        "accent": "oklch(0.97 0 0)",
        "background": "oklch(1 0 0)",
        "border": "oklch(0.922 0 0)",
        "card": "oklch(1 0 0)",
        "destructive": "oklch(0.577 0.245 27.325)",
        "foreground": "oklch(0.145 0 0)",
        "input": "oklch(0.922 0 0)",
        "muted": "oklch(0.97 0 0)",
        "mutedForeground": "oklch(0.556 0 0)",
        "popover": "oklch(1 0 0)",
        "popoverForeground": "oklch(0.145 0 0)",
        "primary": "oklch(0.205 0 0)",
        "ring": "oklch(0.708 0 0)",
        "secondary": "oklch(0.97 0 0)",
        "secondaryForeground": "oklch(0.205 0 0)"
    },
    "currency": "EUR",
    "is_active": true,
    "language_code": "en",
    "logo_url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/d3/Vienna_Convention_road_sign_Aa-15b-V1.svg/1920px-Vienna_Convention_road_sign_Aa-15b-V1.svg.png",
    "payment_systems":
    {
        "curo":
        {
            "config":
            {
                "api_key": "{CURO_API_KEY}",
                "enabled_methods":
                {
                    "bancontact": true,
                    "creditcard": false,
                    "ideal": true,
                    "klarna": false,
                    "riverty": false,
                    "paypal": true
                },
                "merchant_id": "{CURO_MERCHANT_ID}",
                "site_id": "{CURO_SITE_ID"
            },
            "enabled": true
        }
    },
    "shop_domain": "sti4g0-wr.myshopify.com",
    "shop_name": "Test Guillermo Shop",
    "shopify_access_token": "{ADMIN_API_TOKEN}",
    "shopify_store_url": "https://sti4g0-wr.myshopify.com",
    "frontend_url": "https://checkouts.modelumiere.com",
    "shopify_storefront_token": "{STOREFRONT_API_TOKEN}"
}
```

### Verification
After making the request, verify that response contains the correct shop configuration data.

---

## Chapter 4: Implementing the Custom Checkout Button

### Step 1: Access Theme Editor
1. Navigate to **Sales Channels** → **Online Store** in your Shopify admin
2. Click the **"Customize"** button for your active theme

### Step 2: Edit Theme Code
1. Click the **three dots (...)** menu at the top of the theme editor
2. Select **"Edit code"**

### Step 3: Locate and Replace Checkout Button
Find your existing checkout button implementation and replace it with the following code (should be the same for all shops):

```html
<a href="#" id="external-checkout" class="cart__checkout-button button">
  Checkout
</a>

<script>
  document.addEventListener("DOMContentLoaded", function () {
    const checkoutButton = document.getElementById("external-checkout");

    checkoutButton.addEventListener("click", async function (e) {
      e.preventDefault();

      try {
        // Fetch current cart data
        const cart = await fetch('/cart.js').then(r => r.json());

        // Initialize custom checkout
        const response = await fetch("https://api.guillermo.click/api/checkout/init", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            shop: "{{ shop.permanent_domain }}",
            cart: cart,
            origin: window.location.origin
          }),
          redirect: 'follow'
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        // Handle redirect if checkout initialization was successful
        if (response.redirected) {
          console.log('Redirecting to checkout:', response.url);
          window.location.href = response.url;
          return;
        }

      } catch (err) {
        console.error("Checkout initialization failed:", err);
        alert("Error during checkout: " + err.message);
      }
    });
  });
</script>
```
---

## Testing the integration

### Flow
1. **Add products to cart**: Navigate to your store and add some products to the shopping cart
2. **Open cart page**: Go to your cart page where the checkout button is located
3. **Test checkout flow**: Click the **"Checkout"** button
4. **Verify redirect**: You should be automatically redirected to the custom checkout page
5. (Optional) -> fill in the Discount Code if you have any
6. Fill in all the requested information, choose the required Payment System, press 'Pay with {payment_system}'

Since the system is now in the sandbox mode, there will be no actual charge performed, for Curo (iDeal, bancontact, klarna, etc), you should first send the webhook (callback) for Success (200) using the respective radiobutton and then press the 'Submit' button -> then you should be redirected to the 'thank you' page.
For PayPal - just log in to your account (could be sandbox as well), and proceed with the transaction, the redirect will be handled automatically.

Once the payment is processed - the customer's cart will be cleared and the new order on the shopify side will be created.

---
