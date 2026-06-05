# IRCTC Sprint Feature Specifications — Part B

## Feature Spec 1: Tatkal Virtual Queue and Rate Limiting State Machine

### Problem Statement
The current system locks up entirely during peak Tatkal booking hours (10:00 AM / 11:00 AM IST) due to extreme transaction concurrency. Daily dynamic commuters and emergency travelers face silent connection drops and sudden session terminations without visual feedback, losing highly contested quotas within seconds.

### Current State (from Part A)
As documented in Problem 1 of PROBLEMS.md, the current system allows a rapid, un-throttled torrent of checkout clicks to hit the transactional endpoints simultaneously at 10:00 AM. This mimics a distributed denial-of-service (DDoS) spike, crashing the API gateway, returning raw HTTP 504 Timeouts, and wiping local application memory, leaving the user completely blind to their booking status.

### Proposed Solution
We will implement an architectural token-bucket virtual queue and an adaptive polling rate limiter on the client application. When a user triggers a Tatkal booking request, the UI immediately locks downstream entry actions and switches to a deterministic, high-fidelity visual progress tracker displaying their specific FIFO (First-In, First-Out) position sequence and an estimated processing countdown time.

### Proposed User Flow — Step by Step
1. User logs in early, configures their itinerary parameters, and stands by on the train availability screen.
2. At exactly 10:00:00 AM IST, the user taps the active "Book Now" ticket button.
3. The application intercepts the initial touch payload, disables all secondary click inputs globally, and emits a secure reservation payload.
4. The system serves an explicit virtual queue interface displaying a dynamic positional allocation card: *"Your position in queue: 4,102. Estimated wait: 12 seconds."*
5. A low-overhead WebSocket channel pushes real-time queue reduction heartbeats to the client screen.
6. As the countdown hits zero, the token-bucket gateway clears the user's secure state session and seamlessly transfers them directly into the passenger routing workspace.
7. The reservation workflow completes safely without exposing the user to any intermediate server connection drops.

### Technical Implementation Plan
**System components affected:**
- API Gateway Layer (Kong / NGINX reverse proxy integration)
- Queue Management Worker Cluster (Redis Sorted Sets for state tracking)
- Frontend Checkout Client Shell

**New data requirements:**
- `queue_token`: Ephemeral UUID cluster mapping user state, timestamp, and position vector.
- `estimated_hold_seconds`: Dynamic integer calculating real-time backend drain metrics.

**API changes:**
- `POST /api/v2/booking/queue/enter` - Registers user session payload, initializes a unique FIFO token, and pushes it into the target Redis Sorted Set cache.
- `GET /api/v2/booking/queue/status` - Polling or WebSocket channel streaming real-time positional updates to the client client viewport.

**Frontend changes:**
- Implementation of a global, non-blocking absolute layout queue overlay modal matching full screen bounds.
- State machines to automatically disable global confirmation inputs upon initial request submission.

**Third-party services (if any):**
- AWS ElastiCache for lightning-fast, high-throughput memory-grid persistence clusters.

### Success Metrics
- **HTTP 504/500 System Drop-out Rate:** Drops from ~45% down to less than 0.1% during high-congestion Tatkal spikes.
- **Average Interaction Time-to-Resolution:** Stabilizes to a clear, deterministic tracking window under 30 seconds per ticket window.
- **Erroneous Double-Submission Invocations:** Reduced to absolute zero due to proactive client-side action locking.

### Edge Cases and Constraints
- **Network Drop during Active Queue State:** If the client drops connection for under 30 seconds, the frontend hooks back into local storage, fetches the active `queue_token`, and re-establishes the WebSocket connection without resetting the user's placement position.
- **Graceful Degradation:** If the Redis cluster crashes under unexpected loads, the API layer falls back to an automated client-side staggered poll mechanism (`retry-after` header delays), preventing deep transaction server cascades.

---

### Wireframe Reference
![Queue screen wireframe](../assets/wireframes/tatkal-queue-screen.png)
*Caption: Proposed Tatkal virtual queue screen — mobile view*

---

## Feature Spec 2: Persistent Client-Side Query and Search State Caching

### Problem Statement
The search filters on IRCTC do not reliably process search arrays or persist their active structural parameters when a user moves backward or forward through historical page flows. Elderly and budget-conscious passengers are forced to repeatedly configure travel toggles, introducing heavy friction during high-stress booking timelines.

### Current State (from Part A)
According to Problem 2 in PROBLEMS.md, when a user sets complex search filters (e.g., Sleeper Class, Morning Departure) and navigates away to inspect specific coach routing layouts, hitting the browser's native "Back" navigation completely wipes all active query conditions. This occurs because the application lacks global client-side caching or state serialization.

