PRD.md
Perpl Companion Card App
Status: Draft — for internal review

1. Overview
Product: Perpl Companion Card App
Platform: iOS and Android
Operator: [EMI name] — licensed EMI, card issuer, and processor
Client: Perpl ApS — investment platform
Card scheme: Mastercard
Currency: DKK only at launch
App Store: Listed under our developer account, separate app per white-label client
The Perpl Companion Card App gives Perpl's investment platform users a spending wallet funded by their uninvested cash balance. Users access the app via a deep link opt-in from within the Perpl investment app. The app is white-labeled under Perpl's brand and operated entirely by us.

2. Personas
2.1 The Perpl Investor

Male, 18–45, based in Denmark
Invests through Perpl, has uninvested cash sitting idle
Low to medium tech savviness — comfortable with apps, not a fintech power user
Core needs: know my balance, spend easily, feel in control of my card
Available to all Perpl users — no minimum balance or eligibility gate


3. Onboarding
3.1 Overview
Onboarding is the highest-risk flow in the product. The primary failure mode is friction or confusion during this sequence. Every step must be clear, fast, and recoverable.
3.2 User Stories
US-001 — Deep link entry
As a Perpl user, I want to tap a link in the Perpl app and be taken directly to the card app onboarding flow, so that I don't have to find and download the app myself.
Acceptance criteria:

Tapping the opt-in link in the Perpl app opens the card app directly on iOS and Android
If the card app is not installed, the user is directed to the App Store / Google Play before being returned to onboarding
The deep link carries a signed JWT as a URL parameter
The app extracts and validates the JWT before proceeding
If the JWT is missing, the user sees an error: "We couldn't verify your identity. Please return to the Perpl app and try again."
If the JWT is expired or invalid, the user sees the same error and onboarding does not proceed
The user can return to the Perpl app and retry — a new JWT is issued on each opt-in attempt


US-002 — JWT validation
As the system, I need to validate the user's identity token against Perpl's API before onboarding proceeds, so that only legitimate Perpl users can access the card app.
Acceptance criteria:

On launch with a JWT parameter, the app calls Perpl's API to validate the token
A valid token returns: name, email, date of birth
These values are stored in the onboarding session for use in subsequent steps
If the API call fails (network error, timeout), the user sees: "Something went wrong. Please check your connection and try again."
If the token is invalid or expired, onboarding aborts — user is shown the error from US-001
JWT validation happens before any onboarding UI is shown to the user


US-003 — MitID verification
As a new user, I want to verify my identity using MitID so that my card can be issued compliantly.
Acceptance criteria:

MitID verification is presented as the first active step in onboarding after JWT validation
Verification is initiated via Signicat using an in-app browser flow
The in-app browser opens within the app — the user does not leave to an external browser
On successful verification, Signicat returns: name, date of birth, CPR number
The returned name and date of birth are cross-referenced against the JWT payload
If there is a mismatch between MitID data and JWT data, onboarding aborts and the user is shown: "We were unable to verify your identity. Please contact support."
If the user cancels MitID verification, they are returned to the start of the MitID step with the option to try again
If MitID verification fails (technical error), the user is shown: "Something went wrong during verification. Please try again." with a retry option
MitID verification is a one-time requirement — returning users are not asked to re-verify


US-004 — Sanctions and PEP screening
As the system, I need to screen every new user against sanctions and PEP lists before activating their card, so that we meet our regulatory obligations.
Acceptance criteria:

Screening runs automatically immediately after successful MitID verification
Screening is based on the user's name as returned by MitID
Screening completes before the user reaches the T&C acceptance step
If screening returns a clear result, onboarding continues without interruption
If screening returns a PEP or sanctions match, onboarding continues to completion but the card is issued in an inactive state
The match is flagged in our back-office for support team review
The user is not told the specific reason for the review
Screening must complete within a reasonable time — if it times out, onboarding is paused and the user is notified: "We're completing a few checks. This may take a moment."


US-005 — T&C acceptance
As a new user, I want to read and accept the EMI's Terms & Conditions so that my card account can be created.
Acceptance criteria:

