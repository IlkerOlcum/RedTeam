# DOM XSS in document.write sink using source location.search

## 🎯 Summary
This lab demonstrates a DOM-based Cross-Site Scripting (XSS) vulnerability caused by unsafe usage of `document.write()` with user-controlled data from `location.search`.

The application extracts the `search` parameter from the URL and dynamically injects it into the DOM inside an HTML attribute without proper sanitization or encoding. Because the value is written directly into the `src` attribute of an `<img>` tag, it is possible to break out of the attribute context and inject arbitrary JavaScript.

---

## 🔎 Vulnerability Type
DOM-Based XSS

## 📍 Source
```javascript
window.location.search
```

## 📍 Sink
```javascript
document.write()
```

## 📍 Vulnerable Code
```javascript
document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
```

User input is embedded inside a double-quoted `src` attribute.

---

## 💣 Payload Used
```html
"><script>alert(1)</script>
```

---

## 🛠 Exploitation Steps
1. Identified that the `search` parameter is read using `URLSearchParams`.
2. Observed that the value is written to the DOM using `document.write()`.
3. Confirmed the input is injected inside a double-quoted attribute.
4. Broke out of the attribute context using `">`.
5. Injected a `<script>` tag to execute JavaScript.

---

## 🧠 Root Cause
- User-controlled data flows directly from `location.search`
- Unsafe DOM sink (`document.write`) is used
- No output encoding or sanitization is applied
- Attribute context breakout allows script injection

---

## 🛡 Mitigation
- Avoid using `document.write`
- Use safe DOM manipulation methods (e.g., `textContent`, `setAttribute`)
- Apply proper output encoding
- Implement Content Security Policy (CSP)

---

Lab successfully solved.
