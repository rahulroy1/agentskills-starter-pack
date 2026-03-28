---
name: ui-engineering
description: >
  Review frontend architecture, component design, state management, accessibility,
  performance, and styling. Covers React, component libraries (MUI, Carbon), editor
  integrations (CodeMirror, Monaco), layout systems, and real-time UI patterns.
  Activate during UI implementation, code review, or architectural decisions
  involving frontend code.
---

# UI Engineering

## Related Skills
- **code-quality:** General code structure, naming, and refactoring.
- **performance-analysis:** Profiling and optimization strategies.
- **security-baseline:** Input validation, XSS prevention, CSP headers.
- **api-contracts:** API shape consumed by the frontend.
- **testing-strategy:** Component and integration testing.

## Gotchas

- `useEffect` for responding to user actions (click → fetch) is wrong — that's an event handler. `useEffect` is for synchronization (prop changed → update external system).
- `100vh` on mobile includes the browser chrome, causing content to overflow. Use `100dvh` (dynamic viewport height).
- `React.memo` on a component that receives an inline object/function prop as `style={{ color: 'red' }}` is useless — the object is recreated every render. Hoist or memoize the prop.
- Keys in loops must be stable IDs, not array indices — using indices causes re-mount bugs when the list is reordered.
- A WebSocket opened per component instance multiplies connections. Share one connection at the app level and distribute events.

## Checklist

### Component Architecture
- [ ] Components follow single responsibility — one reason to change
- [ ] Smart (container) vs presentational (pure) separation maintained
- [ ] Prop interfaces are minimal — pass only what the component needs
- [ ] No prop drilling beyond 2 levels — use context or state management
- [ ] Reusable components are stateless — state owned by caller
- [ ] Side effects isolated to hooks or effect boundaries, not render logic
- [ ] Components under 200 lines — extract when larger
- [ ] No business logic in JSX — extract to hooks or utilities

### State Management
- [ ] State lives at the lowest common ancestor — not hoisted unnecessarily
- [ ] Server state (API data) separated from client state (UI state)
- [ ] Form state managed locally unless shared across routes
- [ ] Global state is minimal — only truly app-wide concerns (auth, theme, notifications)
- [ ] Derived state computed, not stored — no redundant state that can go stale
- [ ] State updates are immutable — no direct mutation of state objects or arrays
- [ ] No state synchronization between stores — single source of truth per concern
- [ ] Loading, error, and empty states handled for every async operation

### Routing & Navigation
- [ ] Routes are declarative and match URL structure to UI hierarchy
- [ ] Route guards (auth, role) applied at route level, not inside components
- [ ] Navigation state not duplicated in component state
- [ ] Deep links work — any route is directly bookmarkable
- [ ] Unsaved changes guarded — warn before navigating away from dirty forms
- [ ] Lazy loading applied to route-level chunks for large features

### Styling & Theming
- [ ] One styling approach per project — no mixing CSS modules, inline styles, and CSS-in-JS
- [ ] Theme tokens used for colors, spacing, typography — no hardcoded values
- [ ] Dark/light mode handled via theme provider, not conditional classes
- [ ] Responsive breakpoints use the design system's tokens
- [ ] No `!important` — fix specificity at the source
- [ ] Z-index values managed through a scale (e.g., `zIndex.modal`, `zIndex.tooltip`)
- [ ] Component styles scoped — no global CSS leaking between features

### Layout
- [ ] Flexbox for 1D layouts, CSS Grid for 2D layouts
- [ ] Resizable panels use a splitter library or CSS `resize` — not manual mouse tracking
- [ ] Viewport-filling layouts use `height: 100dvh` (not `100vh` on mobile)
- [ ] Overflow handled explicitly — `auto` or `hidden`, never unintentional scroll
- [ ] No fixed pixel widths for containers — use `max-width`, `min-width`, or `fr` units

### Data Fetching
- [ ] Fetch on mount, not on render — avoid waterfalls
- [ ] Loading indicators shown within 200ms — no blank screens
- [ ] Errors shown inline where the data would appear, not as global alerts
- [ ] Stale data handled — show stale with refresh indicator, or refetch on focus
- [ ] Request deduplication — same data requested by multiple components fetched once
- [ ] Abort controllers used for in-flight requests on unmount or navigation
- [ ] Polling has a clear interval, pauses when tab is hidden, and cleans up

### Real-Time & Streaming
- [ ] WebSocket connection managed at app level, not per component
- [ ] Reconnection logic with exponential backoff
- [ ] Connection state (connected/disconnected/reconnecting) shown to user
- [ ] Streaming data appended incrementally — no full re-render on each message
- [ ] Backpressure handled — buffer or throttle high-frequency updates
- [ ] Cleanup on unmount — no orphaned listeners or connections

