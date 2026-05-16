VISION.md
Perpl Companion Card App
Status: Draft — awaiting internal review and client sign-off

1. The Problem
Perpl's investment platform users accumulate uninvested cash in their accounts. Accessing that cash today requires withdrawing to a bank account first — an extra step that creates friction and distances the user from their money.
There is no direct path from "I have cash sitting in my Perpl account" to "I can spend it now."

2. The Product
A co-branded companion debit card app for Perpl's users, built and operated by us as the EMI on our core processing and ledger infrastructure.
The app gives Perpl users a spending wallet funded by their uninvested Perpl cash balance — no bank withdrawal required. It is a standalone mobile app (iOS and Android), white-labeled under Perpl's brand, listed on the App Store and Google Play under our developer account.
The UX benchmark is apps like Revolut, Lunar, and Wise — clean, fast, and intuitive — but intentionally narrower in scope. This is a card programme, not a banking app. No investment features, no savings products, no banking integrations.

3. Who It's For
Primary persona: The Perpl Investor

Male, 18–45, based in Denmark
Uses Perpl to invest but has uninvested cash sitting idle
Low to medium tech savviness — comfortable with apps but not a fintech power user
Wants simplicity: know my balance, spend easily, feel in control of my card
Available to all Perpl users — no minimum balance or eligibility gate


4. Why Now
Perpl wants to deepen engagement with their user base by making their platform a more complete financial tool. A companion spending card converts idle cash into daily utility — increasing platform stickiness and user lifetime value.
For us, this is the first live deployment of our Card-as-a-Service offering. The architecture must support multiple white-label clients (tenants) from day one, even though Perpl is the only client at launch.

5. Core User Flows
5.1 Onboarding

User opts in to the card app from within the Perpl investment app
Deep link opens the companion card app (iOS / Android)
Signed JWT passed as deep link parameter — our app validates the token against Perpl's API
JWT payload contains: name, email, date of birth
If token is expired or invalid, onboarding aborts — user can retry
User completes MitID verification via Signicat (in-app browser flow) — one-time requirement
MitID returns: name, date of birth, CPR number
Automated sanctions and PEP screening runs against user's name
PEP match: card issued in inactive state — support team reviews queue — user notified without specific reason cited
Sanctions match: (see Open Questions — OQ-6)
User accepts EMI Terms & Conditions
Virtual Mastercard issued instantly
User optionally orders a physical Mastercard (75 DKK, custom Perpl brand design)

5.2 Spending & Balance

User tops up the card from within the Perpl investment app — not the card app
Funds locked on Perpl's side, then batch-transferred to our ledger
Card app displays current spendable card balance sourced from our ledger
Spending-only card — no ATM cash withdrawals
DKK only at launch
Accepted wherever Mastercard is accepted, including Apple Pay and Google Pay

5.3 Card Management

Freeze / unfreeze the card
View and reveal card details (PAN, CVV, expiry) for online payments
Report card lost or stolen — card blocked instantly, user then chooses whether to order a replacement
PIN management (set, change, reveal)
Order a replacement card
Close / cancel the card
Physical card in-app activation (required before physical card can be used)
GDPR data request


Note: Spending limits are programme-level controls set by us in the backend. They are not user-configurable and do not appear in the card management UI.

5.4 Physical Card Ordering

Fixed fee: 75 DKK
Delivery address pre-filled from existing records (CPR / MitID data), user can edit
Estimated delivery timeframe shown post-order — no in-app tracking
Non-delivery handled by support — no in-app flow
Custom Perpl brand design on card
Explicit in-app activation step required before the physical card works

5.5 Transaction History

Unified transaction feed — settled and pending/reserved transactions in one list
Pending/reserved transactions visually distinguished, shown below settled
Enriched merchant data (name, category) via Mastercard ISO files — raw data as fallback
Filterable and searchable
No in-app dispute flow — transaction queries handled by support via email

5.6 Returning User Authentication

Primary: biometrics (Face ID / fingerprint)
Fallback: app-specific PIN / passcode
20 minute inactivity timeout — app locks and requires re-authentication
Account recovery: email + forgot password flow

5.7 Notifications

Push notifications for: every transaction, declined payments, top-up received, security events (card frozen, suspicious activity, card blocked)
Every notification type individually toggleable by the user
In-app notification history
iOS (APNs) and Android (FCM) from day one
Push only — no SMS fallback
Push notification provider: to be selected in Navigate

5.8 Support

In-app access to our support email address
All card support handled by us — not Perpl
Transaction disputes, non-delivery of physical card, and account queries all route to support email


