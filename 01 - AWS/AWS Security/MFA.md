> MFA (Multi-Factor Authentication)

MFA requires two or more authentication methods to verify an identity. MFA usually consists of:

1. **Something you know** - Something you know, such as a user name and password or pin number
2. **Something you have** - Something you have, such as a one-time passcode from a hardware device or mobile app
3. **Something you are** - Something you are, such as a fingerprint or face scanning technology

With a combination of this information, systems can provide a layered approach to account access. So even if the first method of authentication, like Bob’s password, is cracked by a malicious actor, the second method of authentication, such as a fingerprint, provides another level of security. This extra layer of security can help protect your most important accounts, which is why you should activate MFA on your AWS root user. 

There are plenty of MFA devices. AWS supports 3: **Virtual MFA (DUO)** - mobile app with short-lived codes, **Hardware TOTP** - hardware device that generates a one-time password, **FIDO security key** - physical USB that could be plugged into USB port.