## Author
Adem Derinel <<derinel@google.com>>

Ken Buchanan <<kenrb@chromium.org>>

Last updated: 23-Jun-2025

## Summary

We propose an “immediate” mediation modality for WebAuthn and password `CredentialsContainer::get()` requests that mirrors the `preferImmediatelyAvailable` API properties on Android and iOS. This modality fails promptly if no credentials are immediately available, and thus allows sites to direct users to fallback sign-in methods in that case.

## Goal

This feature aims to enable a sign-in flow with passkeys and managed passwords that does not require a user to visit a traditional sign-in page containing multiple sign-in methods for the user to choose between (such as a username/password form, multiple federated sign-in options, a recovery button, etc), while at the same time not changing the sign-in experience for users for whom passkeys or managed password are not available. When one or more such credential are available, a user who has reached a sign-in moment in their interaction with a site (such as, for example, by clicking a "Sign In" button) will see a browser dialog containing a list of existing eligible credentials for that site. When the user confirms the credential to use (or selects a credential, if multiple are shown), they can be immediately signed in.

## Background

WebAuthn currently provides two UI flows for sign-in:

* Modal: browser UI always appears. The returned promise resolves when the user exercises a credential or dismisses the UI.  
* Conditional: browser UI may appear, typically integrated with autofill on a text box. The returned promise only resolves if the user exercises a credential.

The `preferImmediatelyAvailable` option on mobile platforms provides a lower-friction flow when there is an eligible credential. In that case it immediately displays UI containing available credentials, but if no credential is available then it returns an error so that the calling application can provide alternative sign-in methods. This is similar to conditional UI on the web, but in that case the relying party does not learn whether a credential is available and therefore has to provide all sign-in options on a single surface.

For a site where only a fraction of users have WebAuthn credentials, WebAuthn has no great answer for sites that want to implement a “Sign-in” button. We ultimately also want to design an API to help realize the original design of Credential Management and support sites making `get()` requests that accept credentials of any of several supported types, including WebAuthn, passwords, and federation.

