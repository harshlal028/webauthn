# Experimenting with the WebAuthn Signal API on Chrome

An initial implementation of the [Signal API](https://w3c.github.io/webauthn/#sctn-signal-methods) is present on Chrome 130.0.6697.0+.

## Requirements
* A Google account.
* [The latest available version of Chrome for your platform](https://www.google.com/intl/en_ca/chrome/canary/).
* A platform that supports Google Password Manager passkeys, which can be either of these:
  * Windows with a TPM
  * MacOS
  * Linux
  * ChromeOS

## Instructions

* Enable the web platform experimental features flag (chrome://flags/#enable-experimental-web-platform-features) and restart Chrome.
* Sign in to Chrome with a Google account if you're not signed in already.
* When creating a passkey, choose "Google Password Manager" and follow instructions. You might have to create a PIN or enter the lockscreen for your Android phone.
* Try calling the [signal family of methods](https://w3c.github.io/webauthn/#sctn-signal-methods).

See [this demo](https://signal-api-demo.glitch.me/) for an example front-end only implementation.

## Limitations

For now, only Google Password Manager passkeys are affected by the API, and only on Chrome desktop.