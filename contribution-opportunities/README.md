# Grafana Contribution Opportunities

This document tracks verified, open Grafana issues suitable for contributions. All issues listed here are:

- ✅ **Open and unassigned** (verified November 10, 2025)
- ✅ **NO active volunteers or pull requests** (filtered for truly available work)
- ✅ **Marked as "good first issue"** or "help wanted"
- ✅ **Have clear implementation context** in the issue description

Use this as a reference for finding issues to contribute to, learning the codebase, and improving Grafana's documentation and features.

---

## ✨ Perfect Starting Points: Zero Comments, No Volunteers

These issues have **no comments** - they're clean and ready for someone to pick up!

### 1. Combobox: Handle proper typing (#95386)

**Difficulty:** Medium | **Effort:** Medium | **Type:** Enhancement

**Status:** Open, unassigned | **Comments:** 0 | **Labels:** help wanted

**Link:** https://github.com/grafana/grafana/issues/95386

**Summary:**
The Combobox component has type inconsistencies when custom values are enabled. If value types are set to numbers and custom value is enabled, the component returns a string instead. Similar issues exist with MultiCombobox "All" value typing.

**What needs fixing:**

- Ensure proper typing when using custom values (should return correct type, not always string)
- Fix typing for MultiCombobox "All" value option
- Make `ComboboxOption["label"]` non-optional when returned by async functions

**Technical context:**

- File: `packages/grafana-ui/src/components/Combobox/Combobox.tsx#L168`
- Related PR discussion: https://github.com/grafana/grafana/pull/94147

**Skills needed:** TypeScript, React components, type systems

**Notes:**

- This touches core UI components used throughout Grafana
- Good opportunity to understand Grafana's typing patterns
- Exploration and cleanup of type definitions is already documented in the issue

---

### 2. Transformations: Clearer conditions in filter by values (#97239)

**Difficulty:** Easy | **Effort:** Small | **Type:** Enhancement

**Status:** Open, unassigned | **Comments:** 0 | **Labels:** good first issue

**Link:** https://github.com/grafana/grafana/issues/97239

**Summary:**
The "Conditions" option in the "Filter by values" transformation is confusing when there's only a single condition. Users don't understand if it means "match all" or "match any" with just one condition. The UI should either hide this option or add a clear explanation.

**What needs fixing:**

- Hide the "Conditions" dropdown when there's only one condition, OR
- Add a help icon with explanation, OR
- Make the label clearer ("Match condition" vs "Match any" vs "Match all")

**Implementation suggestions:**

1. Conditional rendering - hide when `conditions.length === 1`
2. Or add a help icon with tooltip explaining the behavior
3. Test with single and multiple conditions

**Related files:**

- `public/app/plugins/datasource/*/transformations/` or `packages/grafana-ui/transformations/`
- Look for "Filter by values" component

**Skills needed:** React, UI/UX thinking, conditional rendering

**Impact:** This is a direct UX improvement that will help new users understand the transformation better.

---

### 3. Dashboard: Keyboard focus is reset when deleting a panel (#95721)

**Difficulty:** Medium | **Effort:** Small-Medium | **Type:** Bug/UX

**Status:** Open, unassigned | **Comments:** 1 | **Labels:** help wanted, effort/small, accessibility

**Link:** https://github.com/grafana/grafana/issues/95721

**Summary:**
When a user deletes a dashboard panel using keyboard navigation (Tab), the keyboard focus resets instead of moving to the next or previous panel. This breaks keyboard accessibility workflows.

**What needs fixing:**

