## Lab Overview
This lab demonstrated a stored cross-site scripting vulnerability within a comment functionality. User input was stored and rendered in HTML context without proper output encoding.

## Vulnerability Type
Stored Cross-Site Scripting (XSS)

## Root Cause
User-controlled input was inserted directly into the HTML response without any sanitization or encoding. As a result, arbitrary HTML and JavaScript could be executed when the page was viewed.

## Exploitation Logic
1. Identify that the comment field stores user input.
2. Observe that the output is rendered in HTML context.
3. Confirm no output encoding is applied.
4. Inject a JavaScript payload to trigger code execution upon page load.

## Impact
- Arbitrary JavaScript execution
- Session hijacking
- Credential theft
- Admin account compromise (in real-world scenarios)

## Key Takeaways
- Always test stored input points for output encoding.
- HTML context without encoding = direct script injection.
- Stored XSS is more dangerous than reflected XSS due to persistence.