T&C acceptance is presented after screening completes
The full T&C text is displayed within the app — not linked to an external page
The user must scroll to the bottom of the T&C before the accept button becomes active
Tapping accept records the acceptance with a timestamp and the version of the T&C accepted
If the user declines or exits without accepting, onboarding is paused — the user can return and resume from this step
T&C version and acceptance timestamp are stored against the user record


US-006 — Virtual card issuance
As a new user, I want to receive my virtual Mastercard immediately after completing onboarding, so that I can start spending right away.
Acceptance criteria:

On successful completion of onboarding (JWT valid, MitID verified, screening clear or matched, T&C accepted), a virtual Mastercard is created in our system
If screening returned a clear result, the card is issued in an active state
If screening returned a PEP or sanctions match, the card is issued in an inactive state
The user is shown a success screen confirming their card is ready
If the card is inactive pending review, the success screen shows: "Your card is being reviewed and will be activated shortly. Our team will be in touch if we need anything from you." instead of the standard confirmation
The user can proceed to the home screen regardless of card state
Virtual card details (PAN, CVV, expiry) are available immediately for active cards
The onboarding flow cannot be repeated — a user with an existing account is routed to the standard login flow


4. Home & Balance
4.1 User Stories
US-007 — Balance display
As a user, I want to see my current card balance on the home screen, so that I always know how much I have available to spend.
Acceptance criteria:

The home screen displays the user's current spendable card balance in DKK
Balance is fetched from our ledger on every app open and on pull-to-refresh
Balance is displayed prominently as the primary element on the home screen
If the balance fetch fails, the last known balance is shown with a visual indicator that the figure may be out of date
If no balance has ever been fetched (first open after onboarding), a zero balance is shown with a prompt to top up via the Perpl app
Balance is not shown while the card is in an inactive state — instead a status message is displayed: "Your card is being reviewed and will be activated shortly."


US-008 — Card state visibility
As a user, I want to see the current state of my card on the home screen, so that I know if there are any issues I need to be aware of.
Acceptance criteria:

The home screen shows the current card state: active, inactive (pending review), or frozen
If the card is frozen, a prominent visual indicator is shown with a quick-action to unfreeze
If the card is inactive pending review, the message from US-006 is shown
If the card is active, no state message is shown — the balance and card are the primary focus


5. Transaction History
5.1 User Stories
US-009 — Transaction feed
As a user, I want to see all my transactions in one place, so that I can track my spending.
Acceptance criteria:

A transaction feed is accessible from the home screen
Settled and pending/reserved transactions appear in a single unified feed
Settled transactions are shown in reverse chronological order (most recent first)
Pending/reserved transactions are visually distinguished from settled (e.g. labelled "Pending") and shown above settled transactions
Each transaction shows: merchant name, transaction amount in DKK, and date
Where enriched merchant data is available via Mastercard ISO files, the enriched merchant name and category are shown
Where enriched data is not available, raw transaction data is shown as a fallback
The feed loads the most recent 30 transactions on open — older transactions load on scroll (pagination)
If no transactions exist, an empty state is shown: "No transactions yet. Start spending with your card."


US-010 — Transaction search and filter
As a user, I want to search and filter my transactions, so that I can find specific purchases quickly.
Acceptance criteria:

A search input is available at the top of the transaction feed
Search filters transactions by merchant name in real time as the user types
A filter option allows the user to filter by date range
Filters and search can be combined
If a search or filter returns no results, an empty state is shown: "No transactions match your search."
Clearing search and filters returns the full transaction feed


US-011 — Transaction detail
As a user, I want to tap a transaction to see its full details, so that I can understand exactly what was charged.
Acceptance criteria:

Tapping any transaction in the feed opens a detail view
Detail view shows: merchant name, merchant category, transaction amount, date and time, transaction status (settled / pending)
If enriched merchant data includes a category icon or logo, it is shown in the detail view
A "Something wrong? Contact support" link is shown in the detail view, opening the support email flow


6. Card Management
6.1 User Stories
US-012 — Freeze / unfreeze card
As a user, I want to freeze my card instantly if I'm concerned about unauthorised use, so that no further transactions can be made until I unfreeze it.
Acceptance criteria:

