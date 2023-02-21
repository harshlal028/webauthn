TAG review requires filling a [Security and Privacy questionnaire](https://w3ctag.github.io/security-questionnaire/).

1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

   The large blob extension will expose the following information to relying parties (i.e. websites):
     * On registration, whether the authenticator selected by the user supports large blobs or not. This lets relying parties decide e.g. to show an error to the user or to disable whichever feature would have made use of large blobs.
     * On assertion, the large blob that was previously stored by the same relying party. This is the point of the feature. The blob will only ever be revealed to the relying party after the user consents to asserting a credential.

2. Do features in your specification expose the minimum amount of information necessary to enable their intended uses?

   Correct. No other information is revealed by the use of the extension.

3. How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?

   For WebAuthn in general, see [security](https://w3c.github.io/webauthn/#sctn-security-considerations) and [privacy](https://w3c.github.io/webauthn/#sctn-privacy-considerations). While it is possible to store PII on a large blob, this is no different to associating PII to a given credential server-side.

   Each authenticator type might choose to protect this information from offline attackers in posession of a user's device in different ways. CTAP 2.1 does so by:
     * [Protecting reading of the large blob array](https://fidoalliance.org/specs/fido-v2.1-rd-20201208/fido-client-to-authenticator-protocol-v2.1-rd-20201208.html#largeBlobsRW) (the underlying storage) with user verification (i.e. requiring a PIN / fingerprint / etc) when configured.
     * Encrypting each blob with a per-credential key that is only disclosed during registration & assertion. Assertions can be mandated to require user verification through the mandatory "credProtect" extension.

4. How do the features in your specification deal with sensitive information?

   See #3.

5. Do the features in your specification introduce new state for an origin that persists across browsing sessions?

   Yes. The large blob persists across browsing sessions.

6. Do the features in your specification expose information about the underlying platform to origins?

   Somewhat. After creating or asserting a credential, the browser will indicate support of the feature. Support depends on the browser, platform (on Windows, large blob is only supported for versions >= 11), and authenticator. For example, a relying party that knows a user is on Windows can infer that they are on a fairly recent version if large blob registration succeeds.

   This information is not very practical to use for fingerprinting the user since it requires them to go through a registration or assertion ceremony.

7. Does this specification allow an origin to send data to the underlying platform?

   Yes. If the large blob is stored on a platform credential, by definition the platform would store that blob.

8. Do features in this specification enable access to device sensors?

   No.

9. What data do the features in this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

   See #1. Large blobs that were previously stored, and whether they are supported by the given authenticator & platform. This data is new.

10. Do features in this specification enable new script execution/loading mechanisms?

    No.

11. Do features in this specification allow an origin to access other devices?

    WebAuthn by definition exposes an API to use devices for authentication. The large blob extension does not add new devices that can be used.

12. Do features in this specification allow an origin some measure of control over a user agent’s native UI?

    A user agent may choose to inform the user large blob is required for the request, but other than that, no.

13. What temporary identifiers do the features in this specification create or expose to the web?

    None.

14. How does this specification distinguish between behavior in first-party and third-party contexts?

    Large blobs does not change how WebAuthn treats first-party vs third-party contexts. For specific guidance on webauthn & iframes, see [Using Web Authentication within iframe elements](https://w3c.github.io/webauthn/#sctn-iframe-guidance). For a general discussion on credential management, see [Cross domain credential access](https://w3c.github.io/webappsec-credential-management/#security-credential-access) and [Origin confusion](https://w3c.github.io/webappsec-credential-management/#security-origin-confusion).

15. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

    Storing a large blob will leave information that outlasts the incognito session. Chrome already has a warning for incognito mode when storing a discoverable credential (practically speaking, a requirement for large blobs) and tells the user of this fact. Other than that, the feature works the same.

16. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

    [Security Considerations](https://w3c.github.io/webauthn/#sctn-security-considerations) & [Privacy Considerations](https://w3c.github.io/webauthn/#sctn-privacy-considerations)

17. Do features in your specification enable origins to downgrade default security protections?

    No.

18. How does your feature handle non-"fully active" documents?

    No information is shared with the website until the user consents, so that can't happen if the document is not fully active. In practice, a `navigator.credentials.get()` / `navigator.credentials.create()` request will be paused with the promise remaining unresolved, and may be resumed if the document becomes fully active again.