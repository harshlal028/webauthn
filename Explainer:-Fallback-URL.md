# Explainer: Fallback URL Extension for Hybrid Transports

## Authors:
Harsh Lal \<harshlal@google.com\>

_Last updated: 26-Jan-2026_

## Summary

This proposal introduces a `fallbackUrl` extension for FIDO2/WebAuthn hybrid (cross-device) flows. It allows Relying Parties (RPs) to provide a backup URL that an authenticator device (for example, a phone) can open if no passkey is found after scanning a QR code. The mechanism reduces friction in passwordless adoption by preventing dead-end sign-in attempts.

## Background & Motivation

One significant failure reason is “Credential Not Found.” This occurs when the authenticator cannot find a passkey for the requested RP after the user scans the QR code. That failure introduces friction into the passkey sign-in experience and reduces overall sign-in success rate for relying parties.

- **The problem:** Some metrics show that a substantial fraction (for example, ~32%) of authenticator failures are due to missing credentials on the device.

- **Current UX:** When a user scans a QR code but has no passkey on that phone, the flow typically fails with a generic “passkey not found” message, leaving the user without a clear recovery path.

- **Goal:** Eliminate the dead end. By providing a `fallbackUrl`, the authenticator can open a web login page or an app deep link on the phone, allowing the user to authenticate via alternative methods and complete the original sign-in attempt.


## Proposed Solution

The solution defines a new client extension for WebAuthn/CTAP that carries a `fallbackUrl` string.


### How It Works (The Flow)

1. **Initiation:** The Client (for example, a Smart TV) starts a hybrid flow and displays a QR code. The WebAuthn request includes the `fallbackUrl` extension with a URL (for example, https://example.com/login/fallback?session=123).

1. **Connection:** The Authenticator (e.g., user's smartphone) scans the QR code and establishes a connection (via BLE/Tunnel).

1. **Check:** The Authenticator looks for a discoverable credential (passkey) for the requested RP ID.

1. **Fallback trigger:** If no passkey is found, instead of showing only an error, the Authenticator may open the `fallbackUrl` in the device's browser (or hand off to an authorized app deep link).

1. **Recovery:** The user authenticates in the browser (or app). The RP's backend ties that session to the original client request (for example, using the `session` parameter) and completes authentication for the client device.



## Usability Considerations

This extension is primarily a usability fix designed to *"bridge the gap"* for users who are new to passkeys or switching devices.

- **Seamless recovery:** Users confused by a “passkey not found” message can be taken to a familiar web login on their phone, which most users understand and trust.

- **Device suitability:** This is especially useful for constrained clients (for example, Smart TVs or game consoles) where entering credentials is awkward. Moving the recovery interaction to the phone improves usability.

- **Adoption incentive:** RPs may be more comfortable offering passkey-first experiences if there is a reliable fallback path, reducing the risk of locking out users who lack a passkey on the scanned device.


## Security & Privacy Considerations

Introducing a redirect mechanism into the authentication flow requires strict safeguards to prevent abuse.

### 1. Origin binding

- **Risk:** An attacker could supply a malicious URL and cause the authenticator to open a phishing site.

- **Mitigation:** The fallbackUrl must be same-origin with the RP ID or strictly adhere to WebAuthn's origin matching rules. The authenticator should enforce this check before launching the URL to prevent open redirect vulnerabilities.

### 2. Privacy and probing

- **Risk:** A malicious RP could use this feature to silently probe whether a user has a passkey on their device. By triggering the flow and seeing if the fallback URL is hit, they might infer the absence of a credential.

- **Mitigation:**

  - The flow requires user interaction (scanning the QR code). This is not a silent background probe; the user actively consented to an authentication attempt.
  - Authenticators may choose to prompt the user before opening the fallback URL (e.g., "No passkey found. Open login page?") to ensure the user stays in control.


## Alternative Approaches
- Static "Well-Known" Fallback: Instead of sending a specific URL, the device could automatically go to a standardized endpoint (e.g., /.well-known/passkey-fallback). However, this lacks the ability to pass session context (state), requiring the user to manually re-enter codes or credentials, which hurts usability.


## References

- https://github.com/fido-alliance/fido-2-specs/issues/1543
- https://github.com/w3c/webauthn/issues/2341

## Glossary

- **Fallback URL** — A URL provided by the Relying Party (RP) in the WebAuthn request that the Authenticator may open (typically in a browser) if it cannot find a valid passkey for the requested RP ID. It prevents a dead-end user experience.

- **Hybrid Transport (Cross-Device Flow)** - A WebAuthn flow that allows a user to authenticate on one device (the Client, e.g., a Smart TV) by using a passkey stored on a different device (the Authenticator, e.g., a Smartphone). This is typically initiated by scanning a QR code.   

- **Relying Party (RP)** -  The entity (website, application, or service) that is requesting authentication from the user. In the context of this proposal, the RP is responsible for defining and hosting the `fallbackUrl`.   

- **Client** - The device where the user is attempting to sign in (e.g., a desktop computer, public kiosk, or Smart TV). In a Hybrid flow, this device displays the QR code and waits for the authentication result.   

- **Authenticator** - The personal device (typically a smartphone) used to scan the QR code and prove possession of the passkey. If the `fallbackUrl` extension is triggered, this device opens the URL to allow the user to recover or sign in via alternative methods.   