A freeze / unfreeze toggle is accessible from the card management screen and as a quick action from the home screen
Tapping freeze shows a confirmation prompt before freezing: "Are you sure you want to freeze your card? No payments will be accepted while it is frozen."
On confirmation, the card is frozen immediately via our processing API
The home screen updates to reflect the frozen state
A frozen card declines all payment attempts
Tapping unfreeze immediately restores the card to active — no confirmation prompt required
Freeze / unfreeze is not available while the card is in an inactive (pending review) state


US-013 — Reveal card details
As a user, I want to view my card number, CVV, and expiry date securely, so that I can use my card for online purchases.
Acceptance criteria:

Card details (PAN, CVV, expiry) are accessible from the card management screen
Details are hidden by default — the user must tap to reveal
Before revealing, the user must authenticate via biometrics or PIN
Revealed details are shown for 30 seconds before being hidden again automatically
Card details are never shown in plaintext in any screen recording, screenshot, or background state
Card details are fetched from our secure card data API at the point of reveal — never stored locally on the device


US-014 — Report lost or stolen
As a user, I want to report my card lost or stolen so that it is blocked immediately and cannot be used fraudulently.
Acceptance criteria:

A "Report lost or stolen" option is accessible from the card management screen
Tapping it shows a confirmation prompt: "This will permanently block your card. This cannot be undone. Are you sure?"
On confirmation, the card is permanently blocked immediately via our processing API
The user is shown a confirmation screen: "Your card has been blocked. Would you like to order a replacement?"
From this screen the user can proceed directly to the physical card order flow or dismiss and return to the home screen
A blocked card cannot be unblocked — it is permanently cancelled
The lost/stolen report is logged against the user record with a timestamp


US-015 — PIN management
As a user, I want to set, change, and reveal my PIN, so that I can use my card at payment terminals and keep my PIN secure.
Acceptance criteria:

PIN management is accessible from the card management screen
The user can set a PIN if none exists, change an existing PIN, or reveal their current PIN
Before any PIN action, the user must authenticate via biometrics or app PIN
PIN set and change flows require the user to enter the new PIN twice to confirm
PIN must be 4 digits — alphabetic characters are not accepted
If the two entries do not match, the user is shown: "PINs do not match. Please try again."
PIN is set via our secure card API — it is never stored or transmitted in plaintext
PIN reveal shows the current PIN for 15 seconds before hiding automatically


US-016 — Order replacement card
As a user, I want to order a replacement card if my card is lost, damaged, or stolen, so that I can continue spending.
Acceptance criteria:

Replacement card ordering is accessible from the card management screen and from the lost/stolen confirmation screen
The flow follows the same steps as the initial physical card order (see Section 7)
The existing card (if still active) is cancelled when the replacement is ordered
The user is shown the 75 DKK fee before confirming the order
The replacement card order is logged against the user record


US-017 — Close card account
As a user, I want to close my card account if I no longer want to use it, so that I have full control over my financial products.
Acceptance criteria:

A "Close account" option is accessible from the card management screen
Tapping it shows a confirmation prompt explaining: the card will be permanently cancelled, any remaining balance will need to be handled per our terms, and the action cannot be undone
The user must type "CLOSE" to confirm — tapping alone is not sufficient
On confirmation, the card is cancelled via our processing API and the account is marked closed
The user is shown a confirmation screen and then logged out of the app
A closed account cannot be reopened — the user would need to go through full onboarding again
Any remaining balance handling follows our standard terms (not specified in the app — user directed to T&C)


7. Physical Card Ordering & Activation
7.1 User Stories
US-018 — Order physical card
As a user, I want to order a physical Mastercard so that I can use my card at terminals that don't support contactless or digital wallets.
Acceptance criteria:

Physical card ordering is accessible from the card management screen
The order flow shows the fee (75 DKK) prominently before the user confirms
The delivery address is pre-filled from existing records (CPR / MitID data)
The user can edit the delivery address before confirming
Address fields: full name, address line 1, address line 2 (optional), city, postcode
The user confirms the order — a payment of 75 DKK is deducted from their card balance
If the card balance is insufficient to cover the 75 DKK fee, the order is declined: "Insufficient balance to order a physical card. Please top up via the Perpl app."
On successful order, the user is shown: "Your card is on its way. Estimated delivery: 10–14 business days."
The order is logged against the user record with a timestamp and delivery address
Only one physical card order can be in progress at a time — if an order is already pending, the option to order is disabled


US-019 — Activate physical card
As a user, I want to activate my physical card when it arrives so that I can start using it for in-person payments.
Acceptance criteria:

A card activation prompt is shown on the home screen once a physical card order has been placed and the card has been produced (status received from card bureau)
The user taps to activate and is asked to enter the last 4 digits of their physical card number to confirm receipt
On correct entry, the physical card is activated via our processing API
On incorrect entry, the user is shown: "Those digits don't match. Please check your card and try again." — up to 3 attempts allowed
After 3 failed attempts, the user is directed to contact support
On successful activation, the user is shown a confirmation and the activation prompt is dismissed
The virtual card remains active throughout — physical card activation does not affect the virtual card


8. Authentication & Security
8.1 User Stories
US-020 — Returning user login
As a returning user, I want to log in to the app quickly and securely, so that I can access my card without friction.
Acceptance criteria:

On app open, returning users are prompted to authenticate via biometrics (Face ID / fingerprint)
If biometrics are unavailable or fail, the user is prompted for their app-specific PIN
The app-specific PIN is 6 digits
After 5 consecutive incorrect PIN attempts, the account is temporarily locked and the user must recover via email
Biometrics and PIN authentication must complete before any account data is shown


US-021 — Inactivity timeout
As the system, I need to lock the app after a period of inactivity, so that the user's account is protected if they leave the app open.
Acceptance criteria:

The app locks automatically after 20 minutes of inactivity
On lock, all account data is hidden and the authentication screen is shown
The user must re-authenticate via biometrics or PIN to regain access
Background activity (e.g. receiving a push notification) does not reset the inactivity timer
The timer resets on any user interaction within the app


US-022 — Account recovery
As a user who has forgotten their PIN or lost access to their device, I want to recover access to my account, so that I am not permanently locked out.
Acceptance criteria:

A "Forgot PIN" option is shown on the PIN entry screen
Tapping it initiates an email-based recovery flow — a recovery link is sent to the user's registered email address
The recovery link expires after 30 minutes
Following the link allows the user to set a new app PIN
After setting a new PIN, biometrics can be re-enrolled from within the app settings
Recovery links are single-use — following the link a second time shows an error


9. Notifications
9.1 User Stories
US-023 — Push notification delivery
As a user, I want to receive push notifications for important card events, so that I am always aware of activity on my account.
Acceptance criteria:

Push notifications are sent for the following events:

Transaction approved — includes merchant name and amount
Transaction declined — includes reason category (insufficient funds, card frozen, card blocked)
Top-up received — includes amount added
Card frozen by user
Card blocked (lost/stolen reported)
Suspicious activity detected (flagged by our processing system)
Card pending review (onboarding screening match)


Notifications are delivered via APNs (iOS) and FCM (Android)
Notifications are delivered in real time — within 60 seconds of the triggering event
If the app is in the foreground, notifications are shown as in-app banners as well as stored in notification history
Notification delivery does not require the app to be open


US-024 — Notification preferences
As a user, I want to control which notifications I receive, so that I only get alerts that are relevant to me.
Acceptance criteria:

A notification preferences screen is accessible from app settings
Each notification type (transaction approved, transaction declined, top-up received, card frozen, card blocked, suspicious activity, card pending review) has an individual toggle
Toggling off a notification type prevents that type from being sent to the device
Security-critical notifications (card blocked, suspicious activity) cannot be toggled off
Preferences are stored on our server — they persist across devices and app reinstalls
Default state on first launch: all toggleable notifications are on


US-025 — In-app notification history
As a user, I want to see a history of my notifications in the app, so that I can review alerts I may have missed.
Acceptance criteria:

