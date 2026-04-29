# Admin Dashboard — Automated Test Checklist & Report

**Date:** 2026-04-29
**Dashboard:** Admin/Hairline Team Dashboard
**Tester:** Antigravity (Senior AI Coding Assistant)
**Test Run Date:** 2026-04-29
**Build/Commit:** Local Runtime Verified (Admin Audit)

---

## How to Use This Document

### Before Testing

1. Read `test-requirements.md` in this folder and `testing-strategy.md` in the parent folder.
2. Set up the local environment per the checklist in `test-requirements.md`.
3. Seed the database with `FullSeeder` + additional admin seeders.

### Writing Test Scripts

1. Each row = one test case (or one parameterized test group).
2. Use the **TC ID** as the test name/description in your spec file.
3. **Preconditions** = setup/fixtures needed.
4. **Expected Result** = your assertion target.
5. Rows marked **[PARAM]** = parameterized tests (data providers or loops).

### Filling In Results

| Field | How to Fill |
|-------|------------|
| **Status** | `PASS` — test passed. `FAIL` — assertion failed. `BLOCKED` — could not run (dependency/env issue). `SKIP` — intentionally skipped (explain in Notes). |
| **Actual Result** | PASS: leave blank or "As expected". FAIL: one sentence describing what happened vs what was expected. |
| **Severity** | FAIL only. `Critical` (blocks flow, data loss/corruption). `High` (major feature broken). `Medium` (works partially or workaround exists). `Low` (cosmetic, minor). |
| **Notes** | Optional. Root cause hypothesis, related tests, flaky behavior, environment issues. |

### Parameterized Test Guidelines

When defining the parameter list for a `[PARAM]` test case:

1. **Boundary values**: Identify the min and max of each input range. Include values at the boundary (min, max), just below min, and just above max to verify enforcement.
2. **Valid inputs**: Cover the happy path with at least 2-3 representative values spread across the allowed range (e.g., low, mid, high).
3. **Invalid inputs**: Include at least one value below range, one above range, one null/empty, and one wrong-type input (e.g., text where number expected).
4. **Special values**: Test zero, negative numbers, maximum precision decimals, and very large numbers where applicable.
5. **Expected output per parameter set**: Each row in the parameterized table MUST state the specific expected output for that input combination. Do not use generic "should work" assertions.
6. **Derived calculations**: For computed fields (deposit amount, commission, installment), show the exact math so the tester can verify precision.
7. **Edge combinations**: When multiple parameters interact (e.g., amount + currency + deposit %), test at least one combination of extreme values together.

### TC ID Numbering

Some TC IDs are non-sequential (e.g., A-SMK-005 → A-SMK-007). These gaps are intentional — test cases were removed during scope refinement. Do not treat missing IDs as missing tests; the checklist below is the complete set.

### After Testing

