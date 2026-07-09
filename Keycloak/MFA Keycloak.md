## Enforce TOTP MFA in Keycloak

1. Go to `Authentication` > `Policies` > `OTP Policy`.

![OTP Policy](images/Pasted%20image%2020260701054307.png)

2. Set the number of digits to 6, hash algorithm to SHA1, token period to 30 seconds, and type to Time Based. These defaults work with Google Authenticator, Authy, and similar apps.

![OTP settings](images/Pasted%20image%2020260701055643.png)

3. Set a password for the account.

![Set password](images/Pasted%20image%2020260701055710.png)

4. Log in.

![Login](images/Pasted%20image%2020260701060529.png)

5. Keycloak prompts to scan the QR code to enroll TOTP.

![OTP enrollment prompt](images/redacted_qr_screenshot.png)
