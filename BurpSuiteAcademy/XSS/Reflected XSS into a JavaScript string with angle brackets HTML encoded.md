# Reflected XSS into a JavaScript string with angle brackets HTML-encoded

## 🎯 Summary
This lab demonstrates a reflected Cross-Site Scripting (XSS) vulnerability where user input is reflected inside a JavaScript string. Although angle brackets (`<` and `>`) are HTML-encoded, the application fails to properly escape characters within the JavaScript context, allowing an attacker to break out of the string and execute arbitrary JavaScript.

---

## 🔎 Vulnerability Type
Reflected XSS (JavaScript Context Injection)

---

## 📍 Injection Context

User input is reflected inside a JavaScript string:

```html
<script>
var searchTerms = 'USER_INPUT';
</script>
```

Angle brackets are encoded, preventing direct `<script>` injection. However, the string delimiter (`'`) is not properly escaped.

---

## ❌ Why `<script>` Injection Fails

Attempting:

```html
<script>alert(1)</script>
```

Results in encoded output:

```
&lt;script&gt;alert(1)&lt;/script&gt;
```

Since the payload is inside a JavaScript string, tag injection is ineffective.

---

## 💣 Payload Used (String Breakout)

```javascript
'-alert(1)-'
```

---

## 🛠 Exploitation Steps

1. Submitted a random string in the search field.
2. Intercepted the request using Burp Suite and analyzed the response in Repeater.
3. Observed that the input is reflected inside a single-quoted JavaScript string.
4. Confirmed that angle brackets are encoded but single quotes are not escaped.
5. Injected a payload that:
   - Closes the existing string (`'`)
   - Executes `alert(1)`
   - Reopens the string to prevent syntax errors
6. Reloaded the page and confirmed that the alert dialog was triggered.

---

## 🧠 Why the Attack Works

- User input is embedded inside a JavaScript string.
- Angle brackets are encoded, but string delimiters are not escaped.
- Breaking out of the string allows execution of arbitrary JavaScript.
- The trailing characters restore valid syntax to avoid script errors.

---

## 🛡 Mitigation

- Properly escape user input before inserting into JavaScript.
- Use context-aware encoding for JavaScript strings.
- Avoid embedding raw user input directly in inline scripts.
- Implement a strict Content Security Policy (CSP).

---

Lab successfully solved.