### Proposed Solution
We will implement an automated client-side query state management layer using synchronized URL query parameters and local browser storage serialization. Filters are applied immediately on the client side using optimized local array sorting, and their active variables are mirrored cleanly within the browser's navigation parameters.

### Proposed User Flow — Step by Step
1. User enters their origin, destination, and target travel date on the home layout dashboard.
2. The page loads the complete initial search result array.
3. The user modifies filters by checking "Sleeper (SL)" and sliding the departure slider to "Morning (06:00 - 12:00)".
4. The application matches the active filters and instantly alters the displayed train cards locally without executing a network refetch.
5. The URL query dynamically appends parameters: `?class=SL&dep=morning`.
6. The user clicks a train card to examine detailed stops and intermediate arrival times.
7. The user taps their hardware or browser "Back" button; the search screen reads the URL query parameters instantly, reconstructs the identical filter parameters, and displays the expected, filtered view.

### Technical Implementation Plan
**System components affected:**
- Frontend Routing Framework (Application Client Router State Engine)
- Search Filtering Web Component Matrix

**New data requirements:**
- `session_search_state`: A JSON schema object containing parameters like `origin`, `destination`, `date`, `filters: { class, departure_time_slots, quotas }`.

**API changes:**
- No backend API enhancements are required since data filtering and array operations shift completely to client-side computing resources.

**Frontend changes:**
- Integration of a reactive state observer hooked directly into the URL route engine.
- Implementation of a local persistence mixin to sync current filtering inputs with `sessionStorage`.

**Third-party services (if any):**
- None.

### Success Metrics
- **Search Parameter Reset Incidents:** Drops to absolute zero across subsequent historical browser navigations.
- **Average Click-to-Search Progression Efficiency:** Improves by over 35% due to the removal of iterative input setup loops.
- **Client Search Data Loading Footprint:** Secondary page request overhead drops because the cache acts as a local source of truth.

### Edge Cases and Constraints
- **Corrupted Local Browser Cache Data:** If values parsed from the URL string are corrupted or contain unmappable formatting parameters, the app defaults to standard initial load states, silently falling back without breaking layout renders.
- **Graceful Degradation:** If `sessionStorage` permissions are locked out due to private or incognito security states, the application relies exclusively on active inline URL route memory tracking.

---

### Wireframe Reference
![Filter state wireframe](../assets/wireframes/search-filter-persistence.png)
*Caption: Proposed persistent search filters layout — desktop dashboard viewport*

---

## Feature Spec 3: Optimistic Berth Preference and Allocation Payload Lock

### Problem Statement
The preferred seat and berth assignment token is routinely dropped when serializing passenger information arrays during checkout. This heavily compromises travel coordination for vulnerable segments, including families traveling with small children, pregnant women, and senior citizens with limited mobility.

### Current State (from Part A)
As analyzed in Problem 3 of PROBLEMS.md, checking a layout preference radial button (e.g., Lower Berth) registers only as a passive metadata indicator rather than a strict structural payload validation lock. During checkout payload generation, these selection keys frequently drop from outbound POST requests, resetting the configuration back to "No Preference".

### Proposed Solution
We will implement an optimistic transaction lock on selected seat preferences. The selected preference token is integrated as a required field within the client's global checkout state engine and checked against strict JSON schemas before payment transitions occur.

### Proposed User Flow — Step by Step
1. The user picks their desired train service tier and proceeds to the passenger roster sheet.
2. The user registers a passenger name and clicks the explicit layout selector dropdown, setting it to "Lower Berth".
3. The UI validates the choice, flashes a clean visual badge indicating a locked structural request state, and saves it instantly to local component memory.
4. The user completes any secondary input blocks and taps "Review Booking Journey".
5. The confirmation review layout pulls data straight from the verified component state array, showing the verified allocation request indicator cleanly on screen.
6. The user initiates checkout payment with full confidence that their selection parameters are correctly attached to the booking ticket metadata payload.

### Technical Implementation Plan
**System components affected:**
- Frontend Passenger Input Sheet Matrix
- Checkout State Serializer Service Engine
- Backend Passenger Validation Processing API

**New data requirements:**
- `passenger_berth_preference_lock`: An explicit string enumeration tracking choices: `['LOWER', 'MIDDLE', 'UPPER', 'SIDE_LOWER', 'SIDE_UPPER']` with a mandatory structural validation flag.

**API changes:**
- `POST /api/v2/booking/passenger/validate` - Expects a strict multi-passenger schema definition requiring explicit validation of preference payload arrays.

**Frontend changes:**
- Reworking selection input dropdowns to use active visual validation states.
- Implementation of an automated verification step that screens outbound network payloads before passing navigation focus to payment pages.

**Third-party services (if any):**
- None.

### Success Metrics
- **Berth Parameter Dropping Fault Rate:** Reduced from ~40% on mobile browsers down to absolute zero.
- **User Checkout Drop-off Rates:** Falls by over 18% because passengers don't have to back out to fix lost information fields.

