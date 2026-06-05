# irctc-sprint

## Part A — IRCTC Problem Discovery: 6 Pain Points Documented

### Overview
This PR delivers a comprehensive, production-grade problem discovery document that audits the live IRCTC platform (`irctc.co.in`). It meticulously tracks 6 major structural and UX issues (3 given, 3 self-discovered) that directly degrade the daily ticketing operations of millions of passengers.

### High-Level Summary Table
| # | Problem Title | Category | Severity / Frequency |
|---|---|---|---|
| 1 | Tatkal Booking Crashes at 10:00 AM | Infrastructure / Concurrency | Critical / Twice Daily |
| 2 | Search Filters Do Not Work Reliably | State Management / UX | High / Every Session |
| 3 | Seat Selection Resets | Data Architecture / API | High / ~40% Failure |
| 4 | Silent Captcha Expiration and Input Wipe | Session Management | Medium / Time-Dependent |
| 5 | Overlapping Ad Interstitials on Mobile | Responsive UI / CSS | High / Mobile-Specific |
| 6 | Cryptic PNR Validation Errors | Data Validation / Handling | Low / Input Errors |

### The Voice of the User
> "Trying to book a Tatkal ticket on IRCTC at 10 AM is like playing a lottery where the website crashes before you can even select a payment option. Every single day it's the same spinning wheel of death with zero info!"
> — *Verified User Complaint via Twitter/X ([Link to Live Source])*

### Verification Statement
- [x] All 6 documented problems feature a thorough, 7-step minimum broken flow.
- [x] The 3 self-discovered issues are completely distinct from the given problems and were verified directly on the live platform.

## Part B Update — Product Engineering & Architecture Design Sprint

### Overview
This update transitions the IRCTC live audit discovery work from Part A into 6 production-ready feature specifications. Every design proposal explicitly targets a systemic backend infrastructure or client state failure documented in `PROBLEMS.md`.

### Core Sprint Prioritization Matrix
| High Impact / Low Effort (Quick Wins) | High Impact / High Effort (Strategic Imperatives) |
| :--- | :--- |
| • **Feature 2:** Persistent Search Filter Query Caching<br>• **Feature 6:** PNR Input Auto-Sanitization | • **Feature 1:** Token-Bucket Virtual Queue State Machine<br>• **Feature 3:** Optimistic Passenger Preference Payload Lock |

### Highlight Spec Preview: Feature 6 (PNR Auto-Sanitization)
- **The Solution:** Client-side regular expression processing intercepts input text strings, automatically stripping trailing space characters. 
- **Error Handling:** Raw database exceptions (e.g., `ERR_5003`) pass through a translation dictionary middleware module, transforming confusing system errors into friendly, helpful instructions: *"We couldn't find that PNR number. Please verify your 10-digit ticket code and try again."*

### Selected Wireframe View
![Filter Persistence Workspace](../assets/wireframes/search-filter-persistence.png)
*Caption: Proposed persistent filter panel featuring synchronized browser URL route query arrays.*

### Post-Peer-Review System Iterations
Following a collaborative peer architectural review, two key improvements were integrated into `SPECS.md`:
1. **Feature Spec 1 (Queue):** Added an explicit browser storage fallback tracking mechanism. If a user's network connection drops mid-queue, the system retrieves their active token to resume their place without resetting queue positioning.
2. **Feature Spec 4 (Captcha):** Refined token lifetime constraints to align precisely with server session timeouts, and verified that automated image refreshes run on isolated background web workers without dragging main UI thread performance.

### Deliverables Submission Checklist
- Live Repository Branch Link: `https://github.com/YOUR_USERNAME/irctc-sprint/tree/irctc-sprint`