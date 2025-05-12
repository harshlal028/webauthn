# [Self-Review Questionnaire: Security and Privacy](https://w3c.github.io/security-questionnaire/)

This questionnaire pertains to immediate mediation mode for Web Authentication.
* [Explainer](https://github.com/w3c/webauthn/wiki/Explainer:-WebAuthn-immediate-mediation)
* [PR](https://github.com/w3c/webauthn/pull/2291)

01.  What information does this feature expose,
     and for what purposes?

This feature does not explicitly provide any new information to the calling website. However, it does implicitly reveal information that most user agents do not currently expose, because callers can differentiate between UI having been shown to the user and UI not having been shown by timing how long it takes for the returned promise to be resolved.

If UI is shown, the site can infer that the user has at least one eligible credential for sign-in. If UI is not shown, less information is exposed. It could mean that no eligible credentials are available, but there are other reasons why they might not be shown.

The credentials are not named credentials as described in the specification's [privacy section](https://w3c.github.io/webauthn/#sctn-assertion-privacy), so this is not identifying.

02.  Do features in your specification expose the minimum amount of information
     necessary to implement the intended functionality?

Yes.

03.  Do the features in your specification expose personal information,
     personally-identifiable information (PII), or information derived from
     either?

Any information contained within the credential only because available to the caller after the user has provided consent. This is unchanged from the current Web Authentication behaviour.

04.  How do the features in your specification deal with sensitive information?

There is no change to how sensitive information is being handled.

05.  Does data exposed by your specification carry related but distinct
     information that may not be obvious to users?

No.

06.  Do the features in your specification introduce state
     that persists across browsing sessions?

No.

07.  Do the features in your specification expose information about the
     underlying platform to origins?

No.

08.  Does this specification allow an origin to send data to the underlying
     platform?

The user agent's interactions with platform APIs in immediate mediation mode remain the same as in existing WebAuthn interactions.

09.  Do features in this specification enable access to device sensors?

No.

10.  Do features in this specification enable new script execution/loading
     mechanisms?

No.

11.  Do features in this specification allow an origin to access other devices?

No.

12.  Do features in this specification allow an origin some measure of control over
     a user agent's native UI?

Web Authentication requests to obtain assertions are made by origins and cause the user agent to display authentication UI. The UI that would be displayed in this mode might be different from existing modes, but serve the same purpose.

13.  What temporary identifiers do the features in this specification create or
     expose to the web?

None.

14.  How does this specification distinguish between behavior in first-party and
     third-party contexts?

The PR currently disallowed this mode in third-party contexts. Web Authentication APIs can be called from third-party contexts with appropriate permissions policies, but this mode is not available.

15.  How do the features in this specification work in the context of a browser’s
     Private Browsing or Incognito mode?

Immediate mediation is explicitly disallowed in private browsing modes. User agents should behave identically to when no credentials are available.

16.  Does this specification have both "Security Considerations" and "Privacy
     Considerations" sections?

Yes.

17.  Do features in your specification enable origins to downgrade default
     security protections?

No.

18.  What happens when a document that uses your feature is kept alive in BFCache
     (instead of getting destroyed) after navigation, and potentially gets reused
     on future navigations back to the document?

This feature does not change the interaction between active WebAuthn calls and BFCache.

19.  What happens when a document that uses your feature gets disconnected?

This feature does not change the interaction between active WebAuthn calls and the DOM tree.

20.  Does your spec define when and how new kinds of errors should be raised?

All new error modes introduced by immediate mediation result in a `NotAllowedError` DOM exception being thrown.

21.  Does your feature allow sites to learn about the user's use of assistive technology?

No.

22.  What should this questionnaire have asked?

There is a broader description in [the privacy section of the explainer](https://github.com/w3c/webauthn/wiki/Explainer:-WebAuthn-immediate-mediation#privacy-considerations), which suggests additional mitigations to ensure the small amount of information that this mode reveals cannot be accumulated across many calls in order to be useful for user fingerprinting.