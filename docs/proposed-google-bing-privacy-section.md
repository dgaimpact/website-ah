# Proposed "Data from Google and Bing Services" section — FOR REVIEW ONLY

**Status:** Draft, not live. Not added to `privacy.html` or any published page. Do not publish until Steve (and counsel, if he wants that review) approves — this is legal text and none of it was written to be authoritative on my own judgment.

**Why this exists:** Google's OAuth branding verification requires the privacy policy to "fully document how your application interacts with user data" and "thoroughly disclose the manner in which your application accesses, uses, stores, or shares Google user data," and apps using restricted/sensitive scopes must include a Limited Use disclosure. The current privacy policy (v2.0, April 2026) doesn't cover this at all for any of the Google-scoped connections the app requests, or for Bing.

---

## Gap analysis — what the current policy (v2.0) does and doesn't cover

The current policy's only Google-related line is in **Section 5.2 (Service Providers)**: *"...and Google (AI content generation via Gemini)."* That sentence is about Gemini API usage for content generation — a completely different data flow from the OAuth-scoped connections below. It does not mention:

| Gap | Detail |
|---|---|
| **No mention of any OAuth-connected platform** | GBP, YouTube, Search Console, Analytics, or Bing Webmaster Tools are not named anywhere in the policy. |
| **No scope-by-scope explanation** | Doesn't say what data each connection actually grants access to. |
| **No Limited Use disclosure** | Google's API Services User Data Policy requires apps that use restricted/sensitive scopes to disclose compliance with it, including the Limited Use requirements, in the privacy policy. Absent entirely. |
| **No "not used for AI/ML training" statement** | A specific, commonly-expected disclosure for apps with AI-generation features (which this app has) — reviewers may otherwise wonder if Google user data feeds into the same content-generation pipeline described in Section 5.2. Worth stating explicitly that it doesn't (confirmed against the code — GBP/GSC/Bing data is displayed back to the subscriber, not fed into P4 content generation). |
| **No retention/disconnect behavior** | Doesn't explain what happens to this data if a subscriber disconnects a platform. |
| **No distinction between platform-wide and subscriber-specific access** | GBP, YouTube (subscriber flows) are subscriber's own OAuth tokens; the Bing fallback and admin YouTube channel use an AH-owned credential instead. The policy currently makes no such distinction anywhere. |

### A separate, non-privacy-policy finding worth flagging

GBP (`business.manage`) and YouTube (`youtube.upload`, `youtube`) are **restricted scopes** under Google's classification, not just "sensitive." Restricted-scope apps typically need a **CASA (Cloud Application Security Assessment) Tier 2** security review in addition to branding/privacy-policy verification, on a recurring annual basis once the app is verified. This is a separate process from the privacy-policy fix — worth knowing before assuming the privacy-policy change alone clears verification for the GBP/YouTube scopes. Search Console (`webmasters.readonly`) was reclassified non-sensitive by Google in 2024 (already noted in the app's own code comments) and Analytics (`analytics.readonly`) is sensitive but not restricted, so neither needs CASA.

---

## Proposed section text (draft)

Suggested placement: a new section after the existing **Section 5 (How We Disclose Your Information)**, renumbering sections 6 onward — exact placement and numbering is Steve's/counsel's call, not decided here.

> ### Data from Google and Bing Services
>
> Where you connect a Google Business Profile, Google Search Console property, or Google Analytics property, or a Bing Webmaster Tools account, from your Connections page, we access data from that service using the access token you grant us through that service's own consent screen — limited to exactly what you authorize, and nothing more.
>
> - **Google Business Profile** (`business.manage` scope): your business/location profile data, which we use to build local AI-visibility signals and to manage your listing on your behalf, only as directed by you through the platform.
> - **Google Search Console** (`webmasters.readonly` scope, read-only): your site's search performance data (clicks, impressions, queries, ranking position), which we display back to you and use to inform your AI-visibility reporting.
> - **Google Analytics** (`analytics.readonly` scope, read-only): your property's traffic and engagement data, which we display back to you and use to inform your AI-visibility reporting.
> - **Bing Webmaster Tools** (read-only, via your own connection or, as a fallback for accounts that haven't connected their own, a DGA Impact-owned account you've added as a Read Only user): your site's search performance and verification data on Bing, which we display back to you.
>
> **We do not use data obtained through these connections to train any general-purpose AI or machine-learning model.** It is used only to power the specific, visible features described above — displaying your own performance data back to you, or, for Google Business Profile, managing your listing at your direction. Our use and transfer of information received from Google APIs adheres to the [Google API Services User Data Policy](https://developers.google.com/terms/api-services-user-data-policy), including the Limited Use requirements.
>
> This data is encrypted at rest alongside your other account data (see "Data Security" above). You may disconnect any of these connections at any time from your Connections page. Disconnecting revokes our ongoing access — we stop pulling new data from that service — but does not retroactively delete data already displayed to you in past reports or already-published content that was informed by it.

---

## Notes on the drafting itself

- The "we do not use it to train... models" sentence and the retention/disconnect description are both **verified against the actual code** in this session (`app/api/oauth/disconnect/route.ts` only sets `revoked_at`, it does not delete cached report rows or prior content; none of `gbp-api.ts`/`gsc-ga4.ts`/`bing-api.ts` feed their pulled data into the P4 content-generation pipeline — that pipeline reads from VBP/IIP/audit data per the existing policy's Section 3.2/3.3, not from these connections).
- The exact Limited Use disclosure sentence quoted above ("Our use and transfer of information received from Google APIs adheres to...") is the commonly-documented boilerplate Google has historically asked verification applicants to include — I could not pull an exact, freshly-quoted line from Google's current live pages confirming this exact wording verbatim (their public policy page states the *requirement* to disclose Limited Use compliance but does not prescribe the literal sentence). If Google's verification correspondence gives you exact required wording, use that instead of this draft.
- YouTube is deliberately **not** listed as a bullet in the proposed section — the only YouTube OAuth connection in the app (`app/api/admin/youtube/*`) is an admin-only, platform-wide connection for the S133 podcast pipeline, not something any subscriber authorizes about their own data. If Google's OAuth consent screen for this app still requests YouTube scopes as part of the *same* OAuth client subscribers see, it likely needs a mention regardless (even if functionally admin-only) — flagging this as something to confirm rather than deciding it myself.
