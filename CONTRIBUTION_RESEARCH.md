# Grafana Contribution Research Document

## Overview
This document provides research on potential contribution opportunities in the Grafana project, focusing on issues labeled as "good first issue" and "help wanted". This research was conducted on November 6, 2025.

## Executive Summary
- **Total "help wanted" issues**: 1,171 open issues
- **Total "good first issue"**: 390 open issues
- **Focus areas**: Frontend (TypeScript/React), Backend (Go), Datasources (Prometheus, SQL), Alerting, and UI components

## Recent High-Priority Contribution Opportunities

### 1. Explore: Keyboard Shortcut to Run Queries (#111675)
**Status**: Open | **Labels**: good first issue, help wanted, area/explore
**Difficulty**: Beginner-Intermediate
**Created**: September 26, 2025

#### Description
Add keyboard shortcut to run queries in Explore (similar to `d r` for dashboards). Currently Explore uses generic/global keyboard bindings, but needs Explore-specific shortcuts like `e ?` for running queries.

#### Technical Details
The issue provides excellent technical guidance:
- Keyboard bindings are initialized in `ExplorePage.tsx`
- When `keybindings.setupTimeRangeBindings(false)` is called, keybindingSrv registers shortcuts
- Shortcuts dispatch events via appEvents
- Explore listens to appEvents invoked by keyboard shortcuts

#### Code Structure
```
public/app/features/explore/
├── ExplorePage.tsx                    # Main explore page component
├── hooks/
│   └── useKeyboardShortcuts.ts       # Keyboard shortcuts hook (where changes needed)
└── state/
    └── query.ts                       # Contains runQueries action (line 531)
    └── time.ts                        # Example of similar time range updates
```

#### Implementation Strategy
1. Create a new event type (e.g., `RunQueriesEvent`) in `app/types/events.ts`
2. Add a custom shortcut method in keybindingsSrv (e.g., `keybindingsSrv.customShortcut(shortcut, event)`)
3. Bind the shortcut in `useKeyboardShortcuts.ts` with the `runQueries` action
4. Handle split view scenario (run queries in both panes or add separate shortcuts)

#### Why This is a Good First Issue
- Well-documented with clear technical guidance
- Follows existing patterns in the codebase
- Limited scope with clear boundaries
- Direct impact on user experience

---

### 2. Explore: Use TableNG Instead of Table (#111158)
**Status**: Open | **Labels**: good first issue, help wanted, type/tech, type/refactor
**Difficulty**: Beginner-Intermediate
**Created**: September 16, 2025

#### Description
Replace the old Table component with TableNG (or use PanelRenderer with timeseries type) inside `TableContainer.tsx`.

#### Technical Details
- Currently using deprecated Table component
- Should migrate to newer TableNG component
- This is a technical debt/refactoring issue

#### Code Structure
```
public/app/features/explore/
└── Table/
    └── TableContainer.tsx            # Component using old Table
```

#### Why This is a Good First Issue
- Straightforward component replacement
- Helps clean up technical debt
- Learn about Grafana's component architecture
- Clear before/after comparison

---

### 3. Prometheus: Change Regex Pattern for Regex Match Queries (#107097)
**Status**: Open | **Labels**: good first issue, internal, kind/enhancement
**Difficulty**: Beginner
**Created**: June 24, 2025

#### Description
Optimize regex patterns in Prometheus queries for better performance. Instead of building:
```
label=~".*value1.*|.*value2.*|.*value3.*"
```

Build the matcher as:
```
label=~".*(value1|value2|value3).*"
```

This provides performance benefits by reducing regex complexity.

#### Code Structure
```
packages/grafana-prometheus/src/
├── language_utils.ts                 # Query building utilities
├── language_utils.test.ts           # Tests for query utilities
└── querybuilder/                    # Query builder components
```

#### Implementation Strategy
1. Locate regex pattern building code in query builder
2. Modify pattern generation logic to group values
3. Add comprehensive tests for the new pattern
4. Ensure backward compatibility

#### Why This is a Good First Issue
- Clear performance improvement
- Well-defined scope
- Good opportunity to learn about regex optimization
- Impact on query performance

---

### 4. Prometheus: Recording Rules Expansion with Empty Label Values (#108600)
**Status**: Closed (but good learning example) | **Labels**: good first issue, datasource/Prometheus
**Difficulty**: Beginner-Intermediate

#### Description
Fix recording rule expansion when there are empty label matchers. The `addLabelToQuery` function doesn't expect empty values and fails.

