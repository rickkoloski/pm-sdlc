# Test Spec Format — Draft v0.1

**Status**: PROTOTYPE — using Brands list page as first example
**Date**: 2026-02-22
**Design goal**: Markdown that's skimmable by humans and precise for LLMs

---

## Format Specification

A test spec has these sections, in order. Sections marked (required) must be present.
Sections marked (when applicable) are included when relevant.

```
# Test: {Feature Name}                        (required)
## Context                                     (required)
## Prerequisites                               (required)
## Acceptance Criteria                         (required)
## Risk Areas                                  (when applicable — Layer 0)
## Test Data                                   (required for features with data input/output)
## Scenarios                                   (required)
  ### Scenario: {Name}
    **Steps** / **Verify** / **Testability**
## Testability Debt                            (when applicable — found during exploration)
## Knowledge References                        (when applicable — links to layers)
```

### Section Details

**Context**: What feature, what page, what scope. One paragraph max. Includes route and key component names so the LLM can locate code if needed.

**Prerequisites**: What must be true before any scenario runs. Server state, auth state, seed data. Each prerequisite is a checkable assertion, not prose.

**Acceptance Criteria**: What "done" looks like, sourced from the upstream spec. These are the success conditions the human cares about. Bulleted, each one verifiable.

**Risk Areas**: Concerns flagged by the architect, planning docs, or prior test runs. Each entry has: what might go wrong, why we think so, and what to probe. This is Layer 0 content.

**Scenarios**: The test cases. Each scenario has:
- **Steps**: Numbered sequence of actions. Each step is one interaction or one navigation. Plain English, but unambiguous — references element roles/labels, not coordinates.
- **Verify**: What to assert after the steps. Each assertion is a concrete, checkable statement.
- **Testability**: Notes on selector strategy. Flags any fragile selectors (coordinates, generated classes, DOM structure). Proposes improvements.

**Test Data**: Tabular data used across scenarios, organized by category. Designed for two audiences: humans scan for boundary conditions and domain reasonableness; LLMs read precise inputs and expected outputs. Subsections:

- **Seed Data**: What exists in the system before tests run. Table format. The baseline the scenarios assume.
- **Valid Inputs**: Good-path data for create/update scenarios. Each row is a test case with all field values and the expected outcome. Includes typical values and domain edge cases (e.g., max ABV, empty optional fields).
- **Boundary & Edge Cases**: Inputs designed to probe limits. Each row: input, which boundary it tests, expected behavior. Includes: empty required fields, max-length strings, numeric limits, special characters, duplicate names.
- **Search/Filter Cases**: Inputs paired with expected match results. Each row: search term, which field(s) it should match, expected row count, expected visible rows. Covers: exact match, partial match, case sensitivity, no match, numeric search.
- **Expected Outputs**: For computed/derived values (formula columns, status mappings, etc.). Architect-proposed, human-validated. Each row: input conditions → expected output value.

Categories are included only when relevant to the feature. A simple list page might only have Seed Data + Search/Filter Cases. A form-heavy feature will emphasize Valid Inputs + Boundary Cases.

**Testability Debt**: Items found during exploration where the code should be more testable. Each entry: what's fragile, where it is, what to fix.

**Knowledge References**: Pointers to knowledge layers that the test agent should read before executing. Prevents re-learning.

---

## Example: Brands List Page

---

# Test: Brands List Page

## Context

The Brands list page (`/brands`) displays all brands in a DataGridExtended with columns: Name, Style, ABV, IBU, Status. Users can search, add new brands via dialog, navigate to brand detail via row click, and toggle column visibility. Component: `BrandsList.tsx`. Grid uses `datagrid-extended` with `storageKey: 'rockcut:brands:list'`.

## Prerequisites

- [ ] API server running at `localhost:4002` (verify: `GET /api/health` returns 200)
- [ ] UI server running at `localhost:5174` (verify: `GET /` returns 200)
- [ ] Authenticated as admin user (Matt) — load saved state or run login flow
- [ ] Seed data present: at least 2 brands exist (verify: `GET /api/brands` returns non-empty array)

## Acceptance Criteria

From D5 scaffold UI spec and cumulative feature deliverables:

