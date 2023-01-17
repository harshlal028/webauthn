Security keys are physical devices, often USB-connected, that can create public–private key pairs and sign with the private keys to authenticate a user. Websites can use them via the [WebAuthn API](https://www.w3.org/TR/webauthn-2/). Several major sites allow users to register security keys for better account security, for example, Microsoft, Dropbox, GitHub, Google, and Facebook, to name a few.

There are several cases where sites would like to combine authentication and release of a secret key. Any site that wants to do end-to-end encryption needs to store key material somewhere, and since security keys are high-security enclaves, several inquiries have been made about whether a WebAuthn assertion could also contain key material.

The underlying protocol for security keys, CTAP2, includes [an optional extension](https://fidoalliance.org/specs/fido-v2.1-rd-20210309/fido-client-to-authenticator-protocol-v2.1-rd-20210309.html#sctn-hmac-secret-extension) (`hmac-secret`) to facilitate this. It was originally designed to enable security keys to decrypt local storage when signing into a computer (i.e. not for a web context) but it exposes a generic pseudo-random function that is useful for lots of things.

A psuedo-random function (PRF) is a function that is externally indistinguishable from a random oracle for a computationally-bound attacker. A [random oracle](https://en.wikipedia.org/wiki/Random_oracle) is a function like this:

```py
oracle_outputs = {}
def random_oracle(x):
  if x not in oracle_outputs:
    value = generate_random_output()
    oracle_outputs[x] = value

  return oracle_outputs[x]
```

Concretely, HMAC with a random key and strong hash function is a practical PRF.

The WebAuthn [`prf` extension](https://w3c.github.io/webauthn/#prf-extension) allows sites to request that a WebAuthn authenticator create a PRF along with a credential and allows sites to query that PRF during assertions. Since this extension can be implementing by using the CTAP2 `hmac-secret` extension, and because many security keys support that, it should immediately have quite wide support. (At least in the subset of users who use security keys.)

The most basic pattern of use would be to request the evaluation of a fixed value in every assertion. I.e. request the evaluation of &ldquo;end-to-end encryption key&rdquo; every time. That will cause WebAuthn assertions, where supported, to contain a per-credential, 32-byte secret key that can be used for that purpose with, for example, the WebCrypto API.

The API also supports more complex uses by allowing each assertion to query the PRF at two inputs. This allows for automatic key rotation if the server generates random evaluation points by getting a "current" and "next" encryption key with each assertion, and rotating the evaluation points over time.

In order that exposing the outputs of the `hmac-secret` extension to the web not invalidate the security assumptions of any non-web users, the PRF evaluation points are hashed with a fixed prefix before use to partition the PRF space. (Assuming that an attacker cannot calculate preimages for SHA-256.)