#### Code Structure
```
packages/grafana-prometheus/src/
├── add_label_to_query.ts            # Lines 24-26 check for empty labels
├── add_label_to_query.test.ts       # Tests
└── language_utils.ts                # Recording rules expansion logic (line 188-206)
```

#### Key Code Locations
1. **add_label_to_query.ts (lines 24-26)**: The validation that needs adjustment
   ```typescript
   if (!key) {
     throw new Error('Need label to add to query.');
   }
   ```

2. **language_utils.ts**: The `expandRecordingRules` function that processes rules

#### Implementation Details
The issue provides a unit test to verify the fix:
```typescript
it('when there is an empty label value it should still be able to expand the rule', () => {
  const query = `sum(max by (cluster, container) (pod_cpu:active:kube_limits{container!="", cluster=~"pink"}))`;
  const mapping = {
    'pod_cpu:active:kube_limits': {
      expandedQuery: `kube_limits{job!="", resource="cpu"} * on (namespace, pod, cluster) group_left () max by (namespace, pod, cluster) ((kube_pod_status_phase{phase=~"Pending|Running"} == 1))`,
    },
  };
  const expected = `sum(max by (cluster, container) (kube_limits{job!="", resource="cpu", container!="", cluster=~"pink"} * on (namespace, pod, cluster) group_left () max by (namespace, pod, cluster) ((kube_pod_status_phase{phase=~"Pending|Running", container!="", cluster=~"pink"} == 1))))`;
  const result = expandRecordingRules(query, mapping);
  expect(result).toBe(expected);
});
```

#### Why This is a Good Learning Example
- Demonstrates how to handle edge cases
- Shows interaction between multiple modules
- Includes clear test case
- Real-world bug fix scenario

---

### 5. Alerting: Options Tooltip Does Not Save Interval Value (#111664)
**Status**: Open | **Labels**: good first issue, type/bug, area/alerting, area/frontend
**Difficulty**: Beginner
**Created**: September 26, 2025

#### Description
When editing alert configuration, changing the interval value in the options tooltip doesn't save if you click outside the tooltip. The `onBlurEvent` is not triggered when the tooltip closes.

#### Technical Details
- Issue affects alert creation/edit page
- Tooltip doesn't save interval or max data points when clicking outside
- Need to handle tooltip close event properly

#### Why This is a Good First Issue
- Clear UI bug with reproducible steps
- Frontend-focused (TypeScript/React)
- Good introduction to form handling in Grafana
- Visible impact on user experience

---

### 6. VQB: Add Custom Table Name Manually (#106348)
**Status**: Closed (but informative) | **Labels**: good first issue, help wanted
**Difficulty**: Beginner
**Created**: June 4, 2025

#### Description
Allow users to add custom table names that are not in the list for SQL visual query builder.

#### Implementation
In `TableSelector.tsx`, add the property:
- `createCustomValue` if component is `Combobox`
- `allowCustomValue` if it is still `Select`

#### Code Structure
```
public/app/plugins/datasource/*/
└── components/
    └── TableSelector.tsx             # Component to modify
```

#### Why This is Instructive
- Shows how to enable custom values in dropdowns
- Applies to MySQL, PostgreSQL, and MSSQL datasources
- Simple property addition with big UX impact

---

### 7. CloudWatch: EMRServerless Job Level Metrics Dimensions Missing (#104062)
**Status**: Closed | **Labels**: good first issue, datasource/CloudWatch, effort/small
**Difficulty**: Intermediate
**Created**: April 15, 2025

#### Description
Add support for EMRServerless Job Level and Job Worker Level metric dimensions in CloudWatch plugin. Currently only Application level dimensions are populated.

#### Technical Details
- Need to populate JobId dimension keys and values
- Affects variables and query builder
- CloudWatch Insights Metrics support

#### Why This is Notable
- Shows how to extend datasource capabilities
- Involves understanding AWS CloudWatch metrics
- Good for learning datasource plugin architecture

---

## Additional Promising Issues