### Edge Cases and Constraints
- **Quota Over-allocation Interventions:** If a user selects a lower berth preference but zero inventory remains at the server level, a clear inline modal surfaces *before* payment to explain the shortage and offer alternative options, avoiding unexpected auto-assignments.
- **Graceful Degradation:** If the preference check service times out, the interface falls back to a warning alert informing the traveler that their allocation request will be handled as an optimal best-effort system routing.

---

### Wireframe Reference
![Seat lock wireframe](../assets/wireframes/seat-preference-lock.png)
*Caption: Proposed optimistic seat preference state — passenger details checkout*

---

## Feature Spec 4: Proactive Inline Captcha Background Refresh Worker

### Problem Statement
The backend security verification token for visual Captchas expires silently within a tight, unannounced time window. Vulnerable segments, including slower typists, screen-reader users, and meticulous forms-checkers, are hit with sudden verification drops that wipe out their entirely typed passenger data arrays.

### Current State (from Part A)
As documented in Problem 4 of PROBLEMS.md, when a user spends more than 45 seconds carefully filling out passenger profiles or verifying family documentation, the session's background validation token expires. Tapping "Submit" triggers a validation error, reloads the checkout shell, and wipes every text input block clean, causing immense frustration.

### Proposed Solution
We will deploy an asynchronous, non-destructive inline background worker task to monitor token lifetimes. The worker proactively refreshes the security token inline *without* altering active data models or wiping any passenger input fields.

### Proposed User Flow — Step by Step
1. The user navigates to the passenger details screen; the form initializes and launches a secure Captcha asset alongside a 45-second countdown worker thread.
2. The traveler enters their passenger records, verifying names and ticket information.
3. At the 35-second mark, an automated background routine detects that the token is expiring and smoothly fetches a fresh Captcha image in the background.
4. A subtle, non-disruptive visual notification badge updates near the field: *"Security code updated automatically to prevent timeout."*
5. The new Captcha graphic fades into view while preserving all user-entered text fields.
6. The traveler finishes typing, enters the active visual security letters, and clicks continue without experiencing data loss or form rejection.

### Technical Implementation Plan
**System components affected:**
- Captcha Generation Node Service Group
- Client Form Handling Model Shell
- Background Web Worker Token Component

**New data requirements:**
- `captcha_token_expiry_timestamp`: A millisecond timestamp tracking real-time asset validation states.

**API changes:**
- `GET /api/v2/security/captcha/refresh` - Returns a fresh, isolated base64 visual asset string alongside a renewed token hash identifier.

**Frontend changes:**
- Implementation of an isolated Web Worker task thread to handle timing and background network updates out of the main thread.
- Reworking form error boundary wrappers to decouple layout components from core view state fields.

**Third-party services (if any):**
- None.

### Success Metrics
- **Captcha-Induced Validation Wipes:** Complete reduction down to 0% across long-session form entries.
- **Form Completion Success Rates:** Boosted by over 22% due to the removal of repetitive input entry loops.

### Edge Cases and Constraints
- **Network Timeout During Background Refresh:** If the background refresh script fails due to spotty cell service, the form holds the current token and displays a manual fallback option: *"Network slow. Tap here to manually refresh security image."*
- **Graceful Degradation:** If background web workers are restricted by legacy user browsers, the app falls back to a non-destructive inline timer running on the primary application framework thread.

---

### Wireframe Reference
![Captcha worker wireframe](../assets/wireframes/inline-captcha-refresh.png)
*Caption: Proposed background captcha refresh mechanism — checkout view*

---

## Feature Spec 5: Breakpoint Container Isolation for Dynamic Mobile Ads

### Problem Statement
The responsive CSS layout engine does not properly isolate third-party display advertisements on smaller web viewports. These ad banners frequently break past their target container limits, overlapping primary navigation items and causing accidental click redirects.

### Current State (from Part A)
As described in Problem 5 of PROBLEMS.md, when dynamic marketing scripts run on mobile viewports, the lack of strict container constraints allows banners to float over the main navigation menu. Tapping the top-left menu icon registers an accidental click on the underlying ad layer, hijacking the user's focus and redirecting them away from the platform.

### Proposed Solution
We will implement isolated sandbox containers for all marketing assets using strict CSS flexbox rules and strict aspect-ratio bounds. Display ads are locked inside isolated component blocks with lower layering parameters (`z-index`) than any primary functional UI elements, ensuring core interactions are completely un-interceptable.