- Brands list page renders with PageHeader showing "Brands & Recipes" title
- DataGrid displays all brands with correct column data
- Search filters brands across name, style, status fields (client-side, case-insensitive)
- "Add Brand" button opens a form dialog for brand creation
- Created brands appear in the grid without page refresh
- Row click navigates to brand detail page (`/brands/{id}`)
- Column visibility toggle works and persists to localStorage
- Status column renders colored StatusChip (active=success, seasonal=info, retired=default)

## Risk Areas

| Risk | Source | What to probe |
|------|--------|---------------|
| Column visibility persistence under React 19 StrictMode | D9 fix — useRef gate on useEffect | Toggle column off → refresh → verify hidden. Toggle on → navigate away → back → verify. Hard refresh → verify. |
| Search filtering completeness | D8 implementation — useMemo filter | Search for partial name, partial style, numeric ABV value. Verify filter applies to all searchable fields. Clear search → verify all rows return. |
| DataGrid row click navigation | datagrid-extended onRowClick | Click row → verify URL changes to `/brands/{id}`. Verify correct id (not always first row). |
| StatusChip color mapping | StatusChip component | Verify "active" renders green chip, not just text. Requires computed style check (DOM text alone doesn't confirm color). |

## Scenarios

### Scenario: Page loads with data

**Steps**:
1. Navigate to `/brands`
2. Wait for DataGrid to render (grid role element visible with at least 1 row)

**Verify**:
- [ ] Page heading reads "Brands & Recipes"
- [ ] "Add Brand" button is visible and clickable
- [ ] Search field is visible with "Search..." placeholder
- [ ] DataGrid has 5 column headers: Name, Style, ABV, IBU, Status
- [ ] DataGrid displays expected number of rows (match against API response count)
- [ ] Pagination shows correct total (e.g., "1–2 of 2")

**Testability**:
- Page heading: accessible via `heading` role, level 4 — stable
- Add Brand button: accessible via `button` role, name "Add Brand" — stable
- Search field: accessible via `textbox` role, name "Search..." — stable
- Grid columns: accessible via `columnheader` role — stable
- Grid rows: accessible via `row` role within `rowgroup` — stable
- No `data-testid` attributes on this page yet — all selectors rely on accessibility roles

### Scenario: Search filters brands

**Steps**:
1. Navigate to `/brands` (or continue from previous scenario)
2. Click the search field (textbox "Search...")
3. Type "IPA"
4. Wait 300ms for filter to apply

**Verify**:
- [ ] Only rows containing "IPA" in any searchable field are visible
- [ ] Row count decreases (e.g., from 2 to 1)
- [ ] Matching row shows "Rockcut IPA" in Name column
- [ ] Pagination updates to reflect filtered count

**Steps** (continued):
5. Clear the search field (select all, delete)

**Verify**:
- [ ] All brands reappear
- [ ] Row count returns to full count

**Testability**:
- Search field: textbox role, name "Search..." — stable
- Filtering is client-side (no network request to wait for) — fast
- Row counting via `row` role elements in `rowgroup` — stable

### Scenario: Create new brand via dialog

**Steps**:
1. Navigate to `/brands`
2. Click "Add Brand" button
3. Wait for dialog to open (dialog role visible)
4. Fill "Name" field with "Test Stout"
5. Fill "Style" field with "Oatmeal Stout"
6. Fill "Status" field with "active"
7. Click "Save" button in dialog

**Verify**:
- [ ] Dialog opens with title "Add Brand"
- [ ] Dialog has fields: Name, Style, Description, Target ABV, Target IBU, Target SRM, Status
- [ ] After save: dialog closes
- [ ] Grid now contains a row with "Test Stout" / "Oatmeal Stout" / "active"
- [ ] No page refresh was required (mutation invalidated query cache)

**Steps** (cleanup):
8. Navigate to the new brand's detail page (click the "Test Stout" row)
9. Delete the test brand (click Delete → confirm)
10. Navigate back to `/brands`

**Verify**:
- [ ] "Test Stout" no longer appears in the grid

**Testability**:
- Dialog: accessible via `dialog` role — stable
- Dialog fields: accessible via `textbox` role with field name labels — **NEEDS VERIFICATION** (MUI TextField label association may vary)
- Save button: accessible via `button` role, name "Save" — stable
- **DEBT**: No `data-testid` on dialog or its fields. Add `data-testid="brand-form-dialog"` and `data-testid="brand-field-{name}"` for each field.

### Scenario: Row click navigates to detail

**Steps**:
1. Navigate to `/brands`
2. Click the row containing "Rockcut IPA"

**Verify**:
- [ ] URL changes to `/brands/{id}` (where id matches the Rockcut IPA brand id)
- [ ] BrandDetail page renders with "Rockcut IPA" heading
- [ ] Back navigation (browser back) returns to `/brands`

**Testability**:
- Row click: accessible via `row` role with name containing "Rockcut IPA" — stable
- URL verification: check `page.url()` — stable
- **Note**: Row identity depends on text content. If two brands had identical names, this selector would be ambiguous. `data-testid="brand-row-{id}"` would be more robust.

### Scenario: Column visibility toggle persists

**Steps**:
1. Navigate to `/brands`
2. Click "Toggle column visibility" button
3. Uncheck "Style" column
4. Close the visibility panel
5. Verify "Style" column is hidden
6. Reload the page (full page refresh)
7. Wait for DataGrid to render

**Verify**:
- [ ] After step 5: grid shows 4 columns (Name, ABV, IBU, Status) — no Style
- [ ] After step 7: grid still shows 4 columns — persistence survived refresh
- [ ] localStorage key `rockcut:brands:list` contains visibility state

**Steps** (restore):
8. Click "Toggle column visibility" button
9. Re-check "Style" column

**Verify**:
- [ ] Grid returns to 5 columns

**Testability**:
- Toggle button: accessible via `button` role, name "Toggle column visibility" — stable
- Column visibility panel: **NEEDS EXPLORATION** — what roles/labels does the visibility checkbox list expose?
- localStorage verification: use `localstorage-get 'rockcut:brands:list'` — direct
- **Risk area**: React 19 StrictMode double-mount (see Risk Areas table). Test must include full refresh, not just re-render.

## Test Data

### Seed Data (baseline — what exists before tests run)

These brands were created manually (not in seeds.exs). Verify via `GET /api/brands`.

| Brand Name | Style | Target ABV | Target IBU | Target SRM | Status |
|------------|-------|------------|------------|------------|--------|
| Altruism | German Amber | — | — | — | active |
| Rockcut IPA | American IPA | — | — | — | active |

**Note**: ABV, IBU, SRM columns show empty in the current data. This is expected (nullable fields, never populated). Not a bug.

### Valid Inputs (create/update happy path)

Each row is a standalone test case. Architect-proposed, domain-reasonable for a craft brewery.

| Case | Name | Style | Description | ABV | IBU | SRM | Status | Expected result |
|------|------|-------|-------------|-----|-----|-----|--------|-----------------|
| Typical beer | Test Stout | Oatmeal Stout | Rich and creamy | 5.5 | 30 | 35 | active | Created successfully |
| Minimal (only required) | Session Lager | | | | | | active | Created — only name + default status |
| All fields populated | Winter Warmer | English Strong Ale | Holiday seasonal with spices | 8.2 | 45 | 22 | seasonal | Created with all fields |
| Retired brand | Old Recipe | Brown Ale | Discontinued 2024 | 4.8 | 25 | 18 | retired | Created with retired status |
| Low ABV | Light Table Beer | Belgian Table | Easy drinking | 2.8 | 10 | 4 | active | Accepted — low but valid |
| High ABV | Imperial Monster | Barleywine | Cellar this one | 14.5 | 90 | 28 | active | Accepted — high but valid for style |
| High IBU | Hop Bomb | Triple IPA | Bitterness showcase | 10.0 | 120 | 8 | active | Accepted — extreme but real for style |

### Boundary & Edge Cases

| Case | Input | Boundary tested | Expected behavior |
|------|-------|-----------------|-------------------|
| Empty name | Name: "" (blank) | Required field validation | Reject — backend `validate_required([:name])` |
| Duplicate name | Name: "Rockcut IPA" | Unique constraint | Reject — `unique_constraint(:name)` returns error |
| Name only | Name: "Solo" + all others blank | Minimal valid input | Accept — name is the only required field |
| Invalid status | Status: "discontinued" | Status enum validation | Reject — `validate_inclusion(:status, ~w(active seasonal retired))` |
| ABV = 0 | ABV: 0 | Numeric zero | Accept — valid decimal, though unusual |
| ABV negative | ABV: -1 | Negative number | Accept? — **QUESTION**: No validation on numeric range in schema. Should there be? |
| ABV very high | ABV: 99.9 | Unreasonable but technically valid | Accept — no max validation. **Testability debt**: Consider adding range validation (0-30 for ABV is domain-reasonable) |
| Very long name | Name: 200+ chars | String length | Accept? — **QUESTION**: No `validate_length` in schema. Should there be? |
| Special chars in name | Name: "Röck & Roll #1" | Unicode, ampersand, hash | Accept — no character restrictions in schema |
| XSS in name | Name: `<script>alert(1)</script>` | Injection defense | Accept at API level (stored as string), but **must not execute in browser** — verify React escapes it |
| SQL injection attempt | Name: `'; DROP TABLE brands; --` | Injection defense | Accept safely — Ecto parameterized queries prevent SQL injection |

### Search/Filter Cases

Client-side filter: `useMemo` filtering across name, style, status, target_abv, target_ibu (case-insensitive substring).

| Search term | Should match | Expected rows | Why |
|-------------|-------------|---------------|-----|
| "IPA" | Rockcut IPA (name + style) | 1 | Partial match in name |
| "ipa" | Rockcut IPA | 1 | Case-insensitive |
| "active" | Both | 2 | Matches status field |
| "Amber" | Altruism | 1 | Matches style "German Amber" |
| "Alt" | Altruism | 1 | Partial match in name |
| "zzzzz" | None | 0 | No match — grid should show empty state |
| "" (cleared) | All | 2 | Empty search = no filter |
| "r" | Both | 2 | "Altruism" and "Rockcut" both contain "r" |

### Expected Outputs (status → visual mapping)

StatusChip rendering — verify both text content AND computed color.

| Status value | Chip text | Chip color (MUI) | Hex/visual | Accessible? |
|-------------|-----------|------------------|------------|-------------|
| active | Active | success | Green tones | **DEBT**: Color conveyed only via CSS, not in a11y tree |
| seasonal | Seasonal | info | Blue tones | Same debt |
| retired | Retired | default | Gray tones | Same debt |

## Testability Debt

Found during Playwright CLI exploration (2026-02-22):

| Item | Location | Current state | Proposed fix |
|------|----------|---------------|-------------|
| No data-testid on Brands page | `BrandsList.tsx` | All selectors rely on accessibility roles | Add `data-testid="brands-page"` to root, `data-testid="brands-search"` to search field, `data-testid="add-brand-btn"` to Add button |
| No data-testid on grid rows | DataGridExtended / BrandsList | Row identity via text content only | Add `data-testid="brand-row-{id}"` via DataGrid `getRowId` or `slotProps` |
| No data-testid on FormDialog | `FormDialog.tsx`, `BrandFormDialog.tsx` | Dialog findable by role, fields by label | Add `data-testid="brand-form-dialog"` to dialog, `data-testid="brand-field-{name}"` to each field |
| Column visibility panel selectors unknown | DataGridExtended | Not yet explored | Explore in next session; add aria-labels or testids if checkbox list lacks accessible names |
| StatusChip color not accessible | `StatusChip.tsx` | Color conveyed only via CSS, not in accessibility tree | Consider adding `aria-label="Status: active"` or `data-testid="status-chip-{value}"` |

## Knowledge References

- **App Map**: Route `/brands` → BrandsList component → DataGridExtended
- **Element Catalog**: See snapshot `.playwright-cli/page-2026-02-22T14-37-10-386Z.yml` for full accessibility tree
- **Interaction Recipes**: Login flow verified via CLI (fill e12, fill e15, click e16)
- **Timing Profile**: DataGrid renders within 2s of navigation (no explicit wait needed with Playwright CLI — it waits for actionability)
- **Gotchas**: Column visibility + StrictMode (D9), DOM != visual for StatusChip colors (D7 lesson)
- **Playbook**: `docs/testing/browser_automation_playbook.md` — element discovery patterns, timing guidelines