### Frontend/UI Components
1. **Combobox: Add Search Match Highlighting** (#99491)
   - Implement fuzzy search highlighting using uFuzzy library
   - Good for learning about search UX patterns

2. **Modals: Semi-transparent Border Issue** (#102190)
   - CSS/styling fix for modal borders
   - Small effort, visual impact
   - Labels: effort/small, prio/low, area/grafana/ui

3. **Dashboard: Keyboard Focus Reset When Deleting Panel** (#95721)
   - Accessibility improvement
   - Focus management in React
   - Labels: effort/small, type/accessibility

### Backend/Go
1. **Plugins Installer: Inherit Permissions** (#107678)
   - File permissions handling in Go
   - Good for pair programming
   - Labels: 🍐-programming

2. **Tooling: Update Workspace Target Issues** (#97840)
   - Build system / Makefile improvements
   - Good for understanding Go modules

### Datasources
1. **Prometheus: Hide Warnings Configuration** (#110480)
   - Add configuration option to hide warnings
   - Backend + Frontend work

2. **CloudWatch: Various improvements** (multiple issues)
   - Good for AWS ecosystem knowledge

### Documentation
1. **UPGRADING_DEPENDENCIES Guide** (#97863, #97842)
   - Improve contributor documentation
   - Low code, high impact on DX

---

## Issue Categories by Difficulty

### Beginner (No prior Grafana knowledge needed)
- Prometheus regex pattern optimization (#107097)
- Documentation improvements (#97863, #97842)
- CSS/styling fixes (#102190)
- Simple property additions (#106348)

### Beginner-Intermediate (Some Grafana knowledge helpful)
- Keyboard shortcuts (#111675)
- Component migrations (#111158)
- Alerting UI bugs (#111664)
- Recording rules expansion (#108600)

### Intermediate (Domain knowledge beneficial)
- CloudWatch dimensions (#104062)
- Complex UI components (Combobox highlighting)
- Backend permissions handling (#107678)

---

## Code Structure Overview

### Key Directories for Contributors

#### Frontend (TypeScript/React)
```
public/app/
├── features/
│   ├── explore/              # Explore feature
│   │   ├── hooks/           # React hooks
│   │   ├── state/           # Redux state management
│   │   └── components/      # UI components
│   ├── alerting/            # Alerting feature
│   └── dashboard/           # Dashboard feature
└── plugins/
    └── datasource/          # Datasource plugins
        ├── prometheus/
        ├── cloudwatch/
        └── ...

packages/
├── grafana-ui/              # UI component library
├── grafana-data/            # Data models and utilities
└── grafana-prometheus/      # Prometheus-specific code
```

#### Backend (Go)
```
pkg/
├── api/                     # REST API endpoints
├── services/               # Business logic services
├── plugins/                # Plugin system
└── datasources/            # Datasource implementations
```

#### Testing
```
- Frontend: .test.ts, .test.tsx files alongside components
- Backend: _test.go files
- E2E: e2e/ and e2e-playwright/ directories
```

---

## Contribution Workflow

### Getting Started
1. Read `CONTRIBUTING.md` for general guidelines
2. Review `contribute/developer-guide.md` for setup
3. Check style guides:
   - Frontend: `contribute/style-guides/frontend.md`
   - Backend: `contribute/backend/style-guide.md`
4. Understand testing: `contribute/style-guides/testing.md`

### Development Process
1. Fork and clone the repository
2. Create a feature branch
3. Make changes following style guides
4. Write/update tests
5. Run linters and tests locally
6. Create pull request following `contribute/create-pull-request.md`
7. Sign CLA if first contribution

### Build and Test Commands
```bash
# Install dependencies
yarn install

# Build frontend
yarn start

# Build backend
make run

# Run frontend tests
yarn test

# Run backend tests
make test

# Run linters
yarn lint
make lint-go
```

---

## Documentation Structure Around Issues

### Issue #111675 (Keyboard Shortcuts)

#### Architecture Pattern: Event-Driven Shortcuts
```
User Presses Key
    ↓
keybindingSrv.bind()
    ↓
Event Dispatcher (appEvents)
    ↓
Event Subscription (getAppEvents().subscribe)
    ↓
Redux Action Dispatch
    ↓
State Update
```

#### File Dependencies
1. **Event Definition**: `app/types/events.ts`
   - Defines event types like `AbsoluteTimeEvent`, `ShiftTimeEvent`
   - Need to add `RunQueriesEvent` here

2. **Keybinding Service**: `app/core/services/keybindingSrv`
   - Registers keyboard shortcuts
   - Binds keys to events

3. **Hook Setup**: `public/app/features/explore/hooks/useKeyboardShortcuts.ts`
   - Subscribes to events
   - Dispatches Redux actions

4. **Redux Actions**: `public/app/features/explore/state/query.ts`
   - Contains `runQueries` thunk (starting line 531)
   - Handles query execution logic

#### Similar Implementations to Reference
- Time range operations in `useKeyboardShortcuts.ts` (lines 26-54)
- Time range updates in `state/time.ts` for multi-pane handling

---

### Issue #108600 (Recording Rules with Empty Labels)

#### Architecture Pattern: Query Transformation Pipeline
```
Raw Query String
    ↓
Parser (lezer-promql)
    ↓
Vector Selector Positions
    ↓
Visual Query Builder
    ↓
Add Label Filters
    ↓
Render Query String
```

#### File Dependencies
1. **Label Addition**: `packages/grafana-prometheus/src/add_label_to_query.ts`
   - Lines 24-26: Validation that needs adjustment
   - Currently throws error on empty key
   - Should handle empty values gracefully

2. **Recording Rules**: `packages/grafana-prometheus/src/language_utils.ts`
   - Line 188-206: `expandRecordingRules` function
   - Processes label objects and adds them to query
   - Calls `addLabelToQuery` for each label

3. **Query Parsing**: Uses `@prometheus-io/lezer-promql` parser
   - Identifies VectorSelector nodes
   - Extracts label matchers

#### Key Insight
The issue is that empty label values (like `container!=""`) are valid in PromQL for filtering out empty values, but the validation in `addLabelToQuery` doesn't account for this use case.

---

## Technology Stack Summary

### Frontend
- **Framework**: React 18.x
- **Language**: TypeScript
- **State Management**: Redux with Redux Toolkit
- **Styling**: Emotion (CSS-in-JS)
- **Testing**: Jest, React Testing Library
- **Build**: Webpack, Yarn

### Backend
- **Language**: Go 1.22+
- **Testing**: Standard Go testing package
- **Build**: Makefile, Go modules

### Key Libraries
- **@grafana/ui**: Component library
- **@grafana/data**: Data models
- **@grafana/runtime**: Runtime utilities
- **@prometheus-io/lezer-promql**: PromQL parser

---

## Tips for Contributors

### Finding Your First Issue
1. Start with "good first issue" label
2. Look for issues with detailed technical descriptions
3. Check for recent activity and maintainer engagement
4. Prefer issues in your area of expertise (frontend/backend)

### Before Starting Work
1. Comment on the issue expressing interest
2. Ask clarifying questions if needed
3. Check if anyone else is already working on it
4. Understand the acceptance criteria

### Writing Good PRs
1. Reference the issue number
2. Provide clear description of changes
3. Include screenshots for UI changes
4. Add tests for new functionality
5. Follow existing code patterns
6. Keep changes focused and minimal

### Getting Help
- **Forums**: https://community.grafana.com/
- **Slack**: https://slack.grafana.com
- **GitHub Discussions**: For architecture questions
- **Issue Comments**: For specific issue questions

---

## Recent Trends in Contributions

### Areas of Active Development (Based on Recent Issues)
1. **Prometheus Datasource**: Multiple issues around query building, recording rules
2. **Explore Feature**: Keyboard shortcuts, table components, UI improvements
3. **Alerting**: Configuration issues, UI bugs
4. **SQL Datasources**: Visual query builder improvements
5. **CloudWatch**: Metrics dimensions, configuration
6. **Accessibility**: Focus management, WCAG compliance
7. **Performance**: Regex optimization, component upgrades

### Common Patterns in Good First Issues
1. Component migrations (old → new)
2. Configuration additions
3. UI bug fixes
4. Documentation improvements
5. Test additions
6. Accessibility improvements

---

## Conclusion

The Grafana project offers diverse contribution opportunities across frontend, backend, documentation, and testing. The issues are well-documented, maintainers are responsive, and there's a clear path from issue to merged PR.

**Recommended Starting Points**:
1. For **Frontend**: Keyboard shortcuts (#111675) or component migration (#111158)
2. For **Backend**: Plugins permissions (#107678)
3. For **Documentation**: Dependency upgrade guides (#97863, #97842)
4. For **Datasources**: Prometheus regex optimization (#107097)

All of these issues have clear technical descriptions, manageable scope, and direct impact on user experience.

---

## Next Steps for This Research

This document is meant to be a living resource. Future updates could include:
1. More detailed architecture diagrams
2. Video walkthroughs of contribution process
3. Curated learning paths for different expertise levels
4. Interview with successful contributors
5. Analysis of merged PRs to identify patterns

**Note**: This research is for personal exploration and won't be pushed to grafana/grafana.
