## [2.1 What information does this feature expose, and for what purposes?](https://www.w3.org/TR/security-privacy-questionnaire/##purpose)

This feature does not expose any new information.

## [2.2 Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?](https://www.w3.org/TR/security-privacy-questionnaire/##minimum-data)

No

## [2.3 Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?](https://www.w3.org/TR/security-privacy-questionnaire/##personal-data)

No, this feature does not expose personal information.

## [2.4 How do the features in your specification deal with sensitive information?](https://www.w3.org/TR/security-privacy-questionnaire/##sensitive-data)

No, this feature does not deal with sensitive information.

## [2.5 Does data exposed by your specification carry related but distinct information that may not be obvious to users?](https://www.w3.org/TR/security-privacy-questionnaire/##hidden-data)

This feature allows a WebAuthn credential to be used with a different origin than it was originally created for. WebAuthn client display the calling origin to the user during credential selection.

## [2.6 Do the features in your specification introduce state that persists across browsing sessions?](https://www.w3.org/TR/security-privacy-questionnaire/##persistent-origin-specific-state)

No, this feature does not introduce any new persistent state.

## [2.7 Do the features in your specification expose information about the underlying platform to origins?](https://www.w3.org/TR/security-privacy-questionnaire/##underlying-platform-data)

No, this feature does not expose information about the underlying platform to origins.

## [2.8 Does this specification allow an origin to send data to the underlying platform?](https://www.w3.org/TR/security-privacy-questionnaire/##send-to-platform)

No, this feature does not allow an origin to send any new data to the underlying platform.

## [2.9 Do features in this specification enable access to device sensors?](https://www.w3.org/TR/security-privacy-questionnaire/##sensor-data)

No, this feature does not enable access to device sensors.

## [2.10 Do features in this specification enable new script execution/loading mechanisms?](https://www.w3.org/TR/security-privacy-questionnaire/##string-to-script)

No, this feature does not enable new script execution/loading mechanisms.

## [2.11 Do features in this specification allow an origin to access other devices?](https://www.w3.org/TR/security-privacy-questionnaire/##remote-device)

No, this feature does not enable allow an origin to access other devices.

## [2.12 Do features in this specification allow an origin some measure of control over a user agent’s native UI?](https://www.w3.org/TR/security-privacy-questionnaire/##native-ui)

No, this feature does not allow an origin some measure of control over a user agent’s native UI. 

## [2.13 What temporary identifiers do the features in this specification create or expose to the web?](https://www.w3.org/TR/security-privacy-questionnaire/##temporary-id)

No, this feature does not create or expose any temporary identifiers to the web.

## [2.14 How does this specification distinguish between behavior in first-party and third-party contexts?](https://www.w3.org/TR/security-privacy-questionnaire/##first-third-party)

This feature allows a WebAuthn Relying Party to declare that a WebAuthn credential created for their RP IP, can be used in a limited set of third party contexts. The behavior is controlled by the user agent / WebAuthn client.

## [2.15 How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?](https://www.w3.org/TR/security-privacy-questionnaire/##private-browsing)

This feature works the same in Private Browsing / Incognito modes.

## [2.16 Does this specification have both "Security Considerations" and "Privacy Considerations" sections?](https://www.w3.org/TR/security-privacy-questionnaire/##considerations)

Yes, the WebAuthn specification has these sections.

## [2.17 Do features in your specification enable origins to downgrade default security protections?](https://www.w3.org/TR/security-privacy-questionnaire/##relaxed-sop)

No, this feature does not enable origins to downgrade default security protections.

## [2.18 What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?](https://www.w3.org/TR/security-privacy-questionnaire/##bfcache)

There is no change in behavior for this feature.

## [2.19 What happens when a document that uses your feature gets disconnected?](https://www.w3.org/TR/security-privacy-questionnaire/##non-fully-active)

There is no change in behavior for this feature.

## [2.20 Does your spec define when and how new kinds of errors should be raised?](https://www.w3.org/TR/security-privacy-questionnaire/##error-handling)

There are no new errors or error conditions defined by this feature.

## [2.21 Does your feature allow sites to learn about the user’s use of assistive technology?](https://www.w3.org/TR/security-privacy-questionnaire/##accessibility-devices)

No, this feature does not allow sites to learn about the user’s use of assistive technology.

## [2.22 What should this questionnaire have asked?](https://www.w3.org/TR/security-privacy-questionnaire/##missing-questions)

n/a