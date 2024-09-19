1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

The signal methods by design pass information of the state of the credentials from sites to credential providers (authenticators). This is the point of the API: letting sites keep the state of the credentials (whether they are valid, and what user.name and user.displayName they should have) in sync with the credential provider.

Signal methods do not return any information to the site, so they cannot be used to e.g. figure out if a user has a credential with a given site.

2. Do features in your specification expose the minimum amount of information necessary to enable their intended uses?
Features should only expose information when it’s absolutely necessary to satisfy use cases. If a feature exposes more information than is necessary, why does it do so?

The only information passed from the site to credential providers is: the list of valid credential IDs for the user, the user's `name` and `displayName`, and the relying party identifier (roughly its origin). This information is required to keep credentials in sync, and is already shared when the user creates a credential.

3. How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?

WebAuthn credentials can be created with a user.name and user.displayName set of properties to label credentials. These can be arbitrary strings and are already passed when creating credentials. The signal methods merely provide a way to keep them updated.

4. How do the features in your specification deal with sensitive information?

N/A

5. Do the features in your specification introduce new state for an origin that persists across browsing sessions?
There are many existing mechanisms origins can use to store information about a user. Cookies, ETag, Last Modified, [localStorage](https://html.spec.whatwg.org/multipage/webstorage.html#dom-localstorage), and [indexedDB](https://w3c.github.io/IndexedDB/#dom-windoworworkerglobalscope-indexeddb) are just a few examples.

The `name` and `displayName` properties that can be updated are already part of the credential state that persists across browsing sessions. No state is ever revealed to the site unless the user consents to an authentication ceremony.

6. Do the features in your specification expose information about the underlying platform to origins?
(Underlying platform information includes user configuration data, the presence and attributes of hardware I/O devices such as sensors, and the availability and behavior of various software features.)

No.

7. Does this specification allow an origin to send data to the underlying platform?

Yes.
* The list of valid credential IDs. These are already passed down when doing an authentication ceremony.
* The user name and user display name. These are already passed down when making a new credential.

8. Do features in this specification enable access to device sensors?

No.

9. Do features in this specification enable new script execution/loading mechanisms?

No.

10. Do features in this specification allow an origin to access other devices?

No, or at least not for any device that cannot already be used as a WebAuthn authenticator today.

11. Do features in this specification allow an origin some measure of control over a user agent’s native UI?

User agents can choose whether they notify their user about changes from the signal methods. For Chrome specifically, this results in a small bubble on the top right where the only text controlled by the site is the origin.

12. What temporary identifiers do the features in this specification create or expose to the web?

None.

13. How does this specification distinguish between behavior in first-party and third-party contexts?

This feature can be used in third-party contexts. It does not provide any information to the third party.

14. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?
Most browsers implement a private browsing or incognito mode, though they vary significantly in what functionality they provide and how that protection is described to users [[WU-PRIVATE-BROWSING]](https://www.w3.org/TR/security-privacy-questionnaire/#biblio-wu-private-browsing).

No information is sent back to the site, so if a browser decided to implement something different for incognito mode, it would not let a site identify it.

15. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?
Specifications should have both "Security Considerations" and "Privacy Considerations" sections to help implementers and web developers understand the risks that a feature presents and to ensure that adequate mitigations are in place. While your answers to the questions in this document will inform your writing of those sections, do not merely copy this questionnaire into those sections. Instead, craft language specific to your specification that will be helpful to implementers and web developers.

WebAuthn has extensive [security](https://w3c.github.io/webauthn/#sctn-security-considerations) and [privacy](https://w3c.github.io/webauthn/#sctn-privacy-considerations) considerations.

16. Do features in your specification enable origins to downgrade default security protections?

No.

17. How does your feature handle non-"fully active" documents?

The request is dropped. But this is fine: the signal methods are provided in a best effort basis.