- When a panel is deleted via keyboard, focus should move to:
  - The next panel (if available), or
  - The previous panel (if next doesn't exist), or
  - A sensible fallback element

**Technical context:**

- Focus management in dashboard panel components
- Likely in: `public/app/features/dashboard/components/`
- Related to accessibility (WCAG 2.1.1)

**Skills needed:** React, focus management, accessibility, keyboard events

**Impact:** Improves accessibility for keyboard users and may prevent other focus-related issues.

---

## 🔧 Solid Opportunities: 1-2 Comments Only

### 4. Incomplete Polish Localization in Grafana v12.1.0.0 (#110215)

**Difficulty:** Easy-Medium | **Effort:** Medium | **Type:** Localization

**Status:** Open, unassigned | **Comments:** 1 | **Labels:** help wanted, internationalization

**Link:** https://github.com/grafana/grafana/issues/110215

**Summary:**
Many UI elements in Grafana remain in English when the language is set to Polish. This includes buttons, tooltips, dropdowns, placeholders in sections like Alerting, Dashboards, Admin, and Explore.

**What needs fixing:**

- Identify untranslated strings/keys across the codebase
- Mark them for translation in i18n configuration
- Work with Crowdin or translation system to add Polish translations

**Maintainer guidance (from comment):**

> "Some translations will be updated in a following release. However, some texts still need to be marked up for translation. We've added the `help-wanted` label in case the community wants to contribute to this chore."

**Implementation approach:**

1. Use Polish locale to identify untranslated elements
2. Search codebase for hardcoded English strings in those areas
3. Add translation keys using Grafana's i18n system
4. Test with Polish locale enabled

**Related files:**

- `public/locales/pl/` (Polish translations)
- Look for hardcoded strings in:
  - Alerting UI
  - Dashboards
  - Connections
  - Administration sections

**Skills needed:** i18n/localization, JavaScript, Grafana codebase navigation

**Impact:** Makes Grafana more accessible to Polish-speaking users.

---

### 5. Prometheus: Change regex pattern for regex match queries (#107097)

**Difficulty:** Medium | **Effort:** Small | **Type:** Performance Enhancement

**Status:** Open, unassigned | **Comments:** 1 | **Labels:** good first issue, datasource/Prometheus

**Link:** https://github.com/grafana/grafana/issues/107097

**Summary:**
The Prometheus datasource builds inefficient regex patterns for label matching. Current pattern:

```
label=~".*value1.*|.*value2.*|.*value3.*"
```

Should be:

```
label=~".*(value1|value2|value3).*"
```

This provides a performance benefit for Prometheus query execution.

**What needs fixing:**

- Locate the regex building logic for Prometheus label matching
- Refactor to use the more efficient pattern structure
- Add tests to verify both patterns produce equivalent results
- Ensure no regression in functionality

**Technical context:**

- Prometheus datasource plugin
- Related to regex building for label matching in queries
- Performance optimization

**Skills needed:** TypeScript, Prometheus PromQL, regex patterns

**Impact:** Small but measurable performance improvement for Prometheus queries.

---

## 📚 Documentation & Code Quality

### 6. Prometheus: Update docs to show query builder limitations (#78154)

**Difficulty:** Easy-Medium | **Effort:** Medium-Large | **Type:** Documentation

**Status:** Open, unassigned | **Comments:** 2 | **Labels:** help wanted, type/docs

**Link:** https://github.com/grafana/grafana/issues/78154

**Summary:**
Documentation doesn't clearly communicate that the Prometheus query builder doesn't have full parity with the code editor. There are edge cases due to Lezer updates, new language features, and parsing issues.

**What needs doing:**

- Update Grafana docs to clearly state builder vs code editor differences
- Document edge cases and known limitations
- Update blog posts and marketing materials as needed

**Docs to update:**

- https://grafana.com/blog/2022/07/18/new-in-grafana-9-the-prometheus-query-builder-makes-writing-promql-queries-easier/
- https://grafana.com/docs/grafana/latest/datasources/prometheus/query-editor/#builder-mode

**Skills needed:** Technical writing, Prometheus knowledge, documentation tools

**Related:**

- Epic: https://github.com/grafana/grafana/issues/77879 (planned parity improvements)

**Impact:** Manages customer expectations and reduces confusion.

---

## 🎨 UI/UX Improvements

### 7. Explore: Split view from logs context loses Code vs Builder setting (#95367)

**Difficulty:** Medium | **Effort:** Medium | **Type:** Bug

**Status:** Open, unassigned | **Comments:** 1 | **Labels:** help wanted, bug

**Link:** https://github.com/grafana/grafana/issues/95367

**Summary:**
When opening a split view from log line context menu, the editor mode always defaults to "Code" instead of preserving the user's previous "Builder" selection from the original pane.

**How to reproduce:**

1. Open Explore with Loki datasource
2. Select Builder view
3. Run a search
4. Click context button on a log line
5. Click "Open in Split View"
6. Observe: Split view is now in Code mode (should be Builder)

**What needs fixing:**

- Preserve editor mode preference when creating split view
- Pass editor mode to split view creation logic
- Test with both Builder and Code modes

**Related files:**

- `public/app/features/explore/` (Explore feature)
- Split view/context menu components

**Skills needed:** React, Explore feature knowledge, state management

**Impact:** Improves user experience for log analysis workflows.

---

## 📋 Status & Verification

**Last verified:** November 10, 2025

All issues in this list have been verified to:

- Be open and unassigned on GitHub
- Have no active volunteer comments requesting assignment
- Have no open pull requests actively being worked on
- Have clear descriptions suitable for new contributors

**Finding more opportunities:**
If you'd like to find additional issues, use these GitHub search queries:

```
is:open label:"good first issue" -is:assigned comments:<2
is:open label:"help wanted" -is:assigned comments:<2
```

**Skills needed:** TypeScript, React hooks, Redux, keyboard event handling

**Related files:**

- `public/app/features/explore/ExplorePage.tsx`
- `public/app/features/explore/useKeyboardShortcuts.tsx`
- `public/app/core/services/keybindingsSrv.ts`
- `public/app/features/explore/state/query.ts`

---

### 4. Prometheus: Add configuration option to hide warnings on the UI (#110480)

**Difficulty:** Medium | **Effort:** Small-Medium | **Type:** Feature Request

**Status:** Open, unassigned | **Comments:** 4 | **Engagement:** 2 👍

**Link:** https://github.com/grafana/grafana/issues/110480

**Summary:**
Prometheus shows warnings on the panel UI by default, but not all warnings are useful to users. Add a datasource configuration option to hide warnings. When enabled, remove warnings from backend response (reduces size + improves UX).

**What needs building:**

1. **Backend:** Add config option to Prometheus datasource
2. **Backend:** Filter warnings from response when config is enabled
3. **Frontend:** Show/hide toggle in datasource config UI
4. Minimal frontend changes (warnings already handled by backend)

**Benefits:**

- Reduces response size
- Improves UX for users with noisy warnings
- Per-datasource control

**Skills needed:** Go (backend), TypeScript (frontend), datasource configuration

**Implementation path:**

1. Add config field to Prometheus datasource options struct (Go)
2. Filter warnings in query response handler when flag is set
3. Add UI toggle in datasource configuration (React component)
4. Test with various warning scenarios

**Related files:**

- Prometheus datasource backend handler
- Prometheus datasource frontend config UI

---

### 5. Prometheus: Change regex pattern for regex match queries (#107097)

**Difficulty:** Easy | **Effort:** Small | **Type:** Performance Optimization

**Status:** Open, unassigned | **Comments:** 0 | **Priority:** Good first issue

**Link:** https://github.com/grafana/grafana/issues/107097

**Summary:**
Optimize regex matcher construction for performance. Instead of:

```
label=~".*value1.*|.*value2.*|.*value3.*"
```

Build:

```
label=~".*(value1|value2|value3).*"
```

This is a cleaner regex pattern and has better performance characteristics.

**What needs changing:**

- Find where regex matchers are built
- Refactor to use alternation pattern with grouping
- Add tests for the new pattern

**Skills needed:** Go, regex, Prometheus query building

**Expected impact:** Performance improvement for regex-based label queries

**Related files:**

- Prometheus query builder/formatter
- Label matching logic

---

## 📚 Documentation & Code Improvements

### 6. Prometheus Dashboards: Use `__rate_interval` (#110370)

**Difficulty:** Easy | **Effort:** Very Small | **Type:** Documentation/Config Update

**Status:** Open, unassigned | **Comments:** 3

**Link:** https://github.com/grafana/grafana/issues/110370

**Summary:**
Grafana's built-in Prometheus dashboards use hardcoded `[5m]` intervals instead of the recommended `$__rate_interval` variable. This breaks dashboard responsiveness when users zoom in/out or have different scrape intervals.

**What needs updating:**

1. File: `packages/grafana-prometheus/src/dashboards/prometheus_2_stats.json`
2. Replace all `[5m]` with `[$__rate_interval]`
3. Remove obsolete config:
   - `"intervalFactor": 2` (not needed with `$__rate_interval`)
   - `"step": 20` (not needed)
   - `"interval": ""` (not needed)

**Why this matters:**

- `$__rate_interval` was introduced for exactly this use case
- Makes dashboards adapt to different scrape intervals
- Improves zoom responsiveness

**Skills needed:** JSON editing, Prometheus knowledge, testing dashboards

**How to test:**

1. Load Prometheus dashboard
2. Verify rate() functions use `$__rate_interval`
3. Test zooming in/out
4. Verify with different scrape intervals

**Related files:**

- `packages/grafana-prometheus/src/dashboards/prometheus_2_stats.json`
- Other Prometheus dashboards (may have same issue)

---

### 7. Explore: Use TableNG instead of Table (#111158)

**Difficulty:** Medium | **Effort:** Medium | **Type:** Refactoring / Tech Debt

**Status:** Open, unassigned | **Comments:** 4 | **Priority:** Low

**Link:** https://github.com/grafana/grafana/issues/111158

**Summary:**
Replace old `Table` component with modern `TableNG` in `TableContainer.tsx`. This is a cleanup task to modernize the codebase and use updated components.

**What needs changing:**

- Replace old Table component usage in TableContainer.tsx
- Use TableNG instead (or consider PanelRenderer with timeseries type)
- Ensure feature parity (same behavior, better performance)
- Update any related tests

**Skills needed:** React, TypeScript, Grafana UI components

**Why this matters:**

- TableNG is the modern table implementation
- Reduces maintenance burden
- Likely better performance
- Aligns with current Grafana standards

**Related files:**

- `public/app/features/explore/TableContainer.tsx`
- Table component definitions
- Related tests

---

## 🆕 New Opportunities (Added November 10, 2025)

### 8. Explore: Preserve Code vs Builder preference in split view (#95367)

**Difficulty:** Medium | **Effort:** Medium | **Type:** Bug

**Status:** Open, unassigned | **Comments:** 1 | **Labels:** help wanted

**Link:** https://github.com/grafana/grafana/issues/95367

**Summary:**
When users open a log context popup and click "Open in Split View", the editor mode reverts to Code even if the original pane was using Builder. This breaks user workflow as they need to switch back to Builder mode.

**What needs fixing:**

- Preserve the Code vs Builder query editor preference when opening split view
- Store the user's editor mode choice (Code or Builder)
- Pass this preference when creating the split view panel
- Apply the same preference to the new pane

**Why it matters:**

- Improves UX for users who prefer Builder mode
- Reduces friction during log exploration
- User expectation: split view should preserve original settings

**Skills needed:** TypeScript, React, Loki datasource knowledge, Redux state management

**Implementation path:**

1. Find where split view is triggered from context popup
2. Capture current editor mode from Redux or component state
3. Pass editor mode preference to split view creation
4. Apply preference to new Explore pane
5. Test with both Code and Builder modes

**Related files:**

- `public/app/features/explore/` (Explore feature)
- Context popup component (logs-related)
- Split view logic in Explore

**Similar issues:** #95721 (keyboard focus reset)

---

### 9. Pyroscope: Move "other" out of TOP table with explanation (#110677)

**Difficulty:** Medium | **Effort:** Medium | **Type:** Enhancement

**Status:** Open, unassigned | **Comments:** 3 | **Labels:** good first issue, kind/enhancement

**Link:** https://github.com/grafana/grafana/issues/110677

**Summary:**
The Pyroscope/flamegraph "other" node shows aggregated data that doesn't represent actionable resource consumption, yet it appears prominently in the TOP table, confusing users in large deployments. It should be moved below the table with an explanation.

**What needs building:**

1. **Filter logic**: Remove "other" from the TOP table display
2. **Info section**: Display below table with message:
   - Total value of "other" (self + total)
   - Count of truncated stacktraces
   - Minimum value representation
3. **Interaction**: Keep sandwich view and highlight icons functional for "other"

**Technical context:**

- Pyroscope limits flamegraph to `maxNodes` parameter
- Everything beyond `maxNodes` is aggregated into "other"
- Currently shows mathematically correct but unhelpful data

**Why it matters:**

- Better UX for large/complex profiles
- Clarifies truncation to users
- Helps understand performance bottlenecks more clearly

**Skills needed:** TypeScript, React, Flamegraph visualization, Pyroscope plugin

**Implementation path:**

1. Locate flamegraph table component
2. Add filter to exclude "other" from table rows
3. Create info section below table
4. Calculate and display statistics
5. Ensure icons/actions still work for "other"
6. Add CSS styling for info section

**Related files:**

- Pyroscope plugin flamegraph display
- Table rendering component
- Icons/action handlers

---

### 10. Elasticsearch: Configurable default query mode in Explore (#82812)

**Difficulty:** Easy | **Effort:** Small | **Type:** Feature Request

**Status:** Open, assigned | **Comments:** 2 | **Labels:** good first issue

**Link:** https://github.com/grafana/grafana/issues/82812

**Note:** This issue is assigned to @cauemarcondes but remains open - check if assistance is needed or consider other similar unassigned issues.

**Summary:**
Elasticsearch datasource should allow configuring the default query mode (metrics, logs, raw data, raw document) in Explore. Currently defaults to metrics regardless of user preference, causing friction for teams primarily using logs.

**What needs building:**

1. **Config option**: Add "Default query mode" dropdown to Elasticsearch ConfigEditor
2. **Options**: Metrics (default), Logs, Raw data, Raw document
3. **Apply setting**: Set default query in Explore editor based on config
4. **Backward compatibility**: Maintain metrics as default for new datasources

**Why it matters:**

- Improves UX for teams with specific use cases
- Reduces repeated manual mode switching
- Increases productivity for log analysis workflows

**Skills needed:** TypeScript, React, Elasticsearch datasource plugin, configuration management

**Implementation path:**

1. Add dropdown field to Elasticsearch ConfigEditor component
2. Store selection in datasource options
3. Retrieve setting when initializing Explore
4. Set default query mode based on configuration
5. Test with each mode (metrics, logs, raw data, raw document)

**Related files:**

- `public/app/plugins/datasource/elasticsearch/components/ConfigEditor.tsx`
- Elasticsearch query editor
- Explore initialization logic

---

## 🗓️ Recommended Reading Order

### For **quick wins** (start here):

1. **#110370** (Prometheus dashboards) — straightforward JSON changes
2. **#111664** (Alerting tooltip) — isolated React component fix
3. **#111239** (Prometheus auth cleanup) — focused backend logic

### For **learning the codebase**:

1. **#107097** (Regex optimization) — understand Prometheus query building
2. **#110480** (Hide warnings) — see datasource config pattern
3. **#111158** (TableNG) — learn component replacement patterns

### For **frontend experience**:

1. **#111675** (Keyboard shortcuts) — involves Redux, hooks, and event handling
2. **#111158** (TableNG) — component refactoring practice
3. **#111664** (Tooltip) — form state and event handling

### For **intermediate contributors**:

1. **#95367** (Editor mode preservation) — Redux state management
2. **#110677** (Pyroscope flamegraph) — visualization and UX
3. **#82812** (Elasticsearch defaults) — datasource configuration

---

## 🛠️ Development Workflow

### Before you start:

1. Fork `grafana/grafana` repository
2. Create a feature branch: `git checkout -b fix/issue-XXXXX`
3. Read contributing guidelines: `CONTRIBUTING.md`
4. Check existing discussions on the issue (comments often have context)

### Building and testing locally:

```bash
# Install dependencies
make deps

# Build everything
make build

# Run backend tests (specific area)
make test-go-unit FILES=./pkg/yourpkg

# Run frontend tests
make test-js

# Lint backend
make lint-go

# Run the app
make run
```

### Before submitting PR:

1. Write clear commit messages
2. Add tests for your changes
3. Run linters: `make lint-go` (backend) or `yarn lint` (frontend)
4. Document any new configuration options
5. Reference the issue number in PR description

---

## 📖 Related Resources

- **Grafana Developer Guide:** https://github.com/grafana/grafana/blob/main/contribute/developer-guide.md
- **Build Agent Instructions:** See `AGENTS.md` for CI/build details
- **Contributing Guide:** https://github.com/grafana/grafana/blob/main/CONTRIBUTING.md
- **Grafana Forum:** https://community.grafana.com/
- **Slack (Contributors):** Check grafana/grafana for contributor channels

---

## ✅ Status Tracking

Last verified: **November 10, 2025**

**Verification summary:**

- ✅ All 7 original issues: Open and unassigned (verified)
- ✅ 3 new issues added: #82812, #110677, #95367 (all open/unassigned)
- ✅ Total opportunities: 10 verified contribution opportunities

**What was verified:**

- Issue state (open/closed)
- Assignment status (unassigned)
- Labels (good first issue / help wanted)
- Comment activity and engagement
- Implementation viability

To check current status, visit the GitHub issue directly.

---

## 🤝 Notes for Contributors

- **Start small:** Pick a "good first issue" to learn the codebase
- **Ask questions:** Comment on the issue if anything is unclear
- **Check comments:** Issue discussions often contain implementation hints
- **Test thoroughly:** Both happy path and edge cases
- **Document changes:** Update relevant documentation/comments
- **Be patient:** Code review takes time, but feedback is valuable

Good luck with your contributions! 🚀
