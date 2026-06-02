
# OTP Flooding via CAPTCHA Bypass in a Government Web App

## Summary

Business logic bug. The CAPTCHA only protected the first page, so I could replay the second request and send unlimited OTPs to any phone number.

## Target

- Type: Large Indian government web application
- Assessment context: authorized engagement
- Disclosure status: reported and fixed

## Discovery

I was just going through the registration flow and noticed the CAPTCHA was only on the first page. The request that sends the OTP was on the second page and had nothing protecting it. Caught it in Burp and saw I could just send it on its own.

## Technical Details

The flow had 3 pages:

1. Page 1: pick a field (location/address) and solve a CAPTCHA.
2. Page 2: enter name, email, phone number and submit. This is what sends the OTP.
3. Page 3: enter the OTP.

They put the CAPTCHA there thinking only a real user who passed it could reach page 2. But the page 2 request works on its own. So I caught it once in Burp and replayed it through Intruder as many times as I wanted — no CAPTCHA needed.

The request looked like this:

​```http
POST /redacted HTTP/1.1
Host: redacted
Cookie: 7bbc95523096a3350=[REDACTED]; TS092aa=[REDACTED]
Content-Length: XYZ

{
    "mobileNo": "[REDACTED]",
    "Person name": "[REDACTED]",
    "Address": ""
}
​```

In Intruder I ran the `mobileNo` field through all Indian numbers (6666666666 to 9999999999). Every request sent an OTP SMS to that number. The OTP on page 3 was also brute forceable, which made it worse.

## Impact

- Costs them a lot of SMS money since every request sends a real SMS.
- People get spammed with OTPs they never asked for, and real users can't register when the flow is flooded.
- The OTP itself was brute forceable, so an attacker could also try to guess it and create accounts.

## Remediation

1. Tie the CAPTCHA (or a one-time server token) to the OTP request itself, not just the first page.
2. Add rate limiting on the OTP endpoint — per IP, per session, and per number.
3. Add a cooldown and a cap on how many OTPs a number can get in a time window.
4. Make the OTP harder to guess — longer, short expiry, lock after a few wrong tries.


## Takeaway

A CAPTCHA only protects the page it sits on, not the request behind it. If a flow assumes "you can only reach step 2 after step 1," check if step 2 can just be called directly. That gap is where these bugs hide.