### Proposed User Flow — Step by Step
1. The traveler opens the IRCTC portal via their native mobile device browser.
2. The core viewport loads structural elements, locking down top navigation links and account settings icons.
3. Third-party advertising networks inject marketing scripts asynchronously into designated asset spaces.
4. The ad banner initializes inside a constrained CSS grid shell that limits width extensions to 100% of its target box.
5. If an advertisement attempts to overflow its boundaries, the parent container cuts it off cleanly using `overflow: hidden` rules.
6. The user taps the menu menu layout icon without any danger of accidental ad clicks.
7. The user updates their booking configurations efficiently, entirely free from viewport hijacking or unwanted redirects.

### Technical Implementation Plan
**System components affected:**
- Global Responsive Stylesheet Ecosystem (CSS Theme Engine)
- Core App Component Master Layout Frame

**New data requirements:**
- None. This is entirely a presentation-layer layout fix.

**API changes:**
- None.

**Frontend changes:**
- Adding strict structural style definitions (`max-width: 100vw`, `overflow: hidden`, `z-index: 1`) onto all injected advertising frames.
- Elevating primary operational interfaces (navigation drawers, header rows) into a protected layering context (`z-index: 9999`).

**Third-party services (if any):**
- Dynamic Ad Network Scripts (Google AdSense / Publicis exchanges).

### Success Metrics
- **Accidental Ad Redirection Metrics:** Drops by over 95% within mobile responsive testing environments.
- **Mobile Task Flow Success Efficiency:** Significant improvement in speed-to-completion metrics for mobile web users.

### Edge Cases and Constraints
- **Malicious Interstitial Banner Script Exploits:** If an aggressive ad script attempts to dynamically alter its DOM properties to override container constraints, a persistent MutationObserver routine checks layout styles and forces them back into compliance.
- **Graceful Degradation:** If an advertising network fails to load assets inside the isolated container framework, the container collapses to `display: none`, freeing up useful vertical scrolling space for the traveler.

---

### Wireframe Reference
![Mobile ad isolation wireframe](../assets/wireframes/mobile-ad-isolation.png)
*Caption: Proposed mobile responsive viewport with isolated ad sandboxing*

---

## Feature Spec 6: User-Friendly PNR Validation Mapping and Auto-Sanitization

### Problem Statement
The PNR data verification dashboard surfaces raw engineering error stack codes when queries contain typing mistakes or trailing whitespace characters. Casual travelers are left with zero context on whether the platform is down, if their ticket is invalid, or if they simply made a typing error.

### Current State (from Part A)
As analyzed in Problem 6 of PROBLEMS.md, entering an eleven-digit string or trailing space character into the PNR lookup box triggers a validation error from the backend engine. The user interface exposes a cryptic error message like `Error Code: ERR_5003 - Null Response Object`, leaving the user confused and stranded without helpful next steps.

### Proposed Solution
We will implement real-time, client-side input data sanitization using regular expressions alongside a user-friendly error translation engine. Input strings are auto-stripped of trailing whitespace, and raw system exceptions are mapped to clear, actionable instructions.

### Proposed User Flow — Step by Step
1. The user navigates to the public PNR Lookup status view.
2. The user types or pastes their target reference tracking sequence into the data input field.
3. If a trailing space or invalid character is included, a client-side regular expression instantly strips it out silently behind the scenes.
4. If an invalid or expired sequence is submitted, the frontend catches the error payload before rendering.
5. Instead of showing engineering stack codes, a clear, friendly notification appears: *"We couldn't find that PNR number. Please check your ticket for a 10-digit number and try again."*
6. A large, obvious action button labeled "Clear and Re-type" helps the user reset the workspace with a single tap.

### Technical Implementation Plan
**System components affected:**
- Public PNR Tracking View Component
- Error Code Translation Mapping Middleware Module

**New data requirements:**
- An explicit error translation dictionary dictionary schema: `mapping_pnr_errors = { 'ERR_5003': 'The entered PNR has expired or does not exist.', 'ERR_1002': 'System handling limit reached. Please retry shortly.' }`.

**API changes:**
- No change to core transactional database endpoints; requires adding standard descriptive exception sub-codes onto backend server responses.

**Frontend changes:**
- Adding real-time client-side regex formatting constraints onto text input fields.
- Implementation of a user-facing error translation block component to intercept and map exception returns.

**Third-party services (if any):**
- None.

### Success Metrics
- **Unresolved Query Abandons:** Decreases by over 40% due to clear, helpful user error messages.
- **Support Volume Reduction:** Lowers secondary customer support requests driven by confusing technical errors.

### Edge Cases and Constraints
- **Unmapped Novel Upstream Error Codes:** If a brand-new backend error occurs that isn't in the translation mapping dictionary, the UI displays a safe, helpful default message: *"Something went wrong while checking your PNR. Please double-check your entry or try again in a few minutes."*
- **Graceful Degradation:** If client javascript engine functions fail to load, the input field behaves like a standard html form, falling back onto server-side data sanitization blocks.