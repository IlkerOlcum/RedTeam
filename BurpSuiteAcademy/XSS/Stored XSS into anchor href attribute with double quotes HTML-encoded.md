# Stored XSS into anchor href attribute with double quotes HTML-encoded

## 🎯 Summary
This lab demonstrates a Stored Cross-Site Scripting (XSS) vulnerability in the comment functionality. User-supplied input from the "Website" field is stored and later rendered inside the `href` attribute of an anchor (`<a>`) tag. Although double quotes are HTML-encoded, the application fails to validate dangerous URI schemes, allowing JavaScript execution via a `javascript:` URL.

Because the payload is stored server-side and rendered to all users viewing the post, this vulnerability represents a stored (persistent) XSS.

---

## 🔎 Vulnerability Type
Stored XSS (Anchor `href` Attribute Injection)

---

## 📍 Injection Context

The "Website" field is reflected in the page as:

```html
<a href="USER_INPUT">Author Name</a>
```

Double quotes are encoded, but the value itself is not validated.

---

## ❌ Why Traditional Breakout Is Not Needed

Since the input is directly placed inside the `href` attribute value, there is no need to break out of quotes. Instead, the attack leverages a dangerous URL scheme.

---

## 💣 Payload Used

```
javascript:alert(1)
```

---

## 🛠 Exploitation Steps

1. Posted a comment with a random value in the "Website" field.
2. Intercepted the request using Burp Suite and confirmed the value is stored.
3. Viewed the post and intercepted the response.
4. Observed that the input is rendered inside an anchor `href` attribute.
5. Repeated the process, replacing the Website value with:
   ```
   javascript:alert(1)
   ```
6. Reloaded the page.
7. Clicked on the comment author’s name.
8. The browser executed `alert(1)`, confirming successful XSS.

---

## 🧠 Why the Attack Works

- The application stores user input without validation.
- The value is rendered inside an anchor’s `href` attribute.
- The `javascript:` URI scheme is not filtered.
- When the link is clicked, the browser executes the JavaScript code.
- Since the payload is stored, it affects any user who views the page.

---

## ⚠️ Impact

- Persistent client-side code execution
- Session hijacking via `document.cookie`
- Phishing or redirection attacks
- Account compromise

---

## 🛡 Mitigation

- Validate and restrict allowed URL schemes (only `http` and `https`)
- Reject `javascript:` and other dangerous protocols
- Apply proper output encoding
- Implement Content Security Policy (CSP)
- Sanitize stored user input

---

Lab successfully solved.