### Editor Integration (CodeMirror / Monaco)
- [ ] Editor instance created once, updated via transactions/effects — not re-mounted
- [ ] Extensions composed declaratively — no imperative plugin management in components
- [ ] Custom language modes defined as standalone modules
- [ ] Large documents virtualized — editor handles 10K+ lines without lag
- [ ] Decorations (diffs, markers, highlights) applied via the extension API, not DOM hacks
- [ ] Editor state changes propagated via `onUpdate` — not by reading DOM
- [ ] Read-only mode enforced via editor config, not by swallowing key events

### Accessibility
- [ ] Interactive elements are focusable and keyboard-operable
- [ ] ARIA roles and labels on custom components (modals, dropdowns, tabs)
- [ ] Focus management on modals — trap focus inside, restore on close
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 large text / UI)
- [ ] No information conveyed by color alone — use icons, text, or patterns
- [ ] Skip links for complex layouts (IDE panels, data grids)
- [ ] Screen reader announcements for dynamic content (toast, status changes)

## Patterns

### Component Design

#### Container + Hook pattern
```jsx
// Hook owns the logic
function usePartnerList() {
  const [partners, setPartners] = useState([]);
  const [loading, setLoading] = useState(true);
  useEffect(() => { /* fetch */ }, []);
  return { partners, loading };
}

// Component owns the UI
function PartnerList() {
  const { partners, loading } = usePartnerList();
  if (loading) return <Skeleton />;
  return <DataGrid rows={partners} />;
}
```

#### Compound components for complex UI
```jsx
// Instead of a mega-component with 20 props:
<Panel>
  <Panel.Header title="Explorer" />
  <Panel.Content>
    <FileTree />
  </Panel.Content>
  <Panel.Footer>
    <StatusBar />
  </Panel.Footer>
</Panel>
```

#### Render props for flexible composition
```jsx
// When children need parent data without prop drilling
<DataFetcher url="/api/maps">
  {({ data, loading }) => loading ? <Spinner /> : <MapList maps={data} />}
</DataFetcher>
```

### State Management

#### Context for low-frequency global state
```jsx
// Good: auth, theme, locale — changes rarely
const AuthContext = createContext();

// Bad: form field values, cursor position — changes on every keystroke
// Use local state or Zustand for high-frequency updates
```

#### Zustand for complex client state
```javascript
// Isolated store — no provider tree needed
const useSessionStore = create((set, get) => ({
  mapBuffer: null,
  edits: [],
  applyEdit: (edit) => set((s) => ({ edits: [...s.edits, edit] })),
  undo: () => set((s) => ({ edits: s.edits.slice(0, -1) })),
}));
```

#### Derived state — compute, don't store
```javascript
// Bad: storing filtered list separately (goes stale)
const [items, setItems] = useState([]);
const [filteredItems, setFilteredItems] = useState([]); // stale risk

// Good: compute on render
const filteredItems = useMemo(
  () => items.filter(i => i.status === filter),
  [items, filter]
);
```

### Layout

#### IDE-style resizable panels
```jsx
// Use a splitter library (allotment, react-resizable-panels)
<PanelGroup direction="horizontal">
  <Panel defaultSize={20} minSize={15}>
    <ExplorerPanel />
  </Panel>
  <PanelResizeHandle />
  <Panel defaultSize={55}>
    <EditorPanel />
  </Panel>
  <PanelResizeHandle />
  <Panel defaultSize={25} minSize={20}>
    <ChatPanel />
  </Panel>
</PanelGroup>
```

#### Full viewport takeover (hide parent chrome)
```jsx
// Route-level layout switch — not conditional rendering inside a layout
function App() {
  const location = useLocation();
  const isIDERoute = location.pathname.startsWith('/cr/session/');

  return isIDERoute
    ? <Outlet />                           // IDE provides its own chrome
    : <><AppHeader /><Outlet /></>;        // Standard dashboard layout
}
```

### Real-Time Streaming

#### WebSocket with auto-reconnect
```javascript
function useAgentStream(sessionId) {
  const [events, setEvents] = useState([]);
  const wsRef = useRef(null);

  useEffect(() => {
    const connect = () => {
      const ws = new WebSocket(`/ws/sessions/${sessionId}`);
      ws.onmessage = (e) => {
        const event = JSON.parse(e.data);
        setEvents(prev => [...prev, event]);  // append, don't replace
      };
      ws.onclose = () => setTimeout(connect, 1000 * Math.min(attempt++, 30));
      wsRef.current = ws;
    };
    let attempt = 0;
    connect();
    return () => wsRef.current?.close();
  }, [sessionId]);

  return events;
}
```

