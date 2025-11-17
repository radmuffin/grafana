# Deep Dive: Keyboard Focus Reset on Panel Delete

**Issue**: [#95721 - Dashboard: Keyboard focus is reset when deleting a panel](https://github.com/grafana/grafana/issues/95721)

**Effort**: Small  
**Type**: Accessibility (WCAG 2.1.1 - Keyboard Accessible)  
**Labels**: `good first issue`, `help wanted`, `accessibility`  
**Status**: Open, unassigned

---

## 1. Problem Statement

When users navigate a Grafana dashboard using keyboard (Tab key), focus moves sequentially through interactive elements like panel headers and menu buttons. However, when a user deletes a panel using keyboard navigation, the browser's focus is completely reset instead of moving to an adjacent panel or the next logical element.

### Reproduction Steps

1. Open a Grafana dashboard with multiple panels
2. Press **Tab** repeatedly to navigate through the dashboard
3. When focus reaches a panel header, press **Enter** or **Alt+D** to open the panel menu
4. Select **Delete** from the menu
5. Confirm the deletion in the modal dialog
6. **Observe**: Focus is lost (usually returns to browser's default, often the page top or body element)
7. **Expected**: Focus should move to the next or previous panel

### Current Behavior

- User loses keyboard focus context after panel deletion
- Must start tabbing from the beginning again
- Poor accessibility experience (violates WCAG 2.1.1 Keyboard)
- Especially problematic on dashboards with many panels

### Why This Matters (Accessibility)

- **WCAG 2.1.1 - Keyboard Accessible**: All functionality available via keyboard
- Users with motor disabilities rely on keyboard navigation
- Screen reader users navigate via keyboard
- Keyboard power users expect predictable focus behavior

---

## 2. Root Cause Analysis

### The Deletion Flow

The deletion process follows this chain:

```
User clicks/presses "Delete" menu item
    ↓
getPanelMenu.ts: onRemovePanel() called
    ↓
utils/panel.ts: removePanel() function
    ↓
DashboardModel.removePanel(panel)
    ↓
panel.destroy() + panels array filtered
    ↓
DashboardPanelsChangedEvent published
    ↓
DashboardGrid.tsx receives event & re-renders
    ↓
🔴 PROBLEM: DOM element deleted but focus not managed
```

### File Locations

| File              | Location                                    | Role                                |
| ----------------- | ------------------------------------------- | ----------------------------------- |
| **Menu trigger**  | `dashgrid/PanelHeader/PanelHeaderMenu.tsx`  | Renders menu items                  |
| **Menu handler**  | `utils/getPanelMenu.ts` (line 83)           | `onRemovePanel()` function          |
| **Delete logic**  | `utils/panel.ts` (line 20)                  | `removePanel()` utility             |
| **Model layer**   | `state/DashboardModel.ts` (line 905)        | `removePanel()` method              |
| **Grid layout**   | `dashgrid/DashboardGrid.tsx`                | Re-renders on panel change          |
| **Panel wrapper** | `dashgrid/PanelStateWrapper.tsx` (line 593) | Panel rendering with `onFocus` hook |

### The Root Cause: Three Missing Pieces

#### 1. **No Focus Tracking**

Currently, no code tracks which panel element has focus:

```typescript
// ❌ Current code: utils/panel.ts (line 20-47)
export const removePanel = (dashboard: DashboardModel, panel: PanelModel, ask: boolean) => {
  if (ask !== false) {
    appEvents.publish(
      new ShowConfirmModalEvent({
        title: 'Remove panel',
        yesText: 'Remove',
        onConfirm: () => removePanel(dashboard, panel, false),
      })
    );
    return;
  }

  dashboard.removePanel(panel); // ← Panel deleted here
  dispatch(cleanUpPanelState(panel.key));
  // ❌ NO CODE TO RESTORE FOCUS
};
```

#### 2. **Event-Driven Grid Re-render**

When `removePanel()` is called:

```typescript
// DashboardModel.ts (line 905-909)
removePanel(panel: PanelModel) {
  this.panels = this.panels.filter((item) => item !== panel);
  panel.destroy();
  this.events.publish(new DashboardPanelsChangedEvent());  // ← Triggers re-render
}
```

The `DashboardPanelsChangedEvent` triggers a complete grid re-render. The DOM element with focus is destroyed before focus can be transferred.

#### 3. **React Lifecycle Issue**

`DashboardGrid.tsx` extends `PureComponent` and uses ReactGridLayout. When the panels array changes:

```typescript
// Simplified: DashboardGrid.tsx renders
buildLayout() {
  // Rebuilds layout array from dashboard.panels
  // ❌ Old panel's DOM element is deleted
  // ❌ No focus restoration logic
}
```

The component re-renders, but no code manages where focus should go.

### Why Focus Is Lost

```
Deletion flow:
┌─────────────────────────────────────────┐
│ 1. User has focus on panel element      │
│    <div class="panel" tabIndex={0}>     │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ 2. removePanel() called                 │
│    → DashboardModel.removePanel()       │
│    → panels array filtered              │
│    → DashboardPanelsChangedEvent        │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ 3. DashboardGrid re-renders             │
│    → Old DOM deleted                    │
│    → New DOM created (without old item) │
│    ❌ FOCUS IS LOST HERE ❌             │
│    Focus reverts to <body>              │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ 4. User must re-establish focus         │
│    Tab key doesn't work intuitively     │
└─────────────────────────────────────────┘
```

---

## 3. Component Interaction Diagram

```
┌─────────────────────────────────────────────────────┐
│                DashboardGrid                        │
│  (PureComponent managing panel layout)              │
│                                                     │
│  panels: [panel1, panel2, panel3]                  │
│  buildLayout() → ReactGridLayout                    │
│                                                     │
│  ↑ subscribes to:                                   │
│  └─ DashboardPanelsChangedEvent                     │
└──────────────────┬──────────────────────────────────┘
                   │
      ┌────────────┼────────────┐
      ↓            ↓            ↓
   ┌────────┐  ┌────────┐  ┌────────┐
   │Panel 1 │  │Panel 2 │  │Panel 3 │
   │(JSX)   │  │(JSX)   │  │(JSX)   │
   └───┬────┘  └───┬────┘  └───┬────┘
       │           │           │
       ↓           ↓           ↓
   ┌────────────────────────────────┐
   │ DashboardPanel (wrapper)       │
   │  → PanelStateWrapper           │
   │    → PanelChrome               │
   │      → PanelHeader             │
   │        → PanelHeaderMenu ◄─────┼── FOCUS LOST HERE
   │          (delete button)       │
   └────────────────────────────────┘

WHEN PANEL DELETED:
1. removePanel() → DashboardModel.removePanel()
2. DashboardPanelsChangedEvent published
3. DashboardGrid.buildLayout() recalculates
4. React re-renders: JSX array filters out deleted panel
5. Old panel's DOM destroyed
6. NO CODE: transfers focus to next/previous panel
7. Browser focus returns to document body
```

---

## 4. Potential Solution Approaches

### Approach 1: Focus Management in removePanel Utility (Recommended)

**Pros**:

- Centralized logic (single point of change)
- Works with all deletion methods (menu, keyboard shortcut, programmatic)
- Reusable for other removal scenarios (rows, etc.)
- Simple to test

**Cons**:

- Requires knowing adjacent panel index before deletion
- Must handle edge cases (last panel, single panel, etc.)

**Implementation Outline**:

```typescript
// utils/panel.ts - Enhanced removePanel()

export const removePanel = (dashboard: DashboardModel, panel: PanelModel, ask: boolean) => {
  if (ask !== false) {
    appEvents.publish(
      new ShowConfirmModalEvent({
        title: 'Remove panel',
        yesText: 'Remove',
        onConfirm: () => removePanel(dashboard, panel, false),
      })
    );
    return;
  }

  // 🟢 NEW: Calculate focus target BEFORE deletion
  const panelIndex = dashboard.panels.indexOf(panel);
  let focusTargetPanelId: number | null = null;

  // Try next panel, then previous, then dashboard
  if (panelIndex < dashboard.panels.length - 1) {
    // Next panel exists
    focusTargetPanelId = dashboard.panels[panelIndex + 1].id;
  } else if (panelIndex > 0) {
    // Previous panel exists
    focusTargetPanelId = dashboard.panels[panelIndex - 1].id;
  }

  // Perform deletion
  dashboard.removePanel(panel);
  dispatch(cleanUpPanelState(panel.key));

  // 🟢 NEW: Restore focus after deletion
  if (focusTargetPanelId !== null) {
    // Defer focus to next render cycle
    setTimeout(() => {
      const targetElement = document.querySelector(`[data-panel-id="${focusTargetPanelId}"] [tabIndex="0"]`);
      if (targetElement) {
        (targetElement as HTMLElement).focus();
      }
    }, 0);
  }
};
```

**Key Considerations**:

- Use `data-panel-id` attribute for reliable element selection
- Focus after DOM has updated (setTimeout or requestAnimationFrame)
- Handle panels without focusable elements
- Test with multiple panels and edge cases

---

### Approach 2: DashboardGrid-Level Focus Management

**Pros**:

- Handles focus at component level (React-native)
- Can integrate with component lifecycle
- Could manage focus during other grid changes (resize, reorder)

**Cons**:

- More complex (touches grid component)
- Requires ref management
- Harder to test

**Implementation Outline**:

```typescript
// dashgrid/DashboardGrid.tsx - Add focus tracking

export class DashboardGrid extends PureComponent<Props, State> {
  private panelRefs = new Map<number, HTMLDivElement>();
  private focusedPanelId: number | null = null;

  componentDidMount() {
    // Track panel events
    this.dashboard.events.subscribe(
      DashboardPanelsChangedEvent,
      this.onPanelChange
    );
  }

  onPanelChange = () => {
    // After grid re-render, restore focus
    if (this.focusedPanelId !== null) {
      const nextPanel = this.getAdjacentPanel(this.focusedPanelId);
      if (nextPanel && this.panelRefs.has(nextPanel.id)) {
        const ref = this.panelRefs.get(nextPanel.id);
        ref?.focus();
      }
    }
  };

  render() {
    return (
      <ReactGridLayout>
        {this.dashboard.panels.map((panel) => (
          <div
            key={panel.key}
            ref={(el) => {
              if (el) this.panelRefs.set(panel.id, el);
            }}
          >
            <DashboardPanel panel={panel} />
          </div>
        ))}
      </ReactGridLayout>
    );
  }
}
```

---

### Approach 3: Panel Focus Restoration with useRef Hook

**Pros**:

- Leverages React hooks (modern approach)
- Can track focus state in component
- Integrates with panel lifecycle

**Cons**:

- Requires refactoring to functional component or custom hooks
- Multiple components involved
- More invasive changes

**Implementation Outline**:

```typescript
// Custom hook for panel focus management
function usePanelFocus(panelId: number) {
  const panelRef = useRef<HTMLDivElement>(null);
  const previousIdRef = useRef(panelId);

  useEffect(() => {
    // If panel was deleted, focus nearby element
    if (previousIdRef.current !== panelId) {
      const nextFocusable = document.querySelector(
        `.dashboard-panel[data-panel-id]:not([data-deleted])`
      ) as HTMLElement;
      nextFocusable?.focus();
    }
    previousIdRef.current = panelId;
  }, [panelId]);

  return panelRef;
}

// In PanelStateWrapper
export class PanelStateWrapper extends PureComponent<Props, State> {
  handlePanelDeletion = () => {
    const nextPanel = this.getNextPanel();
    if (nextPanel?.ref) {
      nextPanel.ref.focus();
    }
  };
}
```

---

### Approach 4: Enhanced PanelHeaderMenu with Focus Context

**Pros**:

- Minimal changes to existing code
- Encapsulates focus logic with menu interaction
- Clear separation of concerns

**Cons**:

- Only handles menu-based deletion
- Duplicates some logic from Approach 1

**Implementation Outline**:

```typescript
// dashgrid/PanelHeader/PanelHeaderMenu.tsx - Enhanced

export function PanelHeaderMenu({ items, panel, dashboard }: Props) {
  const handleDeleteClick = (item: PanelMenuItem) => {
    if (item.text.toLowerCase() === 'delete') {
      // Capture focus context before deletion
      const panelIndex = dashboard.panels.indexOf(panel);
      const focusTarget =
        panelIndex < dashboard.panels.length - 1 ? dashboard.panels[panelIndex + 1] : dashboard.panels[panelIndex - 1];

      // Execute delete and restore focus
      if (item.onClick) {
        item.onClick();

        setTimeout(() => {
          const targetElement = document.querySelector(`[data-panel-id="${focusTarget?.id}"]`) as HTMLElement;
          targetElement?.focus();
        }, 0);
      }
    } else {
      item.onClick?.();
    }
  };

  // Rest of component...
}
```

---

## 5. Recommended Solution: Approach 1

**Why Approach 1 is Best**:

1. **Minimal Impact**: Changes only `utils/panel.ts` (existing removal logic)
2. **Single Source of Truth**: All panel removals go through this function
3. **Testable**: Can unit test focus logic independently
4. **Accessible**: Works with keyboard, mouse, and programmatic deletion
5. **Edge Cases Handled**: Manages last panel, single panel scenarios
6. **Performant**: Simple DOM queries with `data-panel-id`

---

## 6. Implementation Steps

### Step 1: Add Data Attributes to Panels

Ensure panels have reliable selectors:

```typescript
// dashgrid/DashboardPanel.tsx

return (
  <div
    data-panel-id={panel.id}
    className="dashboard-panel"
  >
    {/* Panel content */}
  </div>
);
```

### Step 2: Modify removePanel Function

Update `utils/panel.ts` to implement focus restoration:

```typescript
export const removePanel = (dashboard: DashboardModel, panel: PanelModel, ask: boolean) => {
  if (ask !== false) {
    appEvents.publish(
      new ShowConfirmModalEvent({
        title: 'Remove panel',
        yesText: 'Remove',
        onConfirm: () => removePanel(dashboard, panel, false),
      })
    );
    return;
  }

  // Calculate focus target before deletion
  const panelIndex = dashboard.panels.indexOf(panel);
  let focusTargetId: number | null = null;

  if (panelIndex < dashboard.panels.length - 1) {
    focusTargetId = dashboard.panels[panelIndex + 1].id;
  } else if (panelIndex > 0) {
    focusTargetId = dashboard.panels[panelIndex - 1].id;
  }

  // Perform deletion
  dashboard.removePanel(panel);
  dispatch(cleanUpPanelState(panel.key));

  // Restore focus after re-render
  if (focusTargetId !== null) {
    requestAnimationFrame(() => {
      const focusableElement = document.querySelector(
        `[data-panel-id="${focusTargetId}"] button, [data-panel-id="${focusTargetId}"] [role="button"]`
      ) as HTMLElement;

      if (focusableElement) {
        focusableElement.focus();
      }
    });
  }
};
```

### Step 3: Handle Edge Cases

```typescript
// Consider for edge cases:
// 1. Collapsed rows (no visible panels)
// 2. Hidden panels (display: none)
// 3. Panels in edit mode
// 4. Multiple deletions in rapid succession
// 5. Deletion via keyboard shortcut vs menu

const getFocusableElement = (element: HTMLElement): HTMLElement | null => {
  // Find first focusable element in panel
  const focusable = element.querySelector('button, [tabindex]:not([tabindex="-1"]), a') as HTMLElement | null;
  return focusable;
};
```

### Step 4: Add Accessibility Attributes

Enhance panels with ARIA labels for better screen reader support:

```typescript
// dashgrid/DashboardPanel.tsx

<div
  data-panel-id={panel.id}
  role="region"
  aria-label={`Panel: ${panel.title || 'Untitled'}`}
  onFocus={() => this.setPanelAttention()}
>
  {/* Content */}
</div>
```

---

## 7. Testing Strategy

### Unit Tests

```typescript
// utils/panel.test.ts

describe('removePanel with focus management', () => {
  it('should focus next panel when deleting middle panel', () => {
    const dashboard = createTestDashboard([panel1, panel2, panel3]);
    const mockFocus = jest.fn();

    // Setup DOM
    const panel2Element = document.createElement('div');
    panel2Element.setAttribute('data-panel-id', '2');
    const focusButton = document.createElement('button');
    panel2Element.appendChild(focusButton);
    panel2Element.focus = mockFocus;

    removePanel(dashboard, panel2, false);

    // Panel3 should receive focus
    expect(mockFocus).toHaveBeenCalledWith(panel3Element);
  });

  it('should focus previous panel when deleting last panel', () => {
    const dashboard = createTestDashboard([panel1, panel2, panel3]);
    removePanel(dashboard, panel3, false);
    expect(focusSpy).toHaveBeenCalledWith(panel2Element);
  });

  it('should handle deletion of only panel gracefully', () => {
    const dashboard = createTestDashboard([panel1]);
    removePanel(dashboard, panel1, false);
    // No focus should be set
    expect(focusSpy).not.toHaveBeenCalled();
  });
});
```

### Integration Tests (Cypress)

```javascript
// cypress/integration/dashboard_focus.spec.js

describe('Dashboard keyboard focus management', () => {
  beforeEach(() => {
    cy.visit('/d/test-dashboard');
    cy.get('.dashboard-panel').should('have.length', 3);
  });

  it('should restore focus to next panel after deletion', () => {
    // Navigate to panel 2 header
    cy.get('[data-panel-id="2"] button:first').focus();
    cy.get('[data-panel-id="2"] button:first').should('have.focus');

    // Delete panel
    cy.get('[data-panel-id="2"] .panel-menu-button').click();
    cy.get('[data-test="delete-panel"]').click();
    cy.get('.modal-content button:contains("Delete")').click();

    // Focus should move to panel 3
    cy.get('[data-panel-id="3"] button:first').should('have.focus');
    cy.get('[data-panel-id="2"]').should('not.exist');
  });

  it('should restore focus to previous panel when deleting last panel', () => {
    cy.get('[data-panel-id="3"] button:first').focus();
    // Delete panel 3
    cy.get('[data-panel-id="3"] .panel-menu-button').click();
    cy.get('[data-test="delete-panel"]').click();
    cy.get('.modal-content button:contains("Delete")').click();

    // Focus should move to panel 2
    cy.get('[data-panel-id="2"] button:first').should('have.focus');
  });

  it('should be keyboard navigable after deletion', () => {
    // Start at panel 1
    cy.get('[data-panel-id="1"] button:first').focus();

    // Tab to panel 2, delete it
    cy.get('body').tab();
    cy.focused().parent().should('have.attr', 'data-panel-id', '2');
    cy.get('[data-panel-id="2"] .panel-menu-button').click();
    cy.get('[data-test="delete-panel"]').click();
    cy.get('.modal-content button:contains("Delete")').click();

    // Tab should continue smoothly to panel 3
    cy.get('body').tab();
    cy.focused().parent().should('have.attr', 'data-panel-id', '3');
  });
});
```

### Accessibility Tests (Axe, Pa11y)

```typescript
// e2e/a11y/dashboard_focus_a11y.spec.ts

describe('Dashboard focus management - Accessibility', () => {
  it('should not create focus management violations', () => {
    cy.visit('/d/test-dashboard');

    // Delete a panel
    cy.get('[data-panel-id="2"] .panel-menu-button').click();
    cy.get('[data-test="delete-panel"]').click();
    cy.get('.modal-content button:contains("Delete")').click();

    // Run accessibility audit
    cy.injectAxe();
    cy.checkA11y();

    // Verify focus is on a valid element
    cy.focused().should('not.have.css', 'display', 'none');
    cy.focused()
      .should('have.attr', 'tabindex')
      .then((tabindex) => {
        expect(parseInt(tabindex)).toBeGreaterThanOrEqual(0);
      });
  });

  it('should maintain keyboard navigation context', () => {
    // Tab through panels
    cy.get('[data-panel-id="1"]').focus();
    cy.get('body').tab().tab(); // Move to panel 2

    // Delete current panel
    cy.get('[data-panel-id="2"] .panel-menu-button').click();
    cy.get('[data-test="delete-panel"]').click();
    cy.get('.modal-content button:contains("Delete")').click();

    // Should be able to continue tabbing
    cy.focused().should('be.visible');
    cy.get('body').tab();
    cy.focused().should('not.equal', document.body);
  });
});
```

### Manual Testing Checklist

- [ ] Tab through dashboard with 3+ panels
- [ ] Delete first panel → focus should move to panel 2
- [ ] Delete middle panel → focus should move to next panel
- [ ] Delete last panel → focus should move to previous panel
- [ ] Delete only panel → focus should be neutral (not lost)
- [ ] Delete via menu (mouse) → focus restored
- [ ] Delete via keyboard → focus restored
- [ ] Test with collapsed rows
- [ ] Test with hidden panels
- [ ] Test with single-panel dashboard
- [ ] Verify screen reader announces focus change
- [ ] Test with keyboard-only navigation (no mouse)

---

## 8. Files to Modify

| File                                          | Change                                         | Impact                              |
| --------------------------------------------- | ---------------------------------------------- | ----------------------------------- |
| `utils/panel.ts`                              | Add focus restoration logic to `removePanel()` | **Medium** - Core logic change      |
| `dashgrid/DashboardPanel.tsx`                 | Add `data-panel-id` attribute                  | **Low** - Selector only             |
| `dashgrid/PanelStateWrapper.tsx`              | Add `role="region"` and `aria-label`           | **Low** - Accessibility enhancement |
| `utils/panel.test.ts`                         | Add tests for focus restoration                | **Low** - Tests only                |
| `cypress/integration/dashboard_focus.spec.js` | Add E2E tests                                  | **Low** - Tests only                |

---

## 9. Related Code References

### Event System

```typescript
// Deletion publishes this event, triggering grid re-render
dashboard.events.publish(new DashboardPanelsChangedEvent());

// DashboardGrid subscribes:
// public/app/features/dashboard/dashgrid/DashboardGrid.tsx (line ~79)
```

### Panel Selection

```typescript
// Current panel tracking (existing)
this.focusedPanel = panel;  // Used in PanelStateWrapper
onFocus={() => this.setPanelAttention()}  // Line 593
```

### DOM Structure

```html
<!-- Expected panel structure -->
<div data-panel-id="2" class="dashboard-panel">
  <div class="panel-header">
    <button class="panel-menu-button">⋮</button>
  </div>
  <div class="panel-content">
    <!-- Panel visualization -->
  </div>
</div>
```

---

## 10. Accessibility Standards Compliance

### WCAG 2.1 Compliance

- **2.1.1 Keyboard (Level A)**: All functionality available via keyboard ✓
- **2.4.3 Focus Order (Level A)**: Focus visible and moves logically ✓
- **2.4.7 Focus Visible (Level AA)**: Focus indicator visible ✓
- **3.2.2 On Input (Level A)**: No unexpected context change on panel delete ✓

### Screen Reader Compatibility

```typescript
// Add aria-labels for announcements
aria-label={`${panel.title} panel. Press Delete to remove.`}
aria-live="polite"
aria-announcement={`${panel.title} panel removed. Focus moved to next panel.`}
```

### Keyboard Shortcuts

- **Tab**: Move focus to next interactive element
- **Shift+Tab**: Move focus to previous interactive element
- **Enter**: Activate focused element
- **Alt+D**: Delete focused panel (or menu item)
- **Escape**: Close menu / cancel deletion

---

## 11. Related Issues & PRs

Look for related work:

- Search for "focus" or "keyboard" in dashboard issues
- Check for existing focus management patterns elsewhere in Grafana
- Review panel menu implementation for similar focus issues
- Check accessibility roadmap for other keyboard navigation work

---

## 12. Learning Resources

### Code Review Checklist

Before submitting PR:

- [ ] Tested with mouse and keyboard
- [ ] Tested with screen reader (NVDA, JAWS, or VoiceOver)
- [ ] All edge cases covered (last panel, single panel, etc.)
- [ ] No race conditions (use `requestAnimationFrame`)
- [ ] Focus restoration works in all browsers
- [ ] No console errors or warnings
- [ ] PR includes tests (unit + E2E)
- [ ] Accessibility audit passes (Axe)
- [ ] Code follows Grafana style guide
- [ ] TypeScript types correct

### Performance Considerations

- Use `requestAnimationFrame` instead of `setTimeout` for better performance
- Debounce multiple rapid deletions
- Avoid querying DOM multiple times
- Consider if focus management could affect grid performance

### Browser Compatibility

- Ensure `requestAnimationFrame` is supported (it is, widely)
- Test focus restoration in:
  - Chrome/Edge (Chromium)
  - Firefox
  - Safari
  - Mobile browsers (iOS Safari, Chrome Android)

---

## 13. Getting Started

### To Work On This Issue

1. **Clone and setup**: Follow [Grafana development setup](https://github.com/grafana/grafana/blob/main/contribute/developer-guide.md)
2. **Create a branch**: `git checkout -b fix/keyboard-focus-panel-delete`
3. **Implement Approach 1**: Modify `utils/panel.ts`
4. **Add data attributes**: Ensure `data-panel-id` on panels
5. **Write tests**: Add unit and E2E tests
6. **Manual testing**: Test all scenarios from checklist
7. **Accessibility audit**: Run Axe and pa11y
8. **Submit PR**: Include tests and accessibility testing notes

### Local Testing

```bash
# Start Grafana in dev mode
make run

# Run tests
make test-go
yarn test

# Run specific test file
yarn test utils/panel.test.ts

# Run Cypress tests
yarn cypress run

# Accessibility audit
yarn run pa11y http://localhost:3000
```

---

## 14. Notes for Contributors

- This is a **small, achievable issue** - perfect for learning Grafana codebase
- Focus management is a **high-impact accessibility improvement**
- The solution is **localized** to one utility function
- **Good first issue** - well-scoped and documented
- Consider **pair programming** or asking for review early

---

## 15. Summary

| Aspect            | Details                                                   |
| ----------------- | --------------------------------------------------------- |
| **Problem**       | Focus resets when deleting a dashboard panel via keyboard |
| **Root Cause**    | No focus management in `removePanel()` function           |
| **Impact**        | Accessibility issue (WCAG 2.1.1), affects keyboard users  |
| **Effort**        | Small (estimated 2-4 hours)                               |
| **Solution**      | Add focus restoration to `utils/panel.ts`                 |
| **Files Changed** | 1 core file + tests                                       |
| **Tests**         | Unit + E2E + accessibility                                |
| **Value**         | High - improves dashboard usability for all users         |
