
# DOM XSS in jQuery selector sink using a hashchange event

## 🎯 Summary
This lab demonstrates a DOM-based Cross-Site Scripting (XSS) vulnerability caused by unsafe usage of jQuery’s `$()` selector with data taken from `location.hash`.

The application listens for a `hashchange` event and uses the value of `location.hash` to dynamically select an element on the page. Because the hash value is user-controlled and not sanitized, it is possible to inject malicious HTML that gets interpreted by jQuery, leading to JavaScript execution.

---

## 🔎 Vulnerability Type
DOM-Based XSS

---

## 📍 Source
```javascript
location.hash
```

The fragment identifier (the part after `#` in the URL) is fully controllable by the attacker.

Example:
```
https://vulnerable-site/#post1
```

---

## 📍 Sink
```javascript
$(location.hash)
```

The application passes user-controlled input directly into jQuery’s `$()` selector, which interprets certain inputs as HTML instead of a safe selector.

---

## 🧠 Root Cause
- The application listens to the `hashchange` event.
- The value of `location.hash` is passed directly to `$()`.
- jQuery interprets strings starting with `<` as HTML.
- No sanitization or validation is performed.
- Malicious HTML is injected and executed in the DOM.

---

## 💣 Exploit Payload

Malicious iframe hosted on the exploit server:

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#"
onload="this.src+='<img src=x onerror=print()>'">
</iframe>
```

---

## 🛠 Exploitation Steps
1. Inspected the homepage JavaScript and identified usage of `location.hash` inside jQuery’s `$()` selector.
2. Confirmed that the page reacts to the `hashchange` event.
3. Opened the provided exploit server.
4. Created a malicious iframe that:
   - Loads the vulnerable page.
   - Appends a malicious HTML payload to the hash.
5. Injected:
   ```html
   <img src=x onerror=print()>
   ```
6. The invalid `src` triggers the `onerror` handler.
7. The `print()` function is executed in the victim’s browser.
8. Delivered the exploit to the victim to solve the lab.

---

## 🧠 Why the Attack Works
- `location.hash` is attacker-controlled.
- jQuery `$()` treats HTML-like input as DOM elements.
- No input validation is applied.
- The browser executes the injected `onerror` event handler.

---

## 🛡 Mitigation
- Never pass user-controlled data directly into jQuery selectors.
- Sanitize and validate `location.hash`.
- Use safe selector patterns.
- Implement a strict Content Security Policy (CSP).

---

Lab successfully solved.