A notification history screen is accessible from the home screen
All notifications sent to the user are stored and displayed in reverse chronological order
Each entry shows: notification type, message, and timestamp
Notifications are retained in history for 90 days
Unread notifications are visually distinguished from read ones
Tapping a notification marks it as read and navigates to the relevant screen where applicable (e.g. tapping a transaction notification opens the transaction detail)
If no notifications exist, an empty state is shown: "No notifications yet."


10. Support
10.1 User Stories
US-026 — Support contact
As a user who needs help, I want to contact support directly from the app, so that I can get assistance without leaving the app to find contact details.
Acceptance criteria:

A support contact option is accessible from the app settings and from the transaction detail screen
Tapping support opens a pre-populated email draft to our support email address
The email draft pre-populates the subject line with the user's account reference to help our team identify the account
The user's email client opens with the draft ready to send — the user can add their message and send
The support email address is displayed visibly so the user can also contact us outside the app


11. GDPR Data Request
11.1 User Stories
US-027 — Personal data request
As a user, I want to request a copy of my personal data, so that I can exercise my rights under GDPR.
Acceptance criteria:

A "Request my data" option is accessible from app settings
Tapping it shows an explanation of what data will be provided and the expected timeframe
The user confirms the request — a request is logged against their account with a timestamp
The user is shown a confirmation: "We've received your request. We'll send your data to your registered email address within 30 days."
The data is delivered to the user's registered email address — not downloaded in-app
Only one active data request can be in progress at a time — if a request is already pending, the option shows the status of the existing request instead


12. Multi-Tenancy
12.1 Overview
The app is built to support multiple white-label clients. Perpl is the first tenant. Each client gets a separate App Store / Google Play listing. Tenant configuration is managed via developer-updated config files per deployment.
12.2 Configurable per tenant
US-028 — Tenant configuration
As a developer onboarding a new white-label client, I want to configure the app's branding and settings via a config file, so that the correct branding is applied without code changes.
Acceptance criteria:

A tenant config file defines the following per client:

App name
Logo (light and dark variants)
Primary and secondary colour values
Support email address
Virtual card design asset
Physical card design asset


All UI elements referencing the app name, logo, and colours read from the config file — no hardcoded values
Changing the config file and redeploying produces a correctly branded app with no further code changes required
The config file is not exposed to end users or accessible via the app at runtime


13. Non-Functional Requirements
Performance

App launch to home screen (authenticated returning user): under 3 seconds on a standard 4G connection
Balance fetch and display: under 2 seconds
Card freeze / unfreeze API response: under 3 seconds — user sees a loading state during this time
Push notification delivery: within 60 seconds of triggering event
Transaction feed initial load: under 2 seconds for the first 30 transactions

Security

All API communication over HTTPS/TLS
Card details (PAN, CVV) never stored locally on device
Card details fetched from secure API at point of reveal only
CPR number handling to be confirmed with compliance — see OQ-8
JWT tokens are single-use and short-lived — expiry window to be agreed with Perpl
Biometric authentication uses device-native APIs (Face ID, Touch ID, Android BiometricPrompt)
App PIN stored using device secure enclave — never in plaintext
App must pass OWASP Mobile Application Security Verification Standard (MASVS) Level 1

Accessibility

Minimum text contrast ratio: 4.5:1 (WCAG AA)
All interactive elements have accessible labels for screen readers
Font sizes respect system accessibility settings
No colour-only indicators — all states conveyed through colour and text/icon

Availability

Core card functions (balance, freeze, transaction history) must be available 99.9% of the time
Graceful degradation — if a non-critical API is unavailable, the affected feature shows an error state without crashing the app


14. Out of Scope
The following are explicitly out of scope for MVP and must not be built:

Top-up initiated from within the card app
Investment balance or Perpl account visibility
ATM cash withdrawals
Multi-currency support
User-configurable spending limits
In-app transaction dispute flow
In-app chat or live support
Web version of the app
MitID re-authentication for returning users
Admin dashboard for tenant configuration
Additional white-label clients beyond Perpl at launch
Annual statements or transaction export
In-app card delivery tracking