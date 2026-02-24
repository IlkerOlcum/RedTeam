# DOM XSS in jQuery anchor href attribute sink using location.search source

## 🎯 Summary
This lab demonstrates a DOM-based Cross-Site Scripting (XSS) vulnerability caused by unsafe manipulation of an anchor (`<a>`) element’s `href` attribute using user-controlled input from `location.search`.

The application uses jQuery’s `$()` selector to locate a link element and dynamically assigns its `href` attribute based on a URL parameter (`returnPath`). Because no validation is performed, an attacker can inject a `javascript:` URI scheme and execute arbitrary JavaScript when the link is clicked.

---

## 🔎 Vulnerability Type
DOM-Based XSS

---

## 📍 Source
```javascript
location.search
```

The `returnPath` parameter is fully controllable via the URL.

Example:
```
?returnPath=/home
```

---

## 📍 Sink
```javascript
$('a#back').attr('href', returnPath);
```

The application uses jQuery to directly assign the `href` attribute of the "Back" link without sanitization.

---

## 💣 Payload Used
```
javascript:alert(document.cookie)
```

---

## 🛠 Exploitation Steps
1. Navigated to the "Submit feedback" page.
2. Observed that the `returnPath` parameter controls the `href` attribute of the "Back" link.
3. Confirmed via browser inspection that the parameter value is inserted directly into the anchor’s `href`.
4. Replaced the parameter value with a `javascript:` URI:
   ```
   ?returnPath=javascript:alert(document.cookie)
   ```
5. Pressed Enter to reload the page.
6. Clicked the "Back" link.
7. The browser executed the JavaScript code and displayed the cookie value.

---

## 🧠 Why the Attack Works
- The application trusts user-controlled URL parameters.
- jQuery’s `.attr()` method directly modifies the `href` attribute.
- No validation is performed to restrict dangerous URI schemes.
- The browser executes `javascript:` URIs when the link is clicked.

---

## 🛡 Mitigation
- Validate and restrict allowed URL schemes (allow only `http` and `https`).
- Reject `javascript:` and other dangerous protocols.
- Avoid directly assigning user input to sensitive DOM attributes.
- Implement a strict Content Security Policy (CSP).

---

Lab successfully solved.
