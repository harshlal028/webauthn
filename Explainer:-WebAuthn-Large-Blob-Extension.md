## Explainer: WebAuthn Large Blob Extension

### Summary

A pair of registration/assertion extensions for WebAuthn that let relying parties store a small amount of data associated to a credential ([spec](https://w3c.github.io/webauthn/#sctn-large-blob-extension)).

### Background

Relying parties are websites, and therefore have access to servers that can store arbitrary amounts of data about a user. However, there are specific cases where a relying party might want to store some data associated to a user's account that can be used in an offline authentication context, such as a certificate. The large blob extension allows relying parties to store and retrieve such data during an [assertion ceremony](https://w3c.github.io/webauthn/#sctn-getAssertion).

### API

#### Registration

Large blobs can only be stored and retrieved at assertion time. However, relying parties should query support during registration:

```javascript
const credential = await navigator.credentials.create({
  publicKey: {
    challenge: ...,
    user: ...,
    rp: ...,
    authenticatorSelection: {
      residentKey: "preferred",  // or "required"
    },
    extensions: {
      largeBlob: {
        support: "preferred",  // or "required"
      },
    },
  }
});

if (!credential.getClientExtensionResults().largeBlob) {
  // Large blob not supported by the user agent.
  return;
}

if (credential.getClientExtensionResults().largeBlob.supported) {
  // Large blob is supported for this credential.
} else {
  // Large blob is not supported (this happens if support="preferred" -- the credential was still created).
}
```

If support is set to `required`, the user agent will only create a credential for authenticators with large blob support, and can inform the user of that need. Otherwise, the credential will be created regardless of support, and support will be advertised by the `largeBlob.support` registration extension output.

Note that discoverable credential ("resident key") support is required to store a large blob on CTAP 2.1 authenticators, so it is recommended to set `residentKey` to `preferred` or `required`. This restriction may not apply to other (e.g. platform) authenticators.

#### Assertion

To store a large blob on a credential that has advertised support at its creation time:

```javascript
const blob = Uint8Array.from(blobBits);
const assertion = await navigator.credentials.get({
  publicKey: {
    challenge: ...,
    allowCredentials: [{
      type: "public-key",
      id: credentialId,  // only a single credential is supported.
    }],
    extensions: {
      largeBlob: {
        write: blob,
      },
    },
  }
});

if (assertion.getClientExtensionResults().largeBlob.written) {
  // Success, the large blob was written.
} else {
  // The large blob could not be written (e.g. because of a lack of space).
  // The assertion is still valid.
}
```

Exactly one `allowCredentials` entry is allowed for writing a large blob.

To read a large blob:

```javascript
const assertion = await navigator.credentials.get({
  publicKey: {
    challenge: ...,
    allowCredentials: [{
      type: "public-key",
      id: credentialId,  // an arbitrary number of credentials is supported
    }],
    extensions: {
      largeBlob: {
        read: true,
      },
    },
  }
});

if (typeof assertion.getClientExtensionResults().largeBlob.read !== "undefined") {
  // Reading a large blob was successful.
  const blobBits = new Uint8Array(assertion.getClientExtensionResults().largeBlob.read);
} else {
  // The large blob could not be read (e.g. because the data is corrupted).
  // The assertion is still valid.
}
```

When reading a large blob, any number of entries is allowed in `allowCredentials` (including none, for a discoverable credential request).