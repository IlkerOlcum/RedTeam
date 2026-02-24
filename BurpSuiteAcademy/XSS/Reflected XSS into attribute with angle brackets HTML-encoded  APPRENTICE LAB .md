
# Reflected XSS into attribute with angle brackets HTML-encoded

## 🎯 Summary
This lab demonstrates a reflected Cross-Site Scripting (XSS) vulnerability where user input is injected into an HTML attribute. Although angle brackets (`<` and `>`) are HTML-encoded, the application fails to properly encode quotation marks, allowing attribute injection and JavaScript execution via an event handler.

---

## 🔎 Vulnerability Type
Reflected XSS (Attribute Context)

---

## 📍 Injection Context

User input is reflected inside a quoted HTML attribute:

```html
<input type="text" value="USER_INPUT">
```

Angle brackets are encoded as:

```
&lt; and &gt;
```

However, double quotes (`"`) are not encoded.

---

## ❌ Why `<script>` Injection Fails

Attempting a classic payload:

```html
<script>alert(1)</script>
```

Results in:

```html
value="&lt;script&gt;alert(1)&lt;/script&gt;"
```

Since angle brackets are encoded, the script is rendered as plain text and does not execute.

---

## 💣 Payload Used (Attribute Injection)

```html
" onmouseover="alert(1)
```

---

## 🛠 Exploitation Steps

1. Submitted a random string in the search field.
2. Observed via Burp Repeater that the input is reflected inside a quoted attribute.
3. Confirmed that angle brackets are encoded but quotation marks are not.
4. Injected a double quote (`"`) to break out of the attribute value.
5. Added a malicious event handler (`onmouseover`) to execute JavaScript.
6. Triggered the event by hovering over the injected element.
7. The `alert(1)` function executed successfully.

---

## 🧠 Why the Attack Works

- User input is reflected inside a quoted attribute.
- Angle brackets are encoded, preventing tag injection.
- Double quotes are not encoded.
- Breaking out of the attribute allows injection of a new attribute.
- The injected event handler executes JavaScript when triggered.

---

## 🛡 Mitigation

- Properly encode all user-controlled input.
- Encode quotation marks (`"` and `'`) in attribute context.
- Use context-aware output encoding.
- Implement a strict Content Security Policy (CSP).

---

Lab successfully solved.