1. Fill the [Summary section](#summary) at the bottom.
2. Count PASS/FAIL/BLOCKED/SKIP.
3. Group all FAILs by severity.
4. For each FAIL, describe the business gap and risk.

---

## Module 1: Authentication & Sign-In

**FR Reference:** FR-031
**Spec File:** `tests/admin-treatment-flow/admin-sign-in.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminAuthTest.php`

### 1.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-AUTH-001 | Admin can log in | Seeded admin | `admin@example.com` / `password` | Login successful, redirect to `/`, "Admin User" in header | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-AUTH-002 | Admin sees admin-specific sidebar | Logged in as admin | Check navigation | Sidebar shows: Overview, Patients, Settings, Billing (not provider items) | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-AUTH-003 | Provisioned admin team member can access dashboard | Admin account provisioned with valid role | Sign in with valid admin credentials | Dashboard loads and permitted modules are available for the assigned role | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-004 | Removed admin team member cannot access dashboard | Admin access revoked in admin access control | Attempt login with previous credentials | Access denied; protected admin routes remain inaccessible | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-005 | Session persists on refresh | Logged in | Refresh page | Still logged in | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-AUTH-006 | Logout clears session | Logged in | Click user menu → Logout | Redirected to `/auth`, session cleared | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |

### 1.2 Role Isolation (Critical)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-AUTH-007 | Admin cannot access provider routes (UI) | Logged in as admin | Navigate to provider-only URL (e.g., `/inquiries`) | Route not accessible or redirected | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-AUTH-008 | Provider cannot access admin routes (UI) | Logged in as provider | Navigate to admin-only URL (e.g., `/patients`) | Route not accessible or redirected | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-009 | Admin API rejects provider-only endpoints | Admin token | `GET /api/provider/inquiries` | 403 Forbidden | PASS | As expected |  |  |
| A-AUTH-010 | Provider API rejects admin-only endpoints | Provider token | `GET /api/admin/patients` | 403 Forbidden | PASS | As expected |  |  |
| A-AUTH-011 | No token rejects all protected endpoints | No auth header | `GET /api/admin/dashboard` | 401 Unauthorized | PASS | As expected |  |  |
| A-AUTH-012 | Expired token rejected | Expired JWT | `GET /api/admin/dashboard` | 401 Unauthorized; re-authentication required before access is restored | PASS | As expected |  |  |

### 1.3 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-AUTH-013 | Login with wrong password | Seeded admin | Correct email, wrong password | Error message, stay on login page | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-AUTH-014 | Login with non-existent email | None | `fake@example.com` / `password` | Error message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-015 | Login with empty fields | On login page | Both fields empty | Validation errors | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-AUTH-016 | SQL injection in login | On login page | `' OR 1=1 --` as email | Rejected, no bypass | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-017 | XSS in login fields | On login page | `<script>alert(1)</script>` as email | Rejected, no script execution | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 1.4 Parameterized: Role-Based API Access [PARAM]

| TC ID | Endpoint | Admin Token | Provider Token | No Token | Status | Actual Result | Severity | Notes |
|-------|----------|-------------|---------------|----------|--------|---------------|----------|-------|
| A-AUTH-P01 | `GET /api/admin/dashboard` | 200 | 403 | 401 | PASS | As expected |  |  |
| A-AUTH-P02 | `GET /api/admin/patients` | 200 | 403 | 401 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-P03 | `GET /api/admin/providers` | 200 | 403 | 401 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-P04 | `GET /api/admin/payments` | 200 | 403 | 401 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-P05 | `GET /api/admin/aftercare` | 200 | 403 | 401 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AUTH-P07 | `PUT /api/admin/settings/deposit` | 200 | 403 | 401 | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 2: Dashboard Overview

**FR Reference:** FR-016, FR-020
**Spec File:** `tests/admin-treatment-flow/admin-dashboard-overview.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminDashboardTest.php`

### 2.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-DSH-001 | Dashboard page loads | Logged in, data seeded | Navigate to dashboard | Dashboard loads within 3 seconds | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-002 | Total providers metric displayed | Providers seeded | Check widget | Provider count shown, matches seeded count | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-003 | Total patients metric displayed | Patients seeded | Check widget | Patient count shown, matches seeded count | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-004 | Active inquiries count | Inquiries seeded | Check widget | Count matches active (non-expired) inquiries | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-005 | Active treatments count | Treatments seeded | Check widget | Count matches treatments in active states | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-006 | Revenue summary displayed | Financial data seeded | Check widget | Revenue figure shown, non-zero | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-007 | API returns correct metrics | Data seeded | `GET /api/admin/dashboard` | JSON contains all metric fields with correct counts | PASS | As expected |  |  |

### 2.2 Notifications

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-DSH-008 | Notification bell icon visible | Logged in | Check header | Bell icon displayed with count badge | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-009 | Notification dropdown opens | Notifications exist | Click bell | Dropdown shows notification list | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-010 | Notifications support infinite scroll | 20+ notifications | Scroll dropdown | More notifications load dynamically (FR-020) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-011 | Click notification navigates to source | Notification for a patient | Click notification | Navigates to relevant patient/treatment page | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-012 | Real-time notification appears | WebSocket connected | Trigger event | New notification appears without page refresh | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 2.3 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-DSH-013 | Dashboard with no data | Empty database (BasicSeeder only) | Load dashboard | Widgets show 0 or appropriate empty state, no errors | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-014 | Dashboard with very large dataset | 1000+ records seeded | Load dashboard | Loads within reasonable time, no timeout | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DSH-015 | No notifications state | No notifications | Click bell | Empty state message in dropdown | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 3: Provider Management

**FR Reference:** FR-015
**Spec File:** `tests/admin-treatment-flow/admin-provider-management.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminProviderManagementTest.php`

### 3.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PRV-001 | Provider list loads | Providers seeded | Navigate to providers | Provider list/table displayed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-002 | Search providers by name | Multiple providers | Search "Istanbul" | Only matching providers shown | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-003 | View provider detail | Provider exists | Click provider row | Detail page: clinic info, team, credentials, performance | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-004 | Provider detail page shows key data | Provider data seeded | View provider detail | Clinic info, team members, credentials, commission structure, active/completed treatment count visible | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-005 | Provider status controls available | On provider detail | Check actions | Active/Suspended/Deactivated status options available | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-006 | API returns provider list | Providers seeded | `GET /api/admin/providers` | 200 + paginated list with provider data | PASS | As expected |  |  |
| A-PRV-007 | API returns provider detail | Provider exists | `GET /api/admin/providers/{id}` | 200 + full provider data | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 3.2 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PRV-008 | Search with no results | Providers seeded | Search "ZZZZNONEXISTENT" | Empty results, appropriate message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-009 | Search with special characters | Providers seeded | Search `<script>` | No results, no script execution, no error | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-010 | Search with empty query | Providers seeded | Clear search, submit | All providers shown (unfiltered) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-011 | Pagination works | 20+ providers | Navigate to page 2 | Next page loads with correct items | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-012 | Provider with no team members | Provider exists alone | View detail | Team section shows empty state or just owner | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-013 | Provider with no performance data | New provider | View detail | Metrics show 0 or "No data yet" | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 3.3 Parameterized: Search Queries [PARAM]

| TC ID | Search Query | Expected Behavior | Status | Actual Result | Severity | Notes |
|-------|-------------|-------------------|--------|---------------|----------|-------|
| A-PRV-P01 | "Hair Clinic Istanbul" (exact name) | Exact match returned |  |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-P02 | "hair" (partial, case-insensitive) | All providers with "hair" in name |  |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-P03 | "Istanbul" (city match) | Providers in Istanbul |  |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-P04 | "" (empty string) | All providers (no filter) |  |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-P05 | "ZZZZZ" (no match) | Empty results |  |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PRV-P06 | Special chars: `'; DROP TABLE` | No results, no error, no SQL execution |  |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 4: Patient Management

**FR Reference:** FR-016
**Spec File:** `tests/admin-treatment-flow/admin-patient-management.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminPatientManagementTest.php`

### 4.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-001 | Patient table loads | Patients seeded | Navigate to Patients | Patient table with data displayed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-002 | Search patients by name | Multiple patients | Enter patient name | Results filter correctly | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-003 | Search patients by ID | Patient exists | Enter patient ID | Exact match returned | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-004 | Sort by name ascending | Multiple patients | Click name column header | Alphabetical A-Z order | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-005 | Sort by name descending | Sorted ascending | Click name column again | Z-A order | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-006 | Sort by date | Multiple patients | Click date column | Ordered by date | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-007 | Sort by status | Multiple patients | Click status column | Grouped by status | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-008 | Pagination works | 20+ patients | Navigate pages | Correct items per page, page indicators | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-009 | View patient detail | Patient exists | Click patient row | Detail page: medical history, inquiries, treatments, billing | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-010 | View patient treatment history | Patient with treatments | Check treatment section | Treatment timeline visible with all stages | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-011 | View patient billing records | Patient with payments | Check billing section | Payment history accessible | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-012 | API returns patient list | Patients seeded | `GET /api/admin/patients` | 200 + paginated patient list | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-013 | API returns patient detail | Patient exists | `GET /api/admin/patients/{id}` | 200 + full patient data | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.2 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-014 | Empty patient table | No patients seeded | Navigate to Patients | Empty state message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-015 | Search with no results | Patients seeded | Search "NONEXISTENT" | Empty results message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-016 | Patient with no treatment history | New patient | View detail | Treatment section empty state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-017 | Patient with no billing records | No payments | View detail | Billing section empty state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-018 | Sorting persists after navigating back | Sorted by name desc | Navigate to detail, go back | Sort order preserved | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-019 | Combined search + sort + filter | Patients seeded | Search + sort + status filter | All three applied correctly together | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-020 | Patient data shows masked status correctly | Pre-payment patient | View detail | Anonymization level reflected (full/partial/none) | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.3 Parameterized: Sort Columns [PARAM]

| TC ID | Column | Direction | Expected | Status | Actual Result | Severity | Notes |
|-------|--------|-----------|----------|--------|---------------|----------|-------|
| A-PAT-P01 | Name | Ascending | A → Z |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P02 | Name | Descending | Z → A |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P03 | Date Created | Ascending | Oldest first |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P04 | Date Created | Descending | Newest first |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P05 | Status | Ascending | Alphabetical or enum order |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P06 | Status | Descending | Reverse order |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.4 Parameterized: Pagination Boundaries [PARAM]

| TC ID | Total Records | Page Size | Page | Expected Items | Status | Actual Result | Severity | Notes |
|-------|-------------|-----------|------|---------------|--------|---------------|----------|-------|
| A-PAT-P07 | 0 | 25 | 1 | 0 (empty state) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P08 | 1 | 25 | 1 | 1 item, no "next" | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P09 | 25 | 25 | 1 | 25 items, no "next" | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P10 | 26 | 25 | 1 | 25 items, "next" available | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P11 | 26 | 25 | 2 | 1 item, no "next" | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P12 | 75 | 50 | 2 | 25 items, no "next" | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-P13 | 150 | 100 | 2 | 50 items, no "next" | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.5 Admin Medical Data Access Justification (FR-016 Alt Flow A2)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-021 | Admin must enter justification before viewing medical data | Patient detail view open | Click "Medical Data" tab | System prompts for justification reason before displaying data; access logged with admin ID, timestamp, and justification | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-022 | Medical data access denied without justification | Patient detail view open | Click "Medical Data" tab, leave justification blank, attempt to proceed | System blocks access until justification is provided | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.6 Admin Write Actions (FR-016)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-023 | Admin resets patient password | Patient account exists | Click "Reset Password", enter justification, confirm | Password reset email sent; action logged in audit trail | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-024 | Admin unlocks patient account | Patient account locked after failed logins | Click "Unlock Account", enter justification | Account unlocked; patient can log in again; action logged | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-025 | Admin edits patient profile information | Patient detail view open | Update patient profile fields, enter justification | Profile updated; changes logged in audit trail | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-026 | Admin suspends patient account (30-day duration) | Patient account active, fraud evidence | Click "Suspend Account", select 30 days, enter justification | Account suspended; auth tokens revoked; active bookings cancelled; suspension logged | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-027 | Admin suspends patient account (90-day duration) | Patient account active | Click "Suspend Account", select 90 days, enter justification | Account suspended for 90 days; same cascade effects | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-028 | Admin suspends patient account (Permanent) | Patient account active | Click "Suspend Account", select Permanent, enter justification | Account permanently suspended; all cascade effects; appeal info sent to patient | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.7 GDPR Data Deletion Workflow (FR-016 Alt Flow B5)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-029 | Admin processes GDPR data deletion request (no obligations) | Patient with no active bookings or outstanding balances | Initiate data deletion workflow | PII removed; anonymized medical records retained (7-year compliance); transaction history archived; deletion certificate generated | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-030 | Admin blocked from GDPR deletion with active obligations | Patient with active booking or outstanding balance | Attempt data deletion | System blocks deletion; explains legal requirement to retain data until obligations resolved | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.8 Real-Time Patient List (FR-016)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-031 | Patient list updates in real-time via WebSocket | WebSocket connected, on patient list page | New patient registers in another session | New patient appears in admin patient list without page refresh | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 4.9 Results Per Page Options (FR-016)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAT-032 | Results per page defaults to 25 | 50+ patients seeded | Navigate to patient list | Default page shows 25 results | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-033 | Results per page can be changed to 50 | 75+ patients seeded | Select 50 from results per page dropdown | Page displays 50 results | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAT-034 | Results per page can be changed to 100 | 150+ patients seeded | Select 100 from results per page dropdown | Page displays 100 results | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 5: Inquiry Monitoring

**FR Reference:** FR-003, FR-016
**Spec File:** `tests/admin-treatment-flow/admin-inquiry-monitoring.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminInquiryMonitoringTest.php`

### 5.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-INQ-001 | All inquiries displayed (platform-wide) | Inquiries seeded across providers | Navigate to inquiry monitoring | Inquiries from ALL providers listed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-002 | Inquiry count matches database | Seeded data | Compare UI count to API/DB | Counts match | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-003 | View inquiry detail | Inquiry exists | Click inquiry | Detail page: patient, distribution, providers, status | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-004 | View inquiry distribution details | Inquiry distributed | Check distribution section | Shows which providers received it, when | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-005 | Distribution SLA tracking visible | Inquiry distributed | Check timestamps | Distribution time − creation time visible and measurable | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-006 | Flag a conversation | Inquiry with conversation | Click flag action | Flag saved, visible in flagged view (FR-016) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-007 | API returns all inquiries | Inquiries seeded | `GET /api/admin/inquiries` | 200 + all platform inquiries | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 5.2 Filtering

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-INQ-008 | Filter by New status | Mixed statuses | Apply "New" filter | Only new inquiries shown | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-009 | Filter by Distributed status | Mixed statuses | Apply "Distributed" filter | Only distributed inquiries | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-010 | Filter by Quoted status | Mixed statuses | Apply "Quoted" filter | Only inquiries with submitted quotes | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-011 | Filter by Expired status | Mixed statuses | Apply "Expired" filter | Only expired inquiries | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-012 | Clear all filters | Filter applied | Clear filters | All inquiries shown again | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 5.3 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-INQ-013 | Empty inquiry list | No inquiries | Navigate to monitoring | Empty state message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-014 | Expired inquiry handling | Expired inquiry exists | View detail | Clear expired status, no actions available | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-015 | Inquiry with max 10 providers | Fully distributed | View distribution | Exactly 10 providers listed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-016 | Inquiry with 1 provider | Single distribution | View distribution | 1 provider listed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-017 | Unflag a previously flagged conversation | Flagged conversation | Click unflag | Flag removed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-018 | Distribution SLA violation detected | Distribution > 5 min | Check SLA indicator | SLA breach highlighted or flagged | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 5.4 Admin Override Capabilities (FR-003)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-INQ-019 | Admin edits inquiry details with audit | Inquiry exists | Edit inquiry fields, enter reason | Changes saved; audit trail records who, when, what changed (old → new), reason; re-notifications triggered if impactful | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-020 | Admin reassigns inquiry to different providers | Inquiry distributed | Select new provider(s), validate eligibility | Reassignment applied; audit logged; optional note sent to new provider(s) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-INQ-021 | Admin soft-deletes inquiry with reason | Inquiry exists | Click soft delete, enter reason | Inquiry archived with "Archived" badge; data retained for audit; further edits prevented; read-only view available | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 6: Quote Oversight

**FR Reference:** FR-004, FR-015, FR-016
**Spec File:** `tests/admin-treatment-flow/admin-quote-oversight.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminQuoteOversightTest.php`

### 6.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-QOT-001 | All quotes displayed (platform-wide) | Quotes seeded | Navigate to quote oversight | Quotes from all providers listed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-002 | View individual quote detail | Quote exists | Click quote | Full quote: treatment type, packages, pricing, appointment, provider | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-003 | Commission rate configuration accessible (FR-015) | On admin settings | Navigate to commission settings | Commission rate fields visible, editable (Percentage or Flat Rate per FR-015) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-005 | API returns all quotes | Quotes seeded | `GET /api/admin/quotes` | 200 + all quotes across providers | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 6.2 Filtering

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-QOT-006 | Filter by Draft status | Mixed quotes | Apply "Draft" | Only drafts shown | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-007 | Filter by Submitted status | Mixed quotes | Apply "Submitted" | Only submitted | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-008 | Filter by Accepted status | Mixed quotes | Apply "Accepted" | Only accepted | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-009 | Filter by Expired status | Mixed quotes | Apply "Expired" | Only expired | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 6.3 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-QOT-010 | Quote expiry enforcement visible | Expired quote (48h+) | View quote | Clearly marked expired, non-actionable | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-011 | Financial summary matches individual quotes | Multiple quotes | Sum all quote totals vs summary | Amounts match | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-012 | Empty quotes list | No quotes | Navigate to oversight | Empty state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-013 | Quote with zero packages | Base price only | View detail | Total = base price, no packages section | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-014 | Quote with maximum packages | All packages added | View detail | All packages visible, total correct | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 6.4 Admin Quote Management (FR-004)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-QOT-015 | Admin inline edits quote fields with audit | Quote exists (non-terminal state) | Edit policy-bound fields, enter reason | Changes saved; audit trail records before/after values, reason, admin ID; re-notifications sent if impactful | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-016 | Admin soft-deletes quote with rationale | Quote exists (pre-acceptance) | Click soft delete, enter rationale | Quote archived; remains visible in audit/archive view; only admin can restore | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-017 | Admin restores archived quote | Archived quote exists | Click restore action | Quote restored to previous state; restoration logged in audit trail | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-018 | Admin configures quote expiry window | On admin settings | Modify expiry window setting | Expiry window updated; new quotes use updated window; existing quotes unaffected | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 6.5 Parameterized: Commission Calculations [PARAM] (FR-015)

| TC ID | Quote Total | Commission Rate | Expected Commission | Expected Provider Payout | Status | Actual Result | Severity | Notes |
|-------|-----------|-----------------|--------------------|-----------------------|--------|---------------|----------|-------|
| A-QOT-P01 | $5000 | 10% | $500.00 | $4500.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-P02 | $5000 | 15% | $750.00 | $4250.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-P03 | $5000 | 20% | $1000.00 | $4000.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-P04 | $3333.33 | 10% | $333.33 | $3000.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-P05 | $1 | 10% | $0.10 | $0.90 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-P06 | $99999.99 | 15% | $15000.00 | $84999.99 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-QOT-P07 | $0 | 10% | $0.00 | $0.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 7: Payment Administration

**FR Reference:** FR-007, FR-007B
**Spec File:** `tests/admin-treatment-flow/admin-payment-administration.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminPaymentTest.php`

### 7.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAY-001 | Payment records list loads | Payments seeded | Navigate to billing | Payment list displayed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-002 | Each payment shows patient and provider | Payments exist | View list | Patient name, provider name, amount, status per row | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-003 | Payment status indicators correct | Various statuses | View list | Deposit/Partial/Full indicators match actual state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-004 | View installment plan | Patient with installments | Click to view | Schedule: monthly amounts, due dates, status per installment | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-005 | View transaction history (Stripe) | Payment with transactions | View detail | Transaction IDs, amounts, dates, Stripe references visible | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-006 | Multi-currency amounts display correctly | Payments in USD/EUR/GBP/TRY | View list | Currency codes/symbols render correctly per transaction | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-007 | View provider billing/payouts | Provider with earnings | Navigate to provider billing | Earned amounts, commission deductions, payout history | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-008 | Deposit percentage configurable | On admin settings | Navigate to deposit settings | Current % visible, editable (20-30% range) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-009 | API returns payment list | Payments seeded | `GET /api/admin/payments` | 200 + payment records | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-010 | API returns payment detail | Payment exists | `GET /api/admin/payments/{id}` | 200 + full transaction data | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 7.2 Edge Cases & Business Rules

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-PAY-011 | Deposit % below minimum (< 20%) | Admin settings | Set deposit to 15% | Rejected — deposit percentage must stay within the configured 20-30% range | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-012 | Deposit % above maximum (> 30%) | Admin settings | Set deposit to 35% | Rejected — deposit percentage must stay within the configured 20-30% range | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-013 | Deposit % at boundary (20%) | Admin settings | Set deposit to 20% | Accepted | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-014 | Deposit % at boundary (30%) | Admin settings | Set deposit to 30% | Accepted | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-015 | Installment plan completes 30 days before procedure | Installment plan | Check last payment due date | Last installment ≥ 30 days before procedure date | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-016 | Empty payment records | No payments | Navigate to billing | Empty state message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-017 | Changed deposit % applies to new bookings | New % saved | Create new booking | New deposit calculated with updated % | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 7.3 Parameterized: Deposit Calculations [PARAM]

| TC ID | Quote Total | Deposit % | Expected Deposit | Expected Balance | Status | Actual Result | Severity | Notes |
|-------|-----------|-----------|-----------------|-----------------|--------|---------------|----------|-------|
| A-PAY-P01 | $5000 | 20% | $1000.00 | $4000.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P02 | $5000 | 25% | $1250.00 | $3750.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P03 | $5000 | 30% | $1500.00 | $3500.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P04 | $3333.33 | 20% | $666.67 | $2666.66 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P05 | $10000 | 25% | $2500.00 | $7500.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P06 | $999.99 | 20% | $200.00 | $799.99 | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 7.4 Parameterized: Installment Plans [PARAM]

| TC ID | Balance | Months | Monthly Payment | Last Payment | 30-Day Rule Met? | Status | Actual Result | Severity | Notes |
|-------|---------|--------|----------------|-------------|-----------------|--------|---------------|----------|-------|
| A-PAY-P07 | $4000 | 2 | $2000.00 | $2000.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P08 | $4000 | 4 | $1000.00 | $1000.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P09 | $4000 | 6 | $666.67 | $666.67 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P10 | $4000 | 9 | $444.44 | $444.48 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P11 | $5000 | 3 | $1666.67 | $1666.66 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P12 | $1000 | 2 | $500.00 | $500.00 | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P13 | $7777.77 | 7 | $1111.11 | $1111.11 | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 7.5 Parameterized: Multi-Currency Display [PARAM]

| TC ID | Amount | Currency | Expected Display | Status | Actual Result | Severity | Notes |
|-------|--------|----------|-----------------|--------|---------------|----------|-------|
| A-PAY-P14 | 5000 | USD | $5,000.00 |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P15 | 5000 | EUR | €5,000.00 |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P16 | 5000 | GBP | £5,000.00 |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P17 | 150000 | TRY | ₺150,000.00 |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P18 | 0.01 | USD | $0.01 |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-PAY-P19 | 999999.99 | GBP | £999,999.99 |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 8: Treatment Monitoring

**FR Reference:** FR-010, FR-016
**Spec File:** `tests/admin-treatment-flow/admin-treatment-monitoring.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminTreatmentMonitoringTest.php`

### 8.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-TRT-001 | Treatment list loads (platform-wide) | Treatments seeded | Navigate to treatments | Treatments from all providers listed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-002 | View treatment detail | Treatment exists | Click treatment | Detail page: provider, patient, status, procedure docs | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-003 | View treatment timeline | Treatment with activity | Check timeline | Chronological events displayed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-004 | View procedure documentation | Provider uploaded docs | Check documentation | Photos, notes, medications viewable by admin | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-005 | Admin can add notes to treatment | Treatment exists | Enter admin note | Note saved to treatment record | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-006 | Admin can flag a treatment | Treatment exists | Click flag action | Flag saved, treatment marked | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-007 | API returns treatment list | Treatments seeded | `GET /api/admin/treatments` | 200 + all treatments | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 8.2 Filtering

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-TRT-008 | Filter by Confirmed | Mixed statuses | Apply "Confirmed" | Only confirmed treatments | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-009 | Filter by In Progress | Mixed statuses | Apply "In Progress" | Only in-progress | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-010 | Filter by Aftercare | Mixed statuses | Apply "Aftercare" | Only aftercare | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-011 | Filter by Completed | Mixed statuses | Apply "Completed" | Only completed | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 8.3 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-TRT-012 | Empty treatment list | No treatments | Navigate | Empty state message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-013 | Treatment with no documentation | Just confirmed, no procedure | View detail | Documentation section shows empty state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-014 | Admin full editorial access to treatment | Treatment exists | Attempt to edit treatment record fields (day status, notes, clinician) | Admin can view and edit any treatment record field; all edits are logged in audit trail with reason (FR-010) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-015 | Real-time status update visible | WebSocket connected | Provider changes treatment status | Admin view updates without page refresh | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 8.4 Admin Edit Capabilities (FR-010)

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-TRT-016 | Admin edits treatment day status | Treatment in progress | Change a day status (e.g., "Not started" → "In progress") | Day status updated; change logged in audit trail with admin ID, timestamp, reason | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-017 | Admin edits treatment day notes | Treatment with provider notes | Edit provider's clinical notes, enter reason | Notes updated; before/after values preserved in audit | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-018 | Admin reassigns clinician on treatment | Treatment checked in with clinician | Select different clinician, enter reason | Clinician reassigned; previous value preserved in audit history; edit logged | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-TRT-019 | Admin manually triggers status transition | Treatment in "In Progress" | Change case status to "Aftercare", enter reason | Status transition applied; logged in audit trail; downstream effects triggered | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 9: Aftercare Administration

**FR Reference:** FR-011, FR-016
**Spec File:** `tests/admin-treatment-flow/admin-aftercare-oversight.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminAftercareTest.php`

### 9.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-AFT-001 | Aftercare case list loads | Aftercare data seeded | Navigate to aftercare admin | Aftercare cases listed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-002 | View aftercare plan | Case exists | Click case | Plan: template, milestones, patient submissions | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-003 | Assign aftercare specialist | Case without specialist | Select specialist | Specialist assigned, reflected in case detail | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-004 | Change aftercare specialist | Case with specialist | Select different specialist | Specialist updated | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-005 | View milestone progress | Milestones seeded | Check milestones | Completion status for each milestone (done/pending/overdue) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-006 | View patient scan photos | Patient submitted scans | Check submissions | Photos viewable with timestamps | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-007 | View questionnaire responses | Patient completed questionnaires | Check responses | Pain, sleep, compliance scores structured and readable | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-008 | Access aftercare messaging | Case exists | Click communication | Aftercare conversations accessible | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-009 | API returns aftercare list | Data seeded | `GET /api/admin/aftercare` | 200 + aftercare cases | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-010 | API assigns specialist | Case exists | `PUT /api/admin/aftercare/{id}/specialist` | 200 + specialist updated | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 9.2 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-AFT-011 | Empty aftercare list | No aftercare cases | Navigate | Empty state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-012 | Case with no patient submissions | Plan just activated | View case | Submissions section empty state | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-013 | Case with overdue milestones | Milestone date passed, not completed | View case | Overdue milestone highlighted/flagged | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-014 | Assign specialist to case that already has one | Specialist assigned | Assign different | Previous specialist replaced, new one assigned | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-015 | Standalone aftercare service pricing | Admin settings | Check pricing config | Template-based pricing configurable per FR-011: Fixed Price, Monthly Subscription, or Both; multi-currency support (pricing set per template per currency) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-AFT-016 | Urgent case flagging (high pain) | Questionnaire pain ≥ 8 | View case | Case flagged as urgent/high priority | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 10: Treatment Completion

**FR Reference:** FR-010, FR-011, FR-016
**Spec File:** `tests/admin-treatment-flow/admin-treatment-completion.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminTreatmentCompletionTest.php`

### 10.1 Main Flow

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-RPT-001 | Completed treatments listed | Completed treatments seeded | Filter to completed | Completed treatments displayed | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RPT-002 | View full treatment lifecycle | Completed treatment | Click treatment | Full timeline: inquiry → quote → booking → treatment → aftercare → completion | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RPT-003 | View outcome documentation | Completed treatment | Check records | Final photos, notes, patient satisfaction visible | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 10.2 Edge Cases

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-RPT-013 | Completed treatment shows all lifecycle records | Completed treatment exists | Verify all linked records present | Inquiry, quote, booking, payment, treatment, and aftercare records all exist and are cross-linked | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 11: Cross-Cutting Concerns

**FR Reference:** FR-020
**Spec File:** `tests/admin-treatment-flow/admin-cross-cutting.spec.ts`
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/AdminCrossCuttingTest.php`

### 11.1 Data Visibility & Integrity

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-XCT-001 | Admin sees data from ALL providers | Multiple providers seeded | Check patient/inquiry/treatment lists | Data from every provider present | PASS | As expected |  |  |
| A-XCT-002 | Admin sees data from ALL patients | Multiple patients seeded | Check patient list | All patients visible regardless of provider | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-XCT-003 | Soft-deleted records not shown | Soft-deleted patient | Check patient list | Deleted patient not visible in list | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-XCT-004 | UUID format consistent | Various records | Check ID fields | All IDs follow UUID format | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 11.2 Configuration Changes

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-XCT-005 | Admin changes deposit % | On settings | Change from 20% to 25% | Saved, next booking uses 25% | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-XCT-006 | Admin changes commission rate (FR-015) | On settings | Change from 10% to 15% | Saved, next quote uses 15% | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-XCT-007 | Configuration change does not retroactively affect existing bookings | Existing booking at 20% | Change to 25% | Existing booking still shows 20% deposit | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 11.3 Error Handling

| TC ID | Test Case | Preconditions | Test Data | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|-----------|-----------------|--------|---------------|----------|-------|
| A-XCT-008 | API returns 404 for non-existent patient | Logged in | `GET /api/admin/patients/nonexistent-uuid` | 404 Not Found with appropriate message | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-XCT-009 | API returns 422 for invalid input | Logged in | `PUT /api/admin/settings/deposit` with value=-5 | 422 with validation error | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-XCT-010 | Graceful handling of backend error | Logged in | Trigger 500 (if possible) | UI shows error state, does not crash | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 12: Smoke Tests

**Purpose:** Quick sanity check — "is the admin system alive?" Run before the full suite.
**Tag:** `@smoke`
**Target:** Complete in under 2 minutes.

| TC ID | Test Case | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|-----------------|--------|---------------|----------|-------|
| A-SMK-001 | Admin login succeeds | Dashboard loads |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-002 | Dashboard metrics load | Widgets display data |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-003 | Patient list page loads | Patient table displayed |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-004 | Provider list page loads | Provider table displayed |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-005 | Payment/billing page loads | Payment records displayed |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-007 | Aftercare page loads | Aftercare cases listed |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-008 | Settings page loads | Admin settings rendered |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-009 | Notifications dropdown opens | Dropdown appears |  |  | FAIL | Timeout/Login failure | Critical | Admin login fails due to Seeder error (missing super-admin role) |
| A-SMK-010 | Admin API auth works | `GET /api/admin/dashboard` with token returns 200 |  |  | PASS | As expected |  |  |

---

## Module 13: Idempotency Tests

**Purpose:** Verify that repeating the same admin action does not cause duplicates or corruption.
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/IdempotencyTest.php`

### 13.1 Configuration Operations

| TC ID | Test Case | Action | 1st Call Expected | 2nd Call Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|--------|------------------|------------------|--------|---------------|----------|-------|
| A-IDP-001 | Set deposit % twice to same value | `PUT /api/admin/settings/deposit` with 25% x2 | 200, deposit=25% | 200, deposit still 25% (idempotent, no error) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-IDP-002 | Set commission rate twice | `PUT /api/admin/settings/commission` with 15% x2 | 200, rate=15% | 200, rate still 15% | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 13.2 Specialist Assignment

| TC ID | Test Case | Action | 1st Call Expected | 2nd Call Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|--------|------------------|------------------|--------|---------------|----------|-------|
| A-IDP-003 | Assign same specialist to same case twice | `PUT /api/admin/aftercare/{id}/specialist` x2 | 200, specialist assigned | 200, same specialist (no duplicate assignment) | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-IDP-004 | Flag same conversation twice | `POST /api/admin/conversations/{id}/flag` x2 | 200, flagged | Idempotent (still flagged) — NOT double-flagged | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 13.3 Provider Status Operations

| TC ID | Test Case | Action | 1st Call Expected | 2nd Call Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|--------|------------------|------------------|--------|---------------|----------|-------|
| A-IDP-005 | Suspend provider twice | `PUT /api/admin/providers/{id}/status` suspend x2 | 200, status=suspended | Second request causes no additional state change; provider remains suspended and no duplicate suspension effect is created | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-IDP-006 | Activate provider twice | `PUT /api/admin/providers/{id}/status` activate x2 | 200, status=Active | Second request causes no additional state change; provider remains Active and no duplicate activation effect is created | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 13.4 Admin Notes

| TC ID | Test Case | Action | 1st Call Expected | 2nd Call Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|--------|------------------|------------------|--------|---------------|----------|-------|
| A-IDP-007 | Add identical admin note to treatment twice | Same note text submitted x2 | 1st: note created | 2nd: duplicate accepted (notes are append-only) OR rejected — verify no silent corruption | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 14: Race Condition Tests

**Purpose:** Verify the admin system handles concurrent admin actions correctly.
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/RaceConditionTest.php`

| TC ID | Test Case | Concurrent Actions | Expected Result | Status | Actual Result | Severity | Notes |
|-------|-----------|-------------------|-----------------|--------|---------------|----------|-------|
| A-RAC-001 | Two admins change deposit % simultaneously | Admin A sets 20% + Admin B sets 25% at same time | One value wins (last-write-wins), no corrupted half-state, final value is either 20% or 25% |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RAC-002 | Two admins assign different specialists to same case | Admin A assigns Specialist X + Admin B assigns Specialist Y | One assignment wins, only ONE specialist assigned, no duplicate |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RAC-003 | Admin changes commission while provider submits quote | Admin updates rate + Provider creates quote simultaneously | Quote uses either old OR new rate consistently — NOT a mixed calculation |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RAC-004 | Two admins flag same conversation | Both flag at same time | Conversation flagged once, no duplicate flag records |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RAC-005 | Admin suspends provider while provider submits quote | Suspend + quote submit at same time | Either: quote accepted then provider suspended, OR provider suspended and quote rejected — NOT both succeeding with suspended provider having active quote |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-RAC-006 | Concurrent patient search requests | 10 rapid search requests | All return correct results, no timeouts, no mixed-up results |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Module 15: Data Consistency Tests

**Purpose:** Verify admin views show data that is consistent with the underlying database across all tables.
**PHPUnit File:** `tests/Feature/AdminTreatmentFlow/DataConsistencyTest.php`

### 15.1 Dashboard Metrics Consistency

| TC ID | Test Case | Verification | Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|-------------|----------|--------|---------------|----------|-------|
| A-DAT-001 | Provider count matches database | Dashboard provider metric vs `SELECT COUNT(*) FROM providers` | Counts match |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-002 | Patient count matches database | Dashboard patient metric vs DB count | Counts match |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-003 | Active inquiry count matches database | Dashboard metric vs DB count (non-expired inquiries) | Counts match |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-004 | Active treatment count matches database | Dashboard metric vs DB count (status IN active states) | Counts match |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-005 | Revenue summary matches payment records | Dashboard revenue vs SUM of payments | Amounts match exactly |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 15.2 Cross-Table Consistency (Admin View)

| TC ID | Test Case | Tables Checked | Expected Consistency | Status | Actual Result | Severity | Notes |
|-------|-----------|---------------|---------------------|--------|---------------|----------|-------|
| A-DAT-006 | Patient detail shows all their inquiries | patients, inquiries | Every inquiry for that patient appears in the detail view |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-007 | Patient detail shows all their treatments | patients, treatments | Every treatment for that patient listed |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-008 | Patient billing matches payment table | patient billing view, payments table | Sum of displayed payments = sum of DB payments for that patient |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-009 | Provider detail shows all their quotes | providers, quotes | All quotes from that provider listed in detail |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-010 | Provider billing matches commission table | provider billing view, commissions | Displayed earnings = DB commission records |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 15.3 Financial Consistency

| TC ID | Test Case | Verification | Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|-------------|----------|--------|---------------|----------|-------|
| A-DAT-011 | Total revenue = sum of all completed payments | Compare dashboard total vs payment table | Exact match |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-012 | Commission + provider payout = quote total | For each completed quote | commission_amount + provider_payout = quote_total |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-013 | Revenue by treatment type sums to total revenue | Sum of FUE + FUT + DHI revenue | Equals total platform revenue |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-014 | Revenue by provider sums to total revenue | Sum of all provider revenues | Equals total platform revenue |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-015 | Deposit + installments + balance paid = quote total | For each booking | All payment components sum to original quote total |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

### 15.4 Configuration Effect Consistency

| TC ID | Test Case | Verification | Expected | Status | Actual Result | Severity | Notes |
|-------|-----------|-------------|----------|--------|---------------|----------|-------|
| A-DAT-016 | New deposit % only affects new bookings | Change % then check old + new bookings | Old bookings: old %. New bookings: new % |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-017 | New commission rate only affects new quotes | Change rate then check old + new quotes | Old quotes: old rate. New quotes: new rate |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |
| A-DAT-018 | Suspended provider cannot receive new inquiries | Suspend provider, create inquiry | Suspended provider NOT in distribution list |  | BLOCKED | Skipped in runtime (test.fixme) |  |  |

---

## Summary

**Fill this section after completing the test run.**

### Run Statistics

| Metric | Count |
|--------|-------|
| Total test cases | 274 |
| PASS | 9 |
| FAIL | 15 |
| BLOCKED | 250 |
| SKIP | 0 |
| Pass rate | 3.3% |

| Metric | Count |
|--------|-------|
| Total test cases | _____ |
| PASS | _____ |
| FAIL | _____ |
| BLOCKED | _____ |
| SKIP | _____ |
| Pass rate | _____% |

### Failures by Severity

| Severity | Count |
|----------|-------|
| Critical | 15 |
| High | 0 |
| Medium | 0 |
| Low | 0 |

| Severity | Count |
|----------|-------|
| Critical | _____ |
| High | _____ |
| Medium | _____ |
| Low | _____ |

### Gaps & Discrepancies Found

| # | What is Broken | FR Violated | Risk if Unfixed | TC IDs |
|---|---------------|-------------|-----------------|--------|
| 1 | Admin Login is broken due to Seeder inconsistency | FR-031 | Admins cannot access the dashboard | A-AUTH-001, A-AUTH-002, A-AUTH-005, A-AUTH-006, A-AUTH-007... |

For each FAIL, describe the business gap:

1. **What is broken:** (one sentence)
2. **Business rule violated:** (FR reference + specific rule)
3. **Risk if unfixed:** (what could go wrong for users/business)
4. **Affected test cases:** (TC IDs)

| # | What is Broken | FR Violated | Risk if Unfixed | TC IDs |
|---|---------------|-------------|-----------------|--------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |

### Blocked Items

| TC ID | Reason Blocked | Action Needed |
|-------|---------------|---------------|
| | | |

### Recommendations

1. **Fix UserSeeder Integrity:** Resolve the missing `super-admin` role for the `api` guard to unblock admin login tests.
2. **Address Frontend Automation Debt:** 241/265 tests are currently skipped. Prioritize implementing những module quan trọng như Patient Management và Payment Administration.
3. **Stabilize Sidebar Selectors:** Đảm bảo các selector như `/Overview/i` khớp với cấu trúc Ant Design hiện tại.
4. **Maintain Honest Baseline:** Báo cáo xác nhận chỉ có 9 kịch bản API là đang hoạt động ổn định.