#### Streaming text into editor (token-by-token)
```javascript
// Batch DOM updates — don't dispatch per token
const tokenBuffer = useRef('');
const flushInterval = useRef(null);

ws.onmessage = (e) => {
  const event = JSON.parse(e.data);
  if (event.type === 'text') {
    tokenBuffer.current += event.content;
  }
};

// Flush to editor every 50ms — smooth, not janky
flushInterval.current = setInterval(() => {
  if (tokenBuffer.current) {
    editor.dispatch({ changes: { from: cursor, insert: tokenBuffer.current } });
    tokenBuffer.current = '';
  }
}, 50);
```

### Code Splitting

#### Lazy load heavy features
```javascript
// CodeMirror + IDE components only loaded when entering CR flow
const CRSessionPage = React.lazy(() =>
  import(/* webpackChunkName: "cr-ide" */ '../pages/change-request/cr-session')
);

// Wrap in Suspense at the route level
<Route path="/cr/session/:id" element={
  <Suspense fallback={<FullScreenLoader />}>
    <CRSessionPage />
  </Suspense>
} />
```

### Error Boundaries

#### Per-panel error isolation
```jsx
// IDE panels fail independently — one panel crashing doesn't take down the IDE
<PanelGroup>
  <Panel>
    <ErrorBoundary fallback={<PanelError name="Explorer" />}>
      <ExplorerPanel />
    </ErrorBoundary>
  </Panel>
  <Panel>
    <ErrorBoundary fallback={<PanelError name="Editor" />}>
      <EditorPanel />
    </ErrorBoundary>
  </Panel>
  <Panel>
    <ErrorBoundary fallback={<PanelError name="Chat" />}>
      <ChatPanel />
    </ErrorBoundary>
  </Panel>
</PanelGroup>
```

## Anti-Patterns

### Component Design
- **God components** — 500+ line components that render, fetch, validate, and route. Split by concern.
- **Prop drilling marathons** — passing props through 4+ levels. Use context or composition.
- **useEffect as event handler** — effects are for synchronization, not for responding to user actions. Use event handlers.
- **State in URL and state in component** — pick one source of truth per piece of data.
- **Rendering inside loops without keys** — causes re-mount bugs and performance issues. Keys must be stable IDs, not array indices (unless list is static).

### State Management
- **Storing API responses in multiple places** — fetch result in context AND local state AND URL params. Single source of truth.
- **Syncing two stores** — if store A updates, then useEffect syncs to store B, you have two sources of truth. Restructure so one derives from the other.
- **Giant monolithic context** — one context with 30 values causes every consumer to re-render on any change. Split by concern or use Zustand selectors.
- **Optimistic updates without rollback** — show success before confirmation, then silently fail. Always revert on error.

### Performance
- **Re-rendering the world** — parent state change re-renders 100 children. Use `React.memo`, selectors, or move state down.
- **Creating objects/arrays in render** — `style={{ color: 'red' }}` creates a new object every render. Hoist or memoize.
- **Fetching in a loop** — N+1 API calls in a `useEffect` iterating over a list. Batch or use a single endpoint.
- **Unbounded lists without virtualization** — rendering 10,000 rows in the DOM. Use `react-window`, `react-virtuoso`, or `tanstack-virtual`.
- **No abort on unmount** — API call completes after navigation, tries to setState on unmounted component.

### Styling
- **Mixing styling paradigms** — CSS modules in one component, inline styles in another, `sx` prop in a third. Pick one per project.
- **Hardcoded colors and spacing** — `color: '#0a34a1'` instead of `theme.palette.primary.main`. Breaks theming.
- **Global CSS without scoping** — `.container { padding: 16px }` bleeds into every component that uses that class name.
- **Z-index arms race** — `z-index: 99999` to fix a stacking issue. Manage a z-index scale.

### Editor Integration
- **Re-mounting editor on state change** — destroys and recreates the editor instance. Use transactions to update content.
- **Reading editor state from DOM** — parsing innerHTML instead of using the editor API. Fragile and slow.
- **Synchronous large document operations** — parsing 50K lines on the main thread. Use workers or incremental parsing.

### Real-Time
- **WebSocket per component** — 5 components each open their own connection. Share one connection at app level.
- **No reconnection logic** — connection drops, UI goes silent. Always implement reconnect with backoff.
- **Full state replacement on each message** — `setState(newData)` instead of `setState(prev => [...prev, delta])`. Causes flicker and wastes renders.
- **No cleanup on unmount** — WebSocket stays open, timers keep firing, event listeners accumulate. Always return cleanup from `useEffect`.
