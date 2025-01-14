## Author
Adem Derinel <<derinel@google.com>

Last updated: 13-Jan-2025

## Summary

We propose an “immediate” mediation modality for WebAuthn `get()` requests which mirrors APIs on Android and iOS, and the “password” method in Credential Management, in response to requests from sites. This modality fails promptly if no credentials are immediately available, and thus allows sites to direct users to fallback sign-in methods in that case.

## Background

WebAuthn currently provides two UI flows for sign-in:

* Modal: browser UI always appears. The returned promise resolves when the user exercises a credential or dismisses the UI.  
* Conditional: browser UI may appear, typically integrated with autofill on a text box. The returned promise only resolves if the user exercises a credential.

Conditional UI has seen significant uptake but it requires an input field as the first step for authentication. Relying parties which want to go formless thus cannot use it. Also, users may ignore autofill and immediately enter their username manually.

Because the conditional UI is not always suitable, several sites make an educated guess about whether the user is likely to have an applicable credential and trigger the modal flow automatically. But the costs of guessing incorrectly are high as the resulting UI can be quite confusing to the user: these users may be prompted for the hybrid flow, but the hybrid flow is time consuming and not always applicable.

For a site where only a fraction of users have WebAuthn credentials (which is overwhelmingly common, for now), WebAuthn has no great answer for sites which want to implement a “Sign-in” button. We ultimately also want to design an API to help realize the original design of Credential Management and support sites making `get()` requests that accept credentials of any of several supported types, including WebAuthn, passwords, and federation.

![Current modal WebAuthn flow for a user with no local WebAuthn credentials. Whether it is the modal flow or the conditional flow, this may result in offering hybrid flow to the user.](https://github.com/user-attachments/assets/9deaa678-801b-485b-8298-b79ea8081b28)

*Current modal WebAuthn flow for a user with no local WebAuthn credentials. Whether it is the modal flow or the conditional flow, this may result in offering hybrid flow to the user.*

## API

We propose a mediation type, `immediate` for `navigator.credentials.get()`.

When such an option is set, the returned promise resolves with `NotFoundError` when there are no locally-available credentials; otherwise, the browser handles the authentication ceremony as if there were no mediation property set. Browsers are always free to return `NotFoundError` if they see fit. (See Privacy section, below.)

```javascript
try {
  const cred = await navigator.credentials.get({
    publicKey: {
      challenge: ...,
      rpId: 'example.com',
      allowCredentials: [],
    },
    mediation: 'immediate'
  });
} catch (error) {  if (error.name === 'NotFoundError') {
    // handle the no credential case
  } else {
    // other cases
  }
} 
```

### **Supporting cross-device authenticators**

This API risks disadvantaging users of security keys, and those who wish to keep credentials on their mobile device. We note that:

1. If a security key supports credential enumeration, as in newer CTAP drafts, those credentials can be considered to be immediately available.  
2. Sites will have to support another way to use WebAuthn credentials because, as noted in the Privacy section, we expect this API to always fail in Incognito/private browsing contexts.

### **Other credential types**

On supported browsers, password credentials (i.e. `navigator.credentials.get({password: true})`) return immediately when there are no passwords available.

Federated credentials and passwords can also support `mediation: ”immediate”` if needed to make a more coherent sign-in flow. With the increased support, relying parties can choose which credential type they want to use as the “primary” sign-in flow and implement the follow-up authentication methods as backup.

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
} catch (error) {  if (error.name === 'NotFoundError') {
    // No immediate WebAuthn, federated or password credentials found.
    // The relying party can fallback to their preferred solution such as
    // asking the user's phone number / email.
  } 
}
```

## Example use cases

Consider a relying party with an existing user base. They want to encourage users to adopt WebAuthn for improved security but also need to support traditional passwords.

The relying party’s goal is to provide a frictionless sign-in experience, minimizing confusion and unnecessary steps. They want to avoid overwhelming users with multiple sign-in options, especially those unfamiliar with passkeys. 

Here's how the relying party could use the new API to achieve this:

1. Upon page load, and after a user gesture (such as clicking a "Sign In" button), the relying party calls `navigator.credentials.get` with a `PublicKeyCredentialRequestOptions` object and `mediation: ”immediate”`. They may also include `password: true` in the request.  
2. The browser checks the local authenticators for any local credentials. Ideally, this would be near-instantaneous.  
3. If there are no local credentials  
   1. The browser throws a `NotFoundError` to the relying party.  
   2. The relying party asks the user for more details (e.g. email address)  
   3. The relying party shows the alternative authentication mechanisms such as a password form, SMS OTP, or the WebAuthn hybrid flow. They can also offer to create a passwordless account if the user details are not in their system.  
4. Otherwise (if there are local credentials):  
   1. The browser presents the required UI to the user for authentication.

![Example flow 1: User flows for a user with no local credentials. Without immediate mediation “Sign in” button would offer hybrid / security key flows, which are not common among users.](https://github.com/user-attachments/assets/73727190-3233-4f5e-bd53-a663a7e40531)

***Example flow 1:** User flows for a user with no local credentials. Without immediate mediation “Sign in” button would offer hybrid / security key flows, which are not common among users.*

![Example flow 2: User flow for a user with local credentials. If the user clicks “Cancel”, the relying party can decide to show a fallback mechanism.](https://github.com/user-attachments/assets/43c112fc-0daf-46fe-a304-2189a767f873)

***Example flow 2:** User flow for a user with local credentials. If the user clicks “Cancel”, the relying party can decide to show a fallback mechanism.*

## Privacy considerations

### **User gesture requirement**

To mitigate silent probing of credential availability and fingerprinting, we are considering requiring a user gesture before this API call can be made. This user gesture can be a button click and we seek feedback from sites on this.

### **Incognito and private sessions**

In incognito or private sessions, any immediate mediation request should throw `NotFoundError`. To avoid incognito fingerprinting, this response can be randomly delayed by the browser to simulate the browser fetching credential metadata from the system.

### **Request with allowlists**

Requests with allowlists should throw `NotAllowedError`. If a relying party queries for a list of credentials and gets a response indicating one exists, this could be used to infer whether the user has previously interacted with the site. Over time, this could allow tracking of users across different sessions. 

### **Cancelation**

Setting the `signal`  parameter on a request with immediate mediation is invalid as sites should not be able to programmatically dismiss any browser UI.

### **Multi RP ID probing**

A site could register every user on a unique subdomain then, using immediate mediation, probe every possible subdomain to find the identity of an unknown user. This is not terribly practical, and the user will ultimately see browser UI, but it is a concern. If implemented, a user gesture requirement would make this implausible. Otherwise some form of rate-limiting will be needed.

## Similar systems

Both Android and iOS APIs support immediately available credential calls for sign in. Android Credential Manager will respond with `NoCredentialException` similar to `NotFoundError` DOMException when the sign in request prefers immediately available credentials but none are available.
