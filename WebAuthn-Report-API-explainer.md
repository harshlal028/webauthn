# WebAuthn Report API explainer

## Authors

@arnar, @nsatragno

## Objective

Allow WebAuthn relying parties to report information about existing credentials back to credential storage providers, so that incorrect or revoked credentials can be updated or removed from provider and system UI.

## Background and motivation

Discoverable credentials, such as passkeys, can be requested with `navigator.credentials.get` with an empty `allowList`. In this case, if a user has any credentials for that relying party, they are presented with some UI to select which credential to use. If the user selects a credential, the resulting assertion carries the `user.id` value set at registration, allowing the relying party to resolve to an account without any further information.

This allows flows where the user is not required to enter a username, they simply select a credential. When used with conditional mediation, this further allows a smoother transition from traditional username entry, by allowing users that have passkeys to "fill" the username field using the passkey, and usually thereby omitting any further entry such as passwords or 2nd-factor authentication.

In such UI, passkeys are represented by the `user.name` and/or the `user.displayName` values that the relying party specified at registration. The user sees the entries as if they are _accounts_, however the entries correspond to individual credentials.

This pattern poses two main problems, given the current available APIs.

1. If a relying party stops accepting a credential, e.g. as a result of revoking it from an account or by completely deleting an account, the credential is still presented by clients during discoverable flows. 
2. Even if relying parties allow a user to change their username or display name on the account, such changes are not reflected in the display of credentials during discoverable flows.

The first case in particular is not only tied to explicit revocation or account deletion as requested by users. RPs may have policies that require them to revoke credentials after periods of inactivity. A common problem is also that the same user ends up with multiple accounts on a single RP unintentionally and may have a hard time keeping track of which accounts they want to use. Here the solution of account deletion or consolidation misses the mark if it cannot be represented in credential selection UI.

## Solution

A new API, `navigator.credentials.report`, allows relying parties to report such state updates back to user agents, who can forward these to the underlying credential providers. The API is opportunistic as there is no guarantee that the correct credential provider is reachable on the current client.

The API takes a relying party ID, and a number of _report types_. RPs may combine multiple reports in a single call. The set of report types is meant to be extendable in the future.

A report type is a key in a JSON structure, where the value of that entry sets additional parameters specific to the report type.

Each report type lists example scenarios in which it makes sense to send it, and possible credential provider actions. Note that any credential provider action is optional and at the discretion of each provider implementation.

### `unknownCredentialId`

This report names a credential ID and indicates that the relying party would reject an assertion with that credential because the credential ID is unknown to the RP.

```javascript
{
  unknownCredentialId: "vI0qOggiE3OT01ZRWBYz5l4MEgU0c7PmAA" // b64-url cred ID
}
```

_Usage scenario:_ Immediately following a response from `.get()` where the credential ID was not recognized. Appropriate to report even if the user was not authenticated.

_Example provider action:_ The credential may be marked for omission from future credential selection UI. If it was recently created and overwrote an existing credential, the previous one might be restored.

This situation may arise, for example, because the credential was revoked, or because the RP performed a create operation but failed to successfully store the public key on its backend.

### `currentCredentials`

This report names a `user.id` value and all accepted credential IDs, and/or updated values for `user.name` and/or `user.displayName` associated with that account.

```javascript
{
  currentCredentials: {
    userId: "M2YPl-KGnA8",
    allAcceptedCredentialIds: [
      "vI0qOggiE3OT01ZRWBYz5l4MEgU0c7PmAA",
      ...
    ],
    user: {  // See https://www.w3.org/TR/webauthn-3/#dictdef-publickeycredentialuserentityjson (minus user.id)
      name: "a.new.email.address@example.com",
      displayName: "J. Doe"
    }
  }
}
```

_Usage scenario:_ Immediately after an accepted `.get()` response, or at any time the user is authenticated and the set of accepted credentials, username or displayName have changed.

This report should only be made if the user has been fully authenticated.

It is not possible to update the `user.id` value. Either `allAcceptedCredentalIds` or `user` can be omitted, but not both.

_Example provider action:_ If `allAcceptedCredentialIds` is present, mark any non-appearing credential for the same RP ID and `user.id` value for omission in future account selectors. If the user value is present, update the credential store to use the supplied values in future UI representing this credential.

Note that it's at the provider’s discretion how to handle conflicts between manually edited usernames/displayNames and the RP-provided reports.

## Examples

A simple way for a relying party to report updates without tracking any additional state is to send a `currentCredentials` report after every successful sign-in (note that this can be done even if WebAuthn wasn't used to sign in).

```javascript
navigator.credentials.report({publicKey: {
  rpId: "example.com",
  currentCredentials: {
    userId: "M2YPl-KGnA8", // same as user.id at creation time
    user: {
      name: "currentemail@relying-party.com",
      userDisplayName: "J. Doe"
    },
    allAcceptedCredentalIds: [
      // IDs of all accepted credentials
      "vI0qOggiE3OT01ZRWBYz5l4MEgU0c7PmAA",
      "Bq43BPs"
    ]
  }
});
```

If a relying party receives an assertion with a credential that it does not recognize, it can report this back to the client. Note that it is safe to do this even if no user is signed in, as long as the credential id was already observed from this client.

```javascript
navigator.credentials.report({publicKey: {
  rpId: "example.com",
  unknownCredentialId: "vI0qOggiE3OT01ZRWBYz5l4MEgU0c7PmAA"
}});
```

If the user revokes or deletes a credential, e.g. in an account settings UI on the relying party's website, the relying party can opportunistically report this at that time with the `unknownCredentialId` type. However this will only have effect if the user agent is able to route the report to the same credential provider that created this credential. It may be better to send a `currentCredentials` report instead, with a complete list of valid credential IDs. In this case the `user` attributes can be omitted to signal they should not be updated.

Similarly, if a user changes their user- or display names while signed in, e.g. in an account settings UI, this can be reported to the current user agent without listing accepted credential ids:

```javascript
navigator.credentials.report({publicKey: {
  rpId: "example.com",
  currentCredentials: {
    userId: "M2YPl-KGnA8", // same as user.id at creation time
    user: {
      name: "currentemail@relying-party.com",
      userDisplayName: "J. Doe"
    }
  }
});
```

In that case no credentials will be marked for removal. There is no _harm_ in also reporting accepted credential IDs, and indeed that report may reach credential providers that have outdated credentials and haven't received previous reports. But the choice is there if e.g. loading the list of accepted credentials would require additional (re-)authentication of the user, or other reasons that data isn't easily available on the server.