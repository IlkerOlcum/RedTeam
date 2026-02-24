## 🎯 Summary
This lab demonstrates a basic Reflected Cross-Site Scripting (XSS) vulnerability where user input is directly embedded into the HTML response without any encoding or sanitization. Because the input is reflected inside the HTML body context, it is possible to inject and execute arbitrary JavaScript.

The vulnerability was exploited by injecting a simple `<script>` payload that successfully executed in the victim's browser.

---

## 🔎 Vulnerability Type
Reflected XSS

## 📍 Context
HTML body context (no output encoding)

## 💣 Payload Used
```html
<script>alert(1)</script>
```

## 🛠 Exploitation Steps
1. Identified that the search parameter reflects user input.
2. Tested with a basic XSS payload.
3. Observed that the payload was rendered without encoding.
4. JavaScript executed successfully.

## 🧠 Root Cause
The application fails to:
- Encode user-controlled input
- Sanitize special HTML characters
- Apply output encoding before rendering data

## 🛡 Mitigation
- Implement proper output encoding
- Validate and sanitize user input
- Use Content Security Policy (CSP)