![Current modal WebAuthn flow for a user with no local WebAuthn credentials. Whether it is the modal flow or the conditional flow, this may result in offering hybrid flow to the user.](https://github.com/user-attachments/assets/f4442367-4ce8-4096-90f8-89ac65837619)

*Current modal WebAuthn flow for a user with no local WebAuthn credentials. Whether it is the modal flow or the conditional flow, this may result in offering hybrid flow to the user.*

## API

> Note from authors: We have received feedback suggesting that adding a value to the `mediation` enum might restrict use cases with some credential types. In particular, credentials such as FedCM or passwords have different client behaviour when the RP sets `mediation: "required"` vs `mediation: "optional"`. If we support combining those credentials with passkeys in `mediation: "immediate"`, RPs will have less control over non-passkey credential mediation on the associated UI than they currently do. There is ongoing discussion over whether a new field should be added.

We propose a mediation type, `immediate` for `navigator.credentials.get()`.

When such an option is set, the returned promise resolves with `NotAllowedError` when there are no locally-available credentials; otherwise, the browser handles the authentication ceremony as if there were no mediation property set. Browsers are always free to return `NotAllowedError` if they see fit. (See Privacy section, below.)

```javascript
// Use `getClientCapabilities` for feature detection
let immediateMediationAvailable = false;
if (window.PublicKeyCredential && PublicKeyCredential.getClientCapabilities) {
  const capabilities = await PublicKeyCredential.getClientCapabilities();
  // `immediateGet` is a new capability for immediate mediation:
  immediateMediationAvailable = capabilities.immediateGet === true;
}

if (immediateMediationAvailable) {
  try {
    const cred = await navigator.credentials.get({
      publicKey: {
        challenge: ...,
        rpId: 'example.com',
        allowCredentials: [],
      },
      mediation: 'immediate'
    });
  } catch (error) {
    if (error.name === 'NotAllowedError') {
      // handle the no credential or cancellation case
    } else {
      // other cases
    }
  } 
}
```

### **Supporting cross-device authenticators**

The UI associated with this API will only show credentials that are known to the user agent or can be discovered without user action. Typically this will not include credentials stored on security keys or on mobile devices, which would be usable through the hybrid transport. We note that:

1. If a security key and the platform both support CTAP 2.2 credential enumeration, those credentials can be considered to be immediately available.  
2. Sites will have to support another way to use WebAuthn credentials because, as noted in the Privacy section, we expect this API to always fail in Incognito/private browsing contexts.

When security key credential enumeration is not available, the sign-in experience for users using security key credentials will typically remain unchanged. No UI will be shown from this call unless there are also credentials from that site available from a platform authenticator, after which the user will proceed to the site's sign-in page.

### **Other credential types**

On supported browsers, password credentials (i.e. `navigator.credentials.get({password: true})`) return immediately when there are no passwords available.

Federated credentials and passwords can also support immediate mediation if needed to make a more coherent sign-in flow. With the increased support, relying parties can choose which credential type they want to use as the “primary” sign-in flow and implement the follow-up authentication methods as backup.

While not addressed in this explainer, a future direction of requesting WebAuthn, federated and password credentials together could look like

```javascript

try {
  const cred = await navigator.credentials.get({
    publicKey: {
      challenge: ...,
      rpId: 'example.com',
      allowCredentials: [],
    },
    identity: { ... },
    password: true,
    mediation: "immediate"
  });
  // Site will also show a button to trigger a modal flow
  // to handle Incognito and security key users
} catch (error) {  
  if (error.name === 'NotAllowedError') {
    // No immediate WebAuthn, federated or password credentials found, or 
    // the user dismissed the browser UI.
    // The relying party can fallback to their preferred solution such as
    // asking the user's phone number / email.
  } 
}
```

## Example use cases

Consider a relying party with an existing user base. They want to use a passkey as easily as possible for users that have them, but users that don’t have passkeys should see the standard sign-in UI. 

The relying party’s goal is to provide a frictionless sign-in experience, minimizing confusion and unnecessary steps. They want to avoid overwhelming users with multiple sign-in options, especially those unfamiliar with passkeys. 

Here's how the relying party could use the new API to achieve this:

1. User navigates to the main page of the website (e.g. a shopping page).  
2. Upon page load, and after a user gesture (such as clicking a "Sign In" button), the relying party calls `navigator.credentials.get` with a `PublicKeyCredentialRequestOptions` object and `mediation: ”immediate”`. They may also include `password: true` in the request.  
3. The browser checks the local authenticators for any local credentials. Ideally, this would be near-instantaneous.  
4. If there are no local credentials  
   1. The browser throws a `NotAllowedError` to the relying party.  
   2. The relying party asks the user for more details (e.g. email address)  
   3. The relying party shows the alternative authentication mechanisms such as a password form, SMS OTP, or the WebAuthn hybrid flow. They can also offer to create a passwordless account if the user details are not in their system.  
5. Otherwise (if there are local credentials):  
   1. The browser presents the required UI to the user for authentication.

![Example flows: If there are WebAuthn credentials (or passwords) locally, browser UI will prompt the user to select one. The user can choose another way in the browser UI to fallback to the sign-in / sign-up page. In the case of no WebAuthn credentials locally, the website should show the existing sign-in / sign-up page.)](https://github.com/user-attachments/assets/286f094e-b4e8-49c2-ad14-b6161d02aa5d)
***Example flows:** If there are WebAuthn credentials (or passwords) locally, browser UI will prompt the user to select one. The user can choose another way in the browser UI to fallback to the sign-in / sign-up page. In the case of no WebAuthn credentials locally, the website should show the existing sign-in / sign-up page.*


## Privacy considerations

Currently the RP does not have a way to learn about the availability of WebAuthn credentials until the user interacts with browser API, authorizing the generation of an assertion. Under this proposal that would change, enabling the RP to learn about the presence of immediately available credentials without such an authorization. It would not learn any information about the credentials until the assertion is returned, but the single bit available from the API returning immediately with a `NotAllowedError`, or having a long delay due to UI being shown to the user, represents a relaxation of WebAuthn privacy protections.

We propose the following measures to mitigate the potential for abuse of that relaxation:

### **User gesture requirement**

To mitigate silent probing of credential availability and fingerprinting, we will require a user gesture before this API call can be made. The user gesture could be [any transient user activation](https://developer.mozilla.org/en-US/docs/Web/API/UserActivation). In particular this requirement makes it difficult for a site to do many calls with varying RP IDs.

### **Incognito and private sessions**

In incognito or private browsing sessions, any immediate mediation request should throw `NotAllowedError`. To avoid incognito fingerprinting, this response can be delayed by the browser to simulate the browser fetching credential metadata from the system.

### **Request with allowlists**

Requests with allowlists should throw `NotAllowedError`. If a relying party queries for a list of credentials and gets a response indicating one exists, this could be used to infer whether the user has previously interacted with the site. Over time, this could allow tracking of users across different sessions. 

### **Cancellation**

Setting the `signal`  parameter on a request with immediate mediation is invalid as sites should not be able to programmatically dismiss any browser UI.

## Similar systems

Both Android and iOS APIs support immediately available credential calls for sign in. Android Credential Manager will respond with `NoCredentialException` similar to `NotAllowedError` DOMException when the sign in request prefers immediately available credentials but none are available.