6. Multi-Tenancy
The app is architected for multiple white-label clients from day one. Perpl is the first tenant.
Configurable per tenant:

Logo
Colour scheme
App name
Support email address
Card design (virtual and physical)

Deployment model:

Each client gets a separate App Store / Google Play listing
Tenant configuration managed via developer-updated config files — no admin dashboard
All clients use the same underlying infrastructure and codebase


7. What Success Looks Like
At 6 months post-launch, success is measured by:

Card activation rate — percentage of Perpl users who opt in and complete onboarding
Number of activated cards — absolute volume of active cardholders
Transaction volume — total spend processed through the card

The primary risk to all three metrics is a difficult or messy onboarding experience. Onboarding completion rate is the leading indicator — a drop here predicts failure across the board.

8. MVP Scope
In scope for MVP:

Full onboarding flow — deep link → JWT validation → MitID (Signicat) → sanctions/PEP screening → T&C acceptance → virtual card issued
Card balance display (DKK)
Transaction history — unified feed, enriched merchant data, filterable and searchable
Card management — freeze/unfreeze, reveal card details, lost/stolen reporting, PIN management, replacement card ordering, card cancellation, physical card activation, GDPR data request
Physical Mastercard ordering — 75 DKK, custom Perpl brand, estimated delivery timeframe
Apple Pay and Google Pay support
Push notifications — per-type toggles, in-app notification history
Returning user authentication — biometrics, PIN fallback, 20 min timeout, email recovery
Perpl white-label branding — logo, colours, app name, support email, card design
Multi-tenant architecture — Perpl as first tenant, config-file based per deployment
Support via in-app email contact

Explicitly out of scope for MVP:

Top-up initiated from within the card app — lives in Perpl's app
Investment balance visibility inside the card app
ATM cash withdrawals — spending only
Multi-currency — DKK only at launch
User-configurable spending limits — programme-level only
In-app transaction dispute flow — support via email
In-app chat or live support
MitID re-authentication for returning users — deferred due to cost
Admin dashboard for tenant configuration
Additional white-label clients beyond Perpl at launch


9. Non-Goals

This app does not replace or extend Perpl's investment platform
This app does not display or interact with investment positions or returns
Customer support does not route through Perpl — all card queries come to us
This app does not provide banking services — it is a card programme only


10. Open Questions
#QuestionOwnerBlocking?OQ-6Sanctions match handling — same inactive card + review queue flow as PEP, or immediate hard block with no card issued?ComplianceYes — blocks onboarding spec in GroundOQ-7PEP/sanctions review — what does the user-facing notification say while their card is pending review? Needs exact copy or approved templateUs (product + compliance)No — needed before Ground is completeOQ-8CPR data handling — is CPR number retained post-verification or discarded after KYC check? Who has access? Needs compliance sign-off given GDPR sensitivityCompliance / LegalNo — needed before NavigateOQ-9MitID re-authentication for high-risk actions (e.g. revealing PAN/CVV, changing PIN) — is biometric sufficient or do we need step-up authentication?Compliance / ProductNo — needed before Ground is completeOQ-10Estimated delivery timeframe for physical card — what do we tell the user? (e.g. 5–7 business days)Us / BureauNo

11. Confidence Notes
Confirmed in session:

Funding model — uninvested cash only, top-up from Perpl app, batch transfer to our ledger
Onboarding sequence, JWT mechanism, and MitID/Signicat integration
Mastercard as card scheme, custom Perpl brand design
Card management feature set — with spending limits correctly removed from user-facing scope
Physical card flow — 75 DKK, address pre-fill, estimated timeframe, in-app activation
Transaction history — unified feed, enriched data, no dispute flow
Returning user auth — biometrics, PIN fallback, 20 min timeout, email recovery
Notification requirements — push only, per-type toggles, in-app history
Persona — Danish, male-skewed, 18–45, low-medium tech savviness
Platform — iOS and Android, no web
Multi-tenant architecture — separate app per client, config-file based
Apple Pay and Google Pay in scope for MVP
GDPR data request in scope for MVP
DKK only, spending only (no ATM)
Support via email, handled by us
App Store listed under our developer account
Success metrics and primary failure mode (onboarding friction)

Assumptions made — confirm before Ground:

CPR number is used for KYC matching and identity confirmation — retention policy to be confirmed with compliance (OQ-8)
Biometric authentication is sufficient for sensitive card actions (PAN reveal, PIN change) — step-up auth not required unless compliance mandates it (OQ-9)
Sanctions match results in a hard block — not the same as PEP review flow (OQ-6, unconfirmed)
Push notification provider will be selected during Navigate — Firebase FCM assumed as default candidate