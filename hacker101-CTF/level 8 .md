# 🐾 Hacker101 CTF — Level 8: Petshop Lab

**Platform:** Hacker101 (HackerOne)
**Level:** 8
**Flags Found:** 1 / ?
**Vulnerability Class:** Business Logic — Price Manipulation via Client-Side Parameter Tampering

---

## 🗺️ Table of Contents

- [Overview](#-overview)
- [Recon](#-recon)
- [Flag 1 — Price Manipulation via Cart Parameter](#-flag-1--price-manipulation-via-cart-parameter)
  - [Steps](#steps)
  - [Burp Suite Request Analysis](#burp-suite-request-analysis)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
- [Vulnerability Breakdown](#-vulnerability-breakdown)
- [Bug Hunter Mindset](#-bug-hunter-mindset)
- [Key Takeaways](#-key-takeaways)

---

## 📌 Overview

The **Petshop Lab** is a simulated e-commerce web application where users can browse pet-related items (e.g., kitten photos), add them to a cart, and proceed to checkout.

The vulnerability exploited here is a classic **business logic flaw** — the server trusts client-supplied price data embedded in the cart parameter, allowing an attacker to manipulate product prices directly from the HTTP request.

---

## 🔍 Recon

- Explored all visible elements on the web page
- Identified the shopping cart functionality
- Added an item (kitten photo) to the cart
- Proceeded to checkout and **intercepted the request using Burp Suite**

---

## 🚩 Flag 1 — Price Manipulation via Cart Parameter

### Steps

1. Navigate through the web application and **click on all visible objects/items**
2. Add the **kitten image** to the cart using the `Add to Cart` button
3. Click **Checkout**
4. In **Burp Suite**, intercept the POST request triggered by the checkout action
5. Send the intercepted request to **Repeater** for analysis
6. Examine the request body — locate the `cart` parameter at the bottom

---

### Burp Suite Request Analysis

After URL-decoding the `cart` parameter, the raw value looks like this:

```
cart = %5B%5B0%2C+%7B%22name%22%3A+%22Kitten%22%2C+%22desc%22%3A+%228%5C%22x10%5C%22+color+glossy+photograph+of+a+kitten.%22%2C+%22logo%22%3A+%22kitten.jpg%22%2C+%22price%22%3A+8.96%7D%5D%5D
```

**URL-decoded:**

```json
[[0, {"name": "Kitten", "desc": "8\"x10\" color glossy photograph of a kitten.", "logo": "kitten.jpg", "price": 8.96}]]
```

**Key observation:** The `price` field is fully controlled by the client. The server performs **no server-side price validation**.

---

### Exploitation

Modify the `price` value inside the cart parameter to an arbitrary number (e.g., `0`, `-1`, or `0.01`):

**Before:**
```json
"price": 8.96
```

**After (tampered):**
```json
"price": 0
```

Re-encode the modified JSON back to URL encoding and send via Burp Suite Repeater.

> 💡 **Tip:** You can use Burp's built-in encoder (`Ctrl+Shift+U` to URL encode) or CyberChef to re-encode.

---

### Flag

```
^FLAG^945c6605636a1840664d235513c540ab0b0778724bd1643987a887cae81acfaf$FLAG$
```

---

## 🧠 Vulnerability Breakdown

| Property | Detail |
|---|---|
| **Vulnerability Type** | Business Logic Flaw |
| **OWASP Category** | A04:2021 – Insecure Design |
| **Attack Vector** | Client-side parameter tampering via intercepted HTTP request |
| **Root Cause** | Server trusts client-supplied price data without server-side validation |
| **Impact** | Attacker can purchase items for any arbitrary price (including free or negative) |
| **Tool Used** | Burp Suite (Proxy + Repeater) |

---

## 🎯 Bug Hunter Mindset

When testing e-commerce or any transactional web application, always ask:

- **Where is the price coming from?** — Is it in the request body, query param, or a hidden form field?
- **Is price validated server-side?** — Try setting it to `0`, `-1`, or `0.01`
- **What other fields can be tampered?** — Quantity, discount codes, product IDs
- **Is the cart stored client-side or server-side?** — Client-side = higher risk of manipulation

### Checklist for Price Manipulation Testing

- [ ] Intercept checkout/order request with Burp Suite
- [ ] Identify price-related parameters in the request body
- [ ] Decode any URL-encoded or Base64-encoded payloads
- [ ] Modify price to `0`, negative value, or extremely small number
- [ ] Re-encode and resend via Repeater
- [ ] Observe server response — does it accept the modified price?

---

## 📝 Key Takeaways

- **Never trust client-supplied data** — prices, quantities, and discounts must always be validated and recalculated **server-side**
- Business logic vulnerabilities are often **not caught by automated scanners** — they require manual testing and creative thinking
- URL-encoded cart parameters are a common attack surface in older or poorly designed e-commerce apps
- This class of bug can have **critical financial impact** in real-world applications

---

