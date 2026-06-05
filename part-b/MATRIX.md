# Impact vs Effort Matrix

## The Matrix

|                   | Low Effort                                            | High Effort                                           |
|-------------------|-------------------------------------------------------|-------------------------------------------------------|
| **High Impact**   | **Quick Wins:**<br>• Problem 2: Search Filter Fix<br>• Problem 6: PNR Error Mapping | **Strategic Imperatives:**<br>• Problem 1: Tatkal Virtual Queue<br>• Problem 3: Seat Preference Lock |
| **Low Impact**    | **Fill-ins:**<br>• Problem 4: Inline Captcha Refresh  | **Thankless Tasks:**<br>• Problem 5: Mobile Ad Breakpoint Fix |

## How I Scored Each Dimension

### Impact Scoring (1–5)
- **5 (Critical):** Core booking path issue directly causing transaction failures or complete data loss for millions of daily users.
- **3 (Medium):** Introduces severe workflow friction and forces repetitive tasks, but does not entirely block ticket booking.
- **1 (Low):** Minor visual confusion or validation layout friction outside the main transactional checkout path.

### Effort Scoring (1–5)
- **5 (Critical):** Demands major changes to backend databases, server-level architecture upgrades, or heavy dependency modifications on the core Railway API.
- **3 (Medium):** Requires updating client-side state management frameworks and implementing deep automated form verification logic.
- **1 (Low):** Simple front-end presentation adjustments, basic text data cleanup, or minor CSS styling isolated modifications.

---

## Placement Justifications

### Problem 1: Tatkal Booking Crashes — High Impact / High Effort
- **Impact (5):** Resolves the single largest source of systemic transaction crashes on the platform, directly helping millions of frantic daily commuters.
- **Effort (5):** Requires implementing token-bucket rate limiters at the server level and deploying highly available Redis clustering infrastructure across core transaction nodes.
- **Prioritization Placement:** **Strategic Imperative.** This is a complex engineering task that is absolutely essential to stabilizing the platform's core operational infrastructure.

### Problem 2: Search Filters Reliability — High Impact / Low Effort
- **Impact (4):** Drastically reduces workflow friction for millions of travelers who use filtering parameters to find specific trains every single day.
- **Effort (2):** Requires shifting array sorting logic to the client side and syncing parameters cleanly with standard local browser URL queries.
- **Prioritization Placement:** **Quick Win.** Delivers immense, immediately noticeable user experience value with minimal engineering risk or backend overhead.

### Problem 3: Seat Selection Resets — High Impact / High Effort
- **Impact (4):** Protects vulnerable travel demographics (vulnerable passengers, elderly, families) from losing critical seating arrangements during payment processing.
- **Effort (4):** Requires rewriting core checkout state data models and applying strict validation rule checks against transactional databases during step changes.
- **Prioritization Placement:** **Strategic Imperative.** Essential for processing transactional data accurately, requiring meticulous verification checks across both front-end and back-end codebases.

### Problem 4: Silent Captcha Expiration — Low Impact / Low Effort
- **Impact (2):** Eliminates frustrating data wipe loops for slow typists, though it primarily benefits users who take an extended time to complete form fields.
- **Effort (2):** Implemented cleanly by running an isolated background web worker thread that handles automated token refreshes without altering active data inputs.
- **Prioritization Placement:** **Fill-in.** A straightforward, low-risk optimization that can be integrated smoothly during routine front-end maintenance windows.

### Problem 5: Overlapping Ad Mobile Interstitials — Low Impact / High Effort
- **Impact (3):** Fixes broken touch interaction errors on mobile views, although a significant portion of power users rely on the native mobile app rather than mobile web browsers.
- **Effort (4):** Highly frustrating to maintain long-term due to unpredictable structural code injected by third-party advertising script providers which frequently override standard CSS scoping rules.
- **Prioritization Placement:** **Thankless Task.** Necessary for polishing mobile web usability, but yields lower immediate ROI compared to fixing core ticket purchasing workflows.

### Problem 6: Cryptic PNR Validation Errors — High Impact / Low Effort
- **Impact (4):** Instantly clarifies lookup errors for millions of anxious travelers tracking journey statuses, removing confusion regarding platform health.
- **Effort (1):** Requires basic front-end text sanitation using regex patterns and introducing a simple dictionary lookup to translate raw exception codes into clear text instructions.
- **Prioritization Placement:** **Quick Win.** A tiny development investment that immediately removes massive user confusion outside the transaction path.

---

## Recommended Sprint Order

1. **Problem 2 [Search Filters] & Problem 6 [PNR Error Mapping] (Sprint 1 - Days 1-3):** Fix these "Quick Wins" right away. They deliver immediate UX upgrades with zero risk to back-end transaction engines.
2. **Problem 1 [Tatkal Virtual Queue] (Sprint 2 - Days 4-10):** Allocate dedicated engineering focus to this critical stability issue. Rewriting the gateway routing layer is essential to handle peak traffic waves smoothly.
3. **Problem 3 [Seat Preference Lock] (Sprint 3 - Days 11-14):** Update checkout state schemas to guarantee that seating parameters travel through payment systems accurately without dropping out of transaction payloads.
4. **Problem 4 [Captcha Refresh] & Problem 5 [Mobile Ads Container] (Sprint 4 - Days 15-17):** Wrap up the sprint cycle by rolling out asynchronous form validation workers and clean CSS containment layout improvements.