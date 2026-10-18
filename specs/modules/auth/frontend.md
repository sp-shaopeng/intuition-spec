# InTuition — Auth Module — Frontend Specification

> Reading note: [Current requirements](../../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 requirements alignment — docs only.** Anonymous filtering/limited request browse is permitted; detail/action authentication follows the current baseline.001/002/M19 supersede the old signup outline;003/004/012 own reset, refresh and contact-change user interface. Historical multi-factor authentication/password examples are not independent policy.
> Existing status/code/evidence statements below describe their original revision; no runtime implementation or deployment is certified by this correction.

**Module:** auth  
**Version:** 1.1.0  
**Stack:** React 18 + TypeScript (Vite), React Hook Form, Zod, Tailwind CSS  
**Last Updated:** 2026-05-27  
**Platform:** InTuition  
**Tech Stack:** React 18 + TypeScript (Vite) / Java 21 + Spring Boot 3.x

---

## 0. Public Browse vs Login Gate Principle

**Unauthenticated users (no session)** may access the following pages freely:
- / — Landing page
- /tutors — Browse approved tutor listings (read-only)
- /requests — Limited public request browsing; anonymous filters are allowed for both browse pools.
- Tutor/request detail requires authentication under the current browse/detail contract; public cards remain personal information-free.
- /rates — Tuition rate guide (public reference)
- /register, /login, /forgot-password, /reset-password

**Login is required** for any action that creates, modifies, or responds to content:
- Post a tuition request (tutee)
- Create a tutor listing
- Request an interview with a tutor
- Full tutor/request detail and protected actions; limited public request browsing/filtering remains available without login
- Access any dashboard or profile page

**Implementation:** Every gated CTA (e g., "Post a Request", "Request Interview", "List as a Tutor") must redirect unauthenticated users to /login?return To=<current-path>. After successful login, the user is returned to the original page with intent preserved.

**Forgot password is Priority P0.** The /forgot-password and /reset-password pages are launch-blocking. Users who cannot recover a lost password will be permanently locked out. These pages must be implemented before any production release.

---

## 1. Route Map

| Route | Component | Auth Required | Role Guard |
|-------|-----------|---------------|------------|
| /register | Register Role Select Page | No | Redirect to dashboard if already authed |
| /register/tutee | Register Tutee Page | No | Redirect if authed |
| /register/tutor | Register Tutor Wizard Page | No | Redirect if authed |
| /login | Login Page | No | Redirect if authed |
| /login/admin | Admin Login Page | No | Redirect if authed |
| /login/admin/mfa | Admin Mfa Page | Partial (mfa pending cookie) | ADMIN only path |
| /forgot-password | Forgot Password Page | No | — |
| /reset-password | Reset Password Page | No | Requires ?token query param flow |
| /account/delete | Account Delete Page | Yes | TUTEE, TUTOR |

All auth pages share the Auth Layout (centred card, agency logo, no nav sidebar).

---

## 2. Shared Components

### Auth Layout
- Full-height centred layout, white card on neutral background.
- Agency logo at top of card.
- No navigation sidebar.
- children prop for page content.

### Password Input
- Text input with show/hide toggle (eye icon button).
- ARIA label: "Show password" / "Hide password".
- Inherits all standard input validation states.

### Form Error
- Red inline message beneath a field for field-level errors.
- Driven by react-hook-form field State error.message.

### Alert Banner
- Full-width banner inside card for form-level errors (e g. "Invalid credentials").
- Variants: error (red), success (green), info (blue).

### Loading Button
- <button> that shows a spinner and is disabled while is Submitting is true.
- Prevents double-submission.

### Step Indicator
- Visual step tracker for multi-step wizard (1 → 2 → 3).
- Current step highlighted; completed steps show checkmark.

---

## 3. Page Specifications

---

### 3.1 /register — Role Selection

**Purpose:** Entry point; user chooses Tutee or Tutor before proceeding.

**Layout:** Two large clickable cards side by side.

**Cards:**
- "I am a Tutee (looking for a tutor)" → navigates to /register/tutee
- "I am a Tutor (offering tuition)" → navigates to /register/tutor

**Footer:** "Already have an account? [Private supporting material excluded (publication note)](../../../docs/publication-notes.md)"

**Redirect logic:** If a valid auth cookie is present (middleware check), redirect to role-appropriate dashboard before rendering.

---

### 3.2 /register/tutee — Tutee Registration

**Form fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| full Name | text | Yes | Display name |
| email | email | Yes | — |
| phone | tel | No | Singapore format |
| password | password | Yes | See strength rules |
| confirm Password | password | Yes | Must match password |
| agree To Terms | checkbox | Yes | Link to T&C and Privacy Policy |

**Zod validation (prose):**

- full Name: non-empty string, min 2 characters, max 100 characters.
- email: valid email format (Zod the email operation), max 255 characters.
- phone: optional; if provided, must match Singapore mobile pattern a Singapore mobile number with an optional country code — accepts +65 prefix or bare 8-digit number starting with 8 or 9.
- password: min 8 characters, max 128 characters; must contain at least one uppercase letter (/ A-Z /), one lowercase letter (/ a-z /), one digit (/ 0-9 /).
- confirm Password: must equal password using Zod the refine operation with cross-field check.
- agree To Terms: must be yes.

**Error states:**

- full Name empty → "Full name is required"
- email invalid format → "Enter a valid email address"
- email already registered (application programming interface 409) → Alert Banner: "An account with this email already exists. [Private supporting material excluded (publication note)](../../../docs/publication-notes.md)?"
- phone invalid format → "Enter a valid Singapore mobile number (e g. [REDACTED PHONE])"
- password too short → "Password must be at least 8 characters"
- password missing complexity → "Password must include uppercase, lowercase, and a number"
- confirm Password mismatch → "Passwords do not match"
- agree To Terms unchecked → "You must agree to the Terms & Conditions"
- Generic application programming interface error (500) → Alert Banner: "Something went wrong. Please try again."

**Loading state:** Loading Button shows spinner; all fields disabled.

**Success:** signed session token cookie set by server; client redirects to /dashboard.

**Footer:** "Registering as a Tutor? [Private supporting material excluded (publication note)](../../../docs/publication-notes.md)"

---

### 3.3 /register/tutor — Tutor Registration Wizard

Multi-step wizard. State persisted in React state; on page refresh, user returns to Step 1 (session Storage backup is a progressive enhancement only).

**Step Indicator:** 1 (Account Info) → 2 (Commission Disclosure) → 3 (Tutor Profile)

---

#### Step 1 — Account Info

Identical fields to Tutee registration with one difference:

- phone: **required** for tutors (Singapore format, same regex).

Additional validation: phone is mandatory → "Phone number is required for tutor accounts".

**Navigation:** "Continue →" button (validates Step 1 fields before proceeding; does NOT call application programming interface). On validation success, advance to Step 2.

---

#### Step 2 — Commission Disclosure

**Layout:** This is a standalone disclosure page within the wizard. No form fields other than a single checkbox.

**Content block:** Renders commission terms fetched from config commission-text interface operation (or static config). Displayed in a scrollable card with a clear heading: "Commission Structure".

**Fields:**

| Field | Type | Required |
|-------|------|----------|
| commission Acknowledged | checkbox | Yes |

**Checkbox label:** "I have read and understood the commission structure above. I agree that the agency’s registered legal name will charge the commission fee as described." (legal name sourced from COMPANY legal Name in company implementation reference — do not hard-code).

**Validation:** commission Acknowledged must be yes → "You must acknowledge the commission structure to continue."

**user experience rules:**
- "Continue →" button is disabled until checkbox is ticked (CSS pointer-events none + aria-disabled).
- Cannot navigate back to Step 1 and skip this step by manipulating URL (wizard enforces forward-only guard in component state).

**Navigation:** "← Back" returns to Step 1. "Continue →" advances to Step 3.

---

#### Step 3 — Tutor Profile

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| subjects | multi-select | Yes | Min 1 |
| levels | multi-select | Yes | Min 1 |
| availability | textarea | No | Free text, max 500 chars |
| preferred Regions | multi-select | No | Singapore planning areas |

**Subjects options (non-exhaustive, from config):** Mathematics, English, Chinese, Malay, Tamil, Sciences, Humanities, Economics, Accounting, GP, Mother Tongue, Music, Art, etc.

**MOE Levels options:**
- Primary 1–6
- Secondary 1–5 (Express / N-Level)
- JC1–JC2 / IP
- IB
- Polytechnic
- Adult / Continuing Education

**Validation:**
- subjects: array, min length 1 → "Select at least one subject"
- levels: array, min length 1 → "Select at least one level"
- availability: optional string, max 500 characters → "Availability description is too long"

**Navigation:** "← Back" returns to Step 2. "Submit Registration" calls application programming interface.

**On submit:**
1. Loading Button shows "Creating your account…".
2. submit operation for auth register tutor with full wizard payload.
3. Success → signed session token cookie set; redirect to /dashboard/tutor with toast "Welcome! Your tutor account is ready."
4. 409 Conflict → Alert Banner: "An account with this email already exists."
5. 422 Unprocessable → field-level errors surfaced from application programming interface response.
6. 500 → Alert Banner: "Something went wrong. Please try again."

---

### 3.4 /login — Standard Login

**Form fields:**

| Field | Type | Required |
|-------|------|----------|
| email | email | Yes |
| password | password | Yes |

**Zod validation:**
- email: valid email format.
- password: non-empty string.

**Error states:**

- email empty → "Email is required"
- password empty → "Password is required"
- 401 Unauthorized → Alert Banner: "Invalid email or password"
- 423 Locked → Alert Banner: "Account is temporarily locked. Try again at retry After time formatted in Singapore time."
- 429 Too Many Requests → Alert Banner: "Too many login attempts. Please wait before trying again."
- Generic 500 → Alert Banner: "Something went wrong. Please try again."

**Loading state:** Loading Button shows spinner, fields disabled.

**Success redirect:**
- TUTEE → /dashboard
- TUTOR → /dashboard/tutor
- ADMIN → /login/admin/mfa (server redirects via partial cookie; full dashboard not accessible yet)

**Footer links:**
- "[Private supporting material excluded (publication note)](../../../docs/publication-notes.md)"
- "Don't have an account? [Private supporting material excluded (publication note)](../../../docs/publication-notes.md)"
- "Admin login → [Private supporting material excluded (publication note)](../../../docs/publication-notes.md)"

---

### 3.5 /login/admin — Admin Login

**Purpose:** Separate entry point for admin users. Identical form to /login but with distinct heading and no registration link.

**Heading:** "Admin Login"

**Form fields:** Same as /login (email, password).

**Current local/dev behavior:** use M16 admin login/password-change gate; multi-factor authentication is explicitly deferred in that scope. The following time-based one-time passcode user interface is a historical proposed production design, not authority to enable it; see the current open decisions.

**Note:** Admin login is not linked prominently from public pages; the URL is known to admins only.

---

### 3.6 /login/admin/mfa — Admin time-based one-time passcode Challenge

**Access guard:** Middleware checks for valid mfa pending cookie (short-lived, 5-minute signed session token with mfa pending: yes). If absent, redirect to /login/admin.

**Form fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| totp Code | text (numeric) | Yes | 6 digits |

**Zod validation:**
- totp Code: string, must match exactly six numeric digits → "Enter the 6-digit code from your authenticator app"

**Error states:**
- Invalid/expired time-based one-time passcode → Alert Banner: "Invalid or expired code. Please try again."
- 3rd consecutive failure → Alert Banner: "Too many failed attempts. Account locked for 15 minutes."
- mfa pending cookie expired (5-min window) → Redirect to /login/admin with info toast: "Session expired. Please log in again."

**Success:** Full signed session token issued; mfa pending cookie replaced; redirect to /admin.

**user experience:** Auto-focus on totp Code field. Numeric keyboard on mobile (input Mode is numeric). No paste blocking (allows password manager / authenticator app paste).

---

### 3.7 /forgot-password — Password Reset Request

**Form fields:**

| Field | Type | Required |
|-------|------|----------|
| email | email | Yes |

**Zod validation:**
- email: valid email format.

**Behaviour:**
- On submit, always show success state regardless of whether email exists (no enumeration).
- Success state: Replace form with message "If an account with that email exists, we've sent a 6-digit reset code. Check your inbox."
- Rate limit hit (429) → Alert Banner: "Too many reset requests. Please wait before trying again."

**Footer:** "[Private supporting material excluded (publication note)](../../../docs/publication-notes.md)"

---

### 3.8 /reset-password — one-time passcode + New Password

**Query params:** email (pre-filled, read-only) — passed from forgot-password flow via URL or session Storage.

**Form fields:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| email | email | Yes | Pre-filled, read-only if passed via query |
| otp | text (numeric) | Yes | 6-digit one-time passcode from email |
| new Password | password | Yes | |
| confirm Password | password | Yes | Must match new Password |

**Zod validation:**
- email: valid email.
- otp: string matching exactly six numeric digits → "Enter the 6-digit code from your email"
- new Password: same rules as registration password.
- confirm Password: must equal new Password.

**Error states:**
- 400 Invalid one-time passcode → Alert Banner: "The code is incorrect. Please check your email."
- 410 Expired/Used one-time passcode → Alert Banner: "This code has expired or already been used. [Private supporting material excluded (publication note)](../../../docs/publication-notes.md)."
- 422 Weak password → field error on new Password.
- Generic 500 → Alert Banner.

**Success:** Alert Banner (success variant): "Password reset successfully." Then redirect to /login after 2 seconds.

---

### 3.9 /account/delete — Account Deletion

**Access guard:** Requires valid auth cookie. Unauthenticated users redirected to /login.

**Role guard:** Available to TUTEE and TUTOR only. ADMIN accounts cannot self-delete via user interface (requires manual database action).

**Layout:** Deletion confirmation page. No form inputs — just confirmation steps.

**user experience flow:**
1. Page renders warning: "Deleting your account is permanent. Your profile and data will be removed within 30 days in accordance with Personal Data Protection Act. Active assignments may be cancelled."
2. User must type their email address into a confirmation input to unlock the "Delete My Account" button.
3. Confirmation input validation: must exactly match the authenticated user's email (fetched from read operation for auth me). If it doesn't match → inline error "Email does not match your account."
4. "Delete My Account" button becomes enabled only when email matches.
5. A secondary confirmation dialog (native the confirm operation or custom modal): "Are you sure? This cannot be undone."

**On confirm:**
- delete operation for auth account called.
- Success: Cookie cleared by server; client redirects to / with toast "Your account has been scheduled for deletion."
- Error 409 (active unpaid commissions): Alert Banner: "You have pending financial obligations. Please contact support before deleting your account."
- Generic 500: Alert Banner.

---

## 4. Client-Side Auth State

### 4.1 Auth Context (Auth Context)

Provided at root layout. Exposes:


Implementation example omitted. This public edition retains the written requirements in this section.


Populated on mount by calling read operation for auth me. On 401, user is set to no value.

### 4.2 Route Guards (Middleware)

Spring Security Jwt Authentication Filter at project root handles:

- Redirect authenticated users away from /register*, /login* to their dashboard.
- Redirect unauthenticated users away from /account/delete to /login?return To=/account/delete.
- Redirect non-ADMIN users away from /admin/* to /403.
- Check mfa pending cookie for /login/admin/mfa; redirect if missing.

### 4.3 Role-Based Dashboard Redirect

After login, the server response includes role in the signed session token. Client middleware reads the decoded role from the cookie on the server side and redirects:

| Role | Post-Login Destination |
|------|------------------------|
| TUTEE | /dashboard |
| TUTOR | /dashboard/tutor |
| ADMIN | /admin with the current admin session/password-change gate; multi-factor authentication depends on the approved environment policy |

---

## 5. Accessibility and user experience Standards

- All form inputs have associated <label> elements.
- Error messages are linked to inputs via aria-describedby.
- aria-live is polite on Alert Banner for screen reader announcements.
- Password fields use autocomplete is current-password (login) and autocomplete is new-password (registration/reset).
- time-based one-time passcode input uses autocomplete is one-time-code.
- Tab order follows visual reading order.
- All interactive elements have minimum 44×44px touch target.
- Colour contrast: all text meets WCAG AA (4.5:1 minimum).

## 2026-09-05 current contract and verification

Use the shared current password policy consistently on register/reset/change; do not infer separate complexity rules from this legacy description. Registration carries001 displayed policy-version IDs through tutor drafts and re-prompts on stale versions. Tutor OAuth role completion creates a usable browse account; commission/profile readiness gate posting/applying.004 temporary429 never clears the auth store; bounded refresh/retry handles terminal auth failure. Remaining policy questions are at [current open decisions](../../CURRENT-REQUIREMENTS.md).
