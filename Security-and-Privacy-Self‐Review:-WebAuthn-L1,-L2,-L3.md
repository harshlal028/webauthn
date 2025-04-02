Since the questionnaire has not previously been performed for WebAuthn L1 or L2,
the following answers pertain to all of WebAuthn, not just the additions in L3.

# 01.  What information does this feature expose, and for what purposes?

In summary: information about cryptographic authentication keys created using the API,
and authentication signatures created with those keys.
These data include:

- Credential IDs: Opaque, authenticator-chosen identifiers for credentials.
  The purpose is to identify returning users, with user consent.
- User handles: Opaque, RP-chosen identifier for the user, chosen during credential creation and echoed during authentication.
  The purpose is to identify returning users, with user consent.
- Authentication signatures by the user's credential private key.
  Signatures are made over a combination of RP-chosen challenge values,
  client-chosen contextual data and authenticator-chosen contextual data.
  The purpose is to authenticate users, with user consent.

  Client-chosen context data inclues:
  - The challenge value chosen by the RP.
  - The origin of the web page where the operation was performed.
  - Whether the operation was performed in a cross-origin iframe.
  - The origin of the top-level parent frame of the web page where the operation was performed.

  Authenticator-chosen context data includes:
  - Bit flags indicating user presence (UP), user verification (UV), credential backup eligibility (BE) and backup state (BS).
  - (Optionally) a signature counter per credential,
    possibly using a shared global counter with per-credential increment step size.
  - (Optionally) additional information added by WebAuthn extensions.
    No such extensions are defined inline in WebAuthn L3.

- Attestation signatures by the authenticator's attestation private key.
  The purpose is to prove provenance of authentication keys.
  Attestation signatures are made over the same context data as authentication signatures,
  with the addition of:

  - The Authenticator Attestation GUID (AAGUID) of the authenticator used to create the credential, if set.
    This AAGUID identifies an authenticator model or client platform, not an individual user.
  - The credential ID for the newly created credential.
  - The public key of the newly created credential.

- Attestation certificates containing the authenticator's attestation public key
  and other information supplied by the authenticator vendor and attestation CA.
  Like the AAGUID, the attestation certificate identifies an authenticator model or client platform, not an individual user.
  The purpose is to convey security assurances from the authenticator vendor to the RP,
  which combined with an attestation signature enables the RP to trust the authenticator-chosen context data.

- Authenticator attachment: an indication of whether the operation was completed using a built-in or detachable authenticator.
  The purpose is to inform the RP of what user flows are possible with the credential.

- (Optionally) Transport hints for how the client might communicate with the authenticator that created the credential.
  This is a list of zero or more strings taken from an enumeration of (currently) 6 valid values.
  The purpose is for these to be fed back to the client in subsequent calls so that the client may optimize UI for the given situation.

- A feature detection API for determining the capabilities of the client platform.
  The purpose is to enable RP applications to gracefully handle situations where some desired functions are unavailable.

- Any additional information added by WebAuthn extensions.

In addition there are WebDriver extensions for automating WebAuthn interactions for testing.
These might enable fingerprinting but do not expose any other information.

All of the above information is exposed to the **first party**, the WebAuthn Relying Party (RP).
None of the above information is exposed to **third parties**, unless forwarded by the first party.

All of the above information is information the parties cannot currently easily determine,
with the possible exception of the feature detection API which may be possible to determine using other methods such as User-Agent header inspection and browser fingerprinting.

The above information includes some pieces of _potentially identifying information_.
The only part available without the user's authorization is the following:

- The feature detection API obviously allows detecting browser and platform capabilities.
  This exposes (currently) 9 tri-state capability flags, so theoretically up to 14.26 bits of fingerprinting information.
  Some capabilities are likely to be highly correlated, though:
  for example the three capabilities relating to the WebAuthn Signal API are likely to all have the same value,
  and the passkeyPlatformAuthenticator capability is an alias of two other capabilities,
  reducing the probable value space to 9.5 bits.
  The feature detection API is only exposed to the **first party**.

- Similarly the presence or absence of the WebDriver extensions may be used for fingerprinting.
  TODO: Help from @nsatragno please?

The following parts are available only with the user's authorization to create or authenticate with a WebAuthn credential:

- The RP could put identifying information into the user handle;
  however, the RP must already know this information since the user handle is chosen by the RP.
  WebAuthn allows the RP to re-discover this identifier in a new session,
  but does not introduce a way for the RP to initially discover this information.
  WebAuthn strongly recommends against putting identifying information into the user handle
  since the user handle may be discoverable outside of the Web platform;
  for example after a credential is stored on an external USB security key,
  the user handle may be discoverable by the USB host.
  On the Web, user handles are only revealed to the **first party** (the RP that created them), never to **third parties** (other web domains).

- Each credential ID is a (probabilistically) unique identifier for a credential,
  and thus a unique identifier for the user who created that credential.
  Like user handles, credential IDs are only revealed to the **first party** on the web,
  but may be discoverable by other entities outside the Web platform by direct interaction with the authenticator.

- Attestation certificates and AAGUIDs are potentially identifying information.
  The intent is that they are shared between all users of a given platform or authenticator model,
  optionally in bounded but large batches to limit impact in case an attestation key is compromised.
  WebAuthn references FIDO Alliance guidance which recommends batches of at least 100,000 authenticator units
  sharing the same AAGUID and attestation certificate.
  Attestation certificates and AAGUIDs are only revealed to the **first party**, never to **third parties**.

- An "Enterprise Attestation" (EA) mode which expands the attestation features to allow personally identifying information
  that may be shared between RPs.
  This is intended for controlled environments such as an employer provisioning authenticators to employees
  and using the EA identifier to automatically enroll the employee to the company intranet.
  This feature is clearly a big privacy threat and therefore subject to strict control of which RPs are allowed to access it:
  the authenticator must be specially configured with a list of EA-enabled RP domains
  or configured to allow client policy to decide EA-enabled RP domains.
  Both options are disabled on "consumer-grade" authenticators.

- The optional signature counter is potentially identifying information,
  and can be correlatable between different RPs
  if the authenticator uses the same counter across all credentials without some layer of obfuscation.
  WebAuthn recommends that signature counters, when implemented,
  be separate between credentials to avoid leaking information between RPs.
  Authenticators may also choose to not implement a signature counter, and instead report a constant signature count of zero.

- WebAuthn extensions may add additional information that could be potentially identifying.
  Extensions defined inline in WebAuthn L3 are `appid`, `appidExclude`, `credProps`, `prf` and `largeBlob`.

  `appid` and `appidExclude` expose no potentially identifying information.

  `credProps` exposes a single tri-state attribute expressing whether the created credential is a "discoverable credential",
  which may provide 1.58 bits of fingerprinting information.
  Its purpose is to inform the RP of what user flows are possible with the credential.

  `prf` exposes a per-credential pseudo-random function (PRF) generating 32-byte binary strings.
  Its purpose is to provide a per-credential entropy source for use cases such as deriving keys for end-to-end encryption.
  Assuming compliant authenticator implementation this provides no fingerprinting information and no identifying information not already implied by the credential ID.

  `largeBlob` allows the RP to store and retrieve moderately large (up to one or a few kB, depending on authenticator) binary information associated with a credential.
  Its purpose is to carry auxiliary information such as an X.509 certificate associated with the user or credential.
  Assuming compliant authenticator implementation this provides no fingerprinting information and no identifying information not already implied by the credential ID.


# 02.  Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?

In spirit yes, much of the information is security-critical:

- Public keys and authentication signatures are the core value proposition:
  to be able to use strong public-key cryptography to authenticate users.

- The User Presence (UP) flag is necessary to detect and prevent unattended authentication attempts,
  for example from remotely-operated malware.

- The User Verification (UV) flag is necessary to enable WebAuthn to deliver privacy-friendly two-factor authentication (2FA):
  the RP only receives a single bit asserting that 2FA has been performed,
  and does not need to know or store a remembered secret or biometric information.

- The client-chosen context data is necessary for domain separation:
  to detect and prevent cross-origin authentication attempts, for example from "phishing" websites
  attempting to intercept authentication credentials and hijack the session.
  This combines with client-enforced origin validation to provide two layers of defense:
  first, webpages are not allowed to request authentication credentials for other websites;
  and second, the authenticator signs the origin of the invoking webpage
  so that the recipient server can verify that the authentication was performed on an authorized domain.
  This domain binding is one of the primary innovations of WebAuthn compared to other authentication mechanisms.

- Credential IDs are necessary to identify to the verification server
  which user is attempting to authenticate and which public key to verify the signature against.

However, some pieces of information are not _strictly_ necessary
but help improve security or the user experience:

- Attestation is not _strictly necessary_ for most applications,
  but is necessary for applications that are subject to regulatory requirements on their security.
  Attestation may also inform high-security RPs' risk scoring even when not required by regulation.
  Attestation is disabled by default; any RP may request it, but some authenticators might not provide it.

- User handles are not _strictly necessary_, as the credential ID is enough to uniquely identify a user,
  but the user handle provides RPs better control of how to structure their database
  since the credential ID is chosen by the authenticator.
  Since the user handle is chosen by the RP, it gives the RP control of the user handle as a database key
  while not introducing any new information not already known by the RP.

- The Backup Eligibility (BE) and Backup State (BS) flags are not _strictly necessary_,
  but allow the RP to evaluate whether to urge the user to register a backup authentication method
  in case they lose their authenticator.

- The authenticator attachment attribute is not _strictly necessary_,
  but helps the RP explain to the user how to use the credential.

- The transport hints are not _strictly necessary_,
  but helps the client optimize UI to reduce the risk of confusing the user.
  For example, if an RP requests authentication with only "built-in" transport hints
  and no eligible credentials exist on the client device,
  the client may inform the user of the error
  instead of asking the user to connect an external authenticator they might not even have.
  When emitted by the client, transport hints are sorted to prevent fingerprinting by the order of entries.

- The feature detection API is not _strictly necessary_ for the protocol to function,
  but is necessary for widespread adoption to give RPs the tools to craft a robust user experience
  that degrades gracefully and for example doesn't offer attempts to use features that are known to be unavailable.


# 03.  Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?

No. The RP could choose to store personal information, PII or information derived from either
in a user handle, but WebAuthn actively recommends against this.
Even if an RP does, that information must first be discovered outside of WebAuthn
since WebAuthn provides no way for the RP to initally discover it.

Public keys and credential IDs in WebAuthn are initially anonymous,
and become personally identifying pseudonyms only after being associated with a user or account by a registration ceremony.

The exception to this is "enterprise attestation" (EA) as described above,
which may expose a unique identifier shared between eligible RPs.
Access to this feature is strictly controlled as explained above,
and cannot be enabled at solely the RP's discretion.


# 04.  How do the features in your specification deal with sensitive information?

Whether the user has a given kind of authenticator may be sensitive information:
for example, a user who owns a dedicated external security key device
may be more likely to be a "person especially at risk"
such as a journalist, political dissident or government official.
WebAuthn exposes such information only after a successful registration ceremony -
which requires active user consent and participation -
and only if the RP requested attestation.

RPs can silently probe only for the existence of specifically a "user-verifying platform authenticator",
which is by definition a capability built into the client platform
and therefore says little more about the user than what may be otherwise discoverable
by inspecting the User-Agent string or other browser fingerprinting.

We are not aware of any other sensitive information exposed by WebAuthn.


# 05.  Does data exposed by your specification carry related but distinct information that may not be obvious to users?

Yes, mainly in the form of attestation and extensions:

- Attestation statements may carry various information about the authenticator,
  such as vendor entity, certification identifiers
  and/or various other authenticator properties reported by the vendor.
  Current client implementations inform the user to the effect of
  "your authenticator make and model may be visible to the site",
  and some allow the user to choose to not reveal the attestation.
  The most prominent option is usually to allow attestation, however,
  and WebAuthn does not formally prescribe a warning nor an opt-out option.

  The privacy considerations section §14.4.1. Attestation Privacy
  describes some ways to prevent tracking users by attestation,
  but WebAuthn does not otherwise attempt to prescribe or limit
  what information might be conveyed by an attestation statement.

  Current implementation experience indicates clients are unforgiving of
  and tend to censor overly-invasive attestation statements,
  but this is not formally prescribed by the WebAuthn specification.

- Extensions may carry arbitrary data,
  and no current extensions specify any indication to the user
  that the extension is being used or what information it conveys.

  - The `prf` extension conveys only opaque cryptographic data that is not meaningful on its own,
    such as entropy to be used as key generation material.

  - The `largeBlob` extension may convey arbitrary binary data that was previously set by the RP.
    RPs cannot use this to discover new information unless it was loaded into the authenticator
    by some external mechanism and explicitly tagged as associated with that RP.


# 06.  Do the features in your specification introduce state that persists across browsing sessions?

Yes, by nature of being an authentication API.
Credentials and any associated information persist across browsing sessions
but are not shared between web origins.
All access to WebAuthn credentials and any associated information
requires a registration or authentication ceremony with active user consent and participation.
User handles are immutable after credential creation and thus cannot be used to store changing information;
though credentials may be overwritten with a new one by using the same user handle in a new registration ceremony.

The `largeBlob` extension allows RPs to store arbitrary mutable information associated with a credential.
This too is only accessible via an authentication ceremony with active user participation.


# 07.  Do the features in your specification expose information about the underlying platform to origins?

Yes, attestation and the feature detection API expose information
about the authenticator and underlying platform, as described in previous answers.
Generally the same information is exposed to all origins.
Attestation is exposed only with active user consent
while the feature detection API may be accessed silently.


# 08.  Does this specification allow an origin to send data to the underlying platform?

Yes, using credential creation and request options objects
based on interfaces defined by the Credential Management API.
These include many data fields, most notably including:

- An opaque binary challenge value, which the client base64-encodes
  and includes in the client-chosen context data.
- An opaque binary user handle, described in previous answers.
- Free-form "user name", "user display name" and "RP display name" text labels
  that may be displayed to the user during credential selection,
  but are never conveyed back to the RP.
- Several enum-valued feature selection attributes,
  and an integer-valued algorithm selection attribute.
- An integer-valued timeout attribute.

The WebAuthn Signal API also includes methods for sending data to the platform
and any applicable authenticator(s).
These methods convey updated credential information,
so their data fields are a subset of those in the intitial credential creation options.


# 09.  Do features in this specification enable access to device sensors?

No.

WebAuthn L1 defines a `loc` (location) extension
which would convey a `Coordinates` record as defined by the Geolocation API,
but this extension is not known to be supported by any client implementation
and is not included inline in WebAuthn L2 or L3.


# 10.  Do features in this specification enable new script execution/loading mechanisms?

No.


# 11.  Do features in this specification allow an origin to access other devices?

Yes: authenticators specifically designed for the purpose.
This includes three categories:

- "Platform authenticators", which are built into the client platform
  and made available as WebAuthn authenticators by some platform-specific or client-internal API.
- "Roaming authenticators": dedicated hardened authenticator devices ("security keys")
  designed specifically for integration with WebAuthn, and sometimes other similar protocols.
  In practice these (currently) use the CTAP1 or CTAP2 protocols defined by the FIDO Alliance.
- "Hybrid authenticators": other client devices with a platform authenticator, usually smartphones,
  made available as roaming authenticators on other client devices,
  usually by a combination of BLE and network connection.
  The target device must explicitly implement support for this integration.

WebAuthn does not allow RPs to enumerate available devices
and does not expose device identifiers apart from any that may be present in attestation statements.


# 12.  Do features in this specification allow an origin some measure of control over a user agent's native UI?

No, only over when to display relevant UI and over some values that may be displayed.

In theory the "RP display name" parameter for credential creation
may appear in client UI during the registration ceremony,
but all known client implementations currently ignore the parameter and never display it.
The parameter is therefore deprecated in WebAuthn L3.

The "user name" and "user display name" parameters
may appear in confirmation prompts during credential creation
and credential selection UI during authentication.

The "conditional mediation" mode, also called "autofill UI",
modifies under what conditions clients show WebAuthn UI,
but does not modify the form of that UI.


# 13.  What temporary identifiers do the features in this specification create or expose to the web?

None; all identifiers in WebAuthn L3 are long-lived.

In WebAuthn L1 and L2, the optional `tokenBinding` attribute of the client-chosen context data
would convey a Token Binding ID, which may be shorter-lived than the WebAuthn credential.
This feature is not known to be supported by any client implementation,
and the attribute is removed from client processing steps in WebAuthn L3.

WebAuthn L1 defines a `uvi` (user verification index) extension
which would expose a unique identifier for a user verification record,
such as a PIN or a biometric template.
The identifier would change if the (for example) PIN or biometric template was changed,
and so be possibly shorter-lived than the parent credential.
This extension is not known to be supported by any client implementation
and is not included inline in WebAuthn L2 or L3.


# 14.  How does this specification distinguish between behavior in first-party and third-party contexts?

WebAuthn defines Permissions Policy feature-identifier tokens
for controlling use of WebAuthn in cross-origin iframes.
Both are set to `'self'` by default, allowing use of WebAuthn only in same-origin context.


# 15.  How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

As noted in question 6, WebAuthn enables creation of identifiers that persist after a browsing session ends.
WebAuthn does not prescribe how its features should interact with private browsing modes.


# 16.  Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

Yes, though they do not include all details elaborated on here.


# 17.  Do features in your specification enable origins to downgrade default security protections?

No. WebAuthn allows feature selection for some of its features,
but these do not impact other security protections outside of WebAuthn.


# 18.  What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?

Nothing; WebAuthn state is not cached.


# 19.  What happens when a document that uses your feature gets disconnected?

WebAuthn does not preserve state between ceremonies,
so new ceremonies are not affected. In progress ceremonies are aborted.


# 20.  Does your spec define when and how new kinds of errors should be raised?

Yes. The error conditions are carefully chosen to not leak unintended or sensitive information,
and some error conditions require user consent before returning a particular error,
otherwise returning the unspecified general error instead.


# 21.  Does your feature allow sites to learn about the user's use of assistive technology?

Not as far as we know.

Attestation statements do include explicit information about authenticator model names,
but these are not known to correlate with assistive technologies.

Users of assistive technologies might take longer or shorter time than other users
to complete a registration or authentication ceremony, and RPs can measure that time taken.
However, the greatest difference lies between uses of "platform authenticators",
"roaming authenticators" and "hybrid authenticators" as described in question 11,
with ceremonies typically taking longer time for each authenticator category in approximately that order.


# 22.  What should this questionnaire have asked?

(No answer)
