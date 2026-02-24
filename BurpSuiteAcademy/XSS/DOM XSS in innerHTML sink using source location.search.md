# DOM XSS in innerHTML sink using source location.search

## 🎯 Summary
This lab demonstrates a DOM-based Cross-Site Scripting (XSS) vulnerability caused by unsafe usage of `innerHTML` with user-controlled input from `location.search`.

The application extracts the `search` parameter from the URL and dynamically inserts it into the DOM using an `innerHTML` assignment. Because `innerHTML` parses and renders HTML content, any injected HTML or JavaScript is executed by the browser.

---

## 🔎 Vulnerability Type
DOM-Based XSS

## 📍 Source
```javascript
window.location.search
```

The `search` parameter is fully controllable via the URL.

---

## 📍 Sink
```javascript
element.innerHTML = ...
```

The application writes user input directly into the DOM using `innerHTML`, which interprets the content as HTML instead of plain text.

---

## 💣 Payload Used
```html
<img src=1 onerror=alert(1)>
```

---

## 🛠 Exploitation Steps
1. Identified that the search parameter is read from `location.search`.
2. Observed that the value is inserted into the page using `innerHTML`.
3. Confirmed that no sanitization or encoding is applied.
4. Injected a malicious `<img>` tag with an invalid `src` attribute.
5. The invalid `src` triggers the `onerror` event handler.
6. The `onerror` event executes `alert(1)`, confirming XSS.

---

## 🧠 Why the Attack Works
- `innerHTML` parses input as HTML.
- The `<img>` tag is rendered by the browser.
- Since `src=1` is invalid, it triggers an error.
- The `onerror` attribute executes JavaScript.
- No output encoding or input sanitization is implemented.

---

## 🛡 Mitigation
- Avoid using `innerHTML` with user-controlled input.
- Use `textContent` or `innerText` instead.
- Apply proper output encoding.
- Implement a strict Content Security Policy (CSP).

---

Lab successfully solved.
