# Phase 4: Structured Accessibility - Research

**Researched:** 2026-03-10
**Domain:** Web accessibility testing via Playwright MCP tools (no external a11y libraries)
**Confidence:** HIGH for Playwright APIs; MEDIUM for specific implementation patterns

---

## Summary

Phase 4 adds Tier 2 accessibility testing to the QA agent — deeper than the baseline checks already in the agent's system prompt (keyboard nav, focus indicators, alt text, form labels). The phase covers 17 requirements across five concern areas: focus management, page structure, interactive elements, and form accessibility.

The core constraint is that all testing must use only Playwright MCP tools — no `@axe-core/playwright`, no external a11y libraries. This means every check is implemented via Playwright's keyboard simulation APIs, `page.evaluate()` for DOM introspection, `locator.boundingBox()` for size measurement, `page.emulateMedia()` for preference emulation, and `page.title()` for SPA navigation checks. The agent must teach itself to use these tools as a deliberate accessibility test methodology, not as incidental byproducts of functional testing.

The implementation pattern follows exactly the same approach as Phase 3: add a new methodology section to `agents/qa-tester.md` and add corresponding instructions to the agent prompt in `skills/qa-run.md`. No new files are needed.

**Primary recommendation:** Structure the phase as a single plan (not 4 sub-plans like Phase 3) since all 17 requirements fit naturally into one cohesive "Structured Accessibility" section in the agent, and the Playwright APIs needed are well-understood. Break into sub-plans only if the methodology section becomes unwieldy (>200 lines).

---

## Standard Stack

### Core
| Tool | Version | Purpose | Why Standard |
|------|---------|---------|--------------|
| `page.keyboard.press('Tab')` | Playwright built-in | Simulate keyboard navigation | Only way to test focus order without a screen reader |
| `page.evaluate()` | Playwright built-in | DOM introspection for heading hierarchy, lang attribute, aria attributes | Direct DOM access for things Playwright locators don't expose |
| `locator.boundingBox()` | Playwright built-in | Measure element width/height for touch target size | Returns `{x, y, width, height}` in pixels |
| `page.emulateMedia()` | Playwright built-in | Emulate `prefers-reduced-motion: reduce` | Native Playwright API, no third-party needed |
| `page.title()` | Playwright built-in | Check page title changes after SPA navigation | Auto-waits; `await expect(page).toHaveTitle(...)` |
| `locator.toBeFocused()` | Playwright built-in | Assert a specific element is currently focused | Works via `document.activeElement` comparison |
| `page.getByRole()` | Playwright built-in | Locate elements by ARIA role (landmark regions) | Semantic selectors; role-based queries |
| `expect(locator).toMatchAriaSnapshot()` | Playwright built-in | Validate ARIA tree structure (headings, landmarks) | YAML representation of accessibility tree |

### No External Libraries Needed
The out-of-scope exclusions (color contrast calculation, full WCAG audit, screen reader testing) are the things that would require `@axe-core/playwright`. Everything in scope is achievable with Playwright alone.

**Installation:** No new packages required. This phase adds only methodology to existing skill files.

---

## Architecture Patterns

### Recommended Structure
Phase 4 follows the exact same pattern as Phase 3 sub-plans:

```
agents/qa-tester.md         — Add "Structured Accessibility" methodology section
skills/qa-run.md            — Add accessibility testing instructions to agent prompt
```

No new files. No new commands. The accessibility methodology is embedded in the agent's system prompt and activated via the agent prompt in qa-run.

### Pattern 1: Focus Management via Tab Simulation

**What:** Press Tab to move focus into a modal, then Tab repeatedly to verify focus stays trapped. Press Escape (or close button) and verify focus returns to the trigger.

**When to use:** Any modal, dialog, or drawer that opens over the page.

**Example:**
```javascript
// Source: Playwright keyboard API (playwright.dev/docs/input)

// Open modal (via clicking trigger)
await page.click('[data-trigger="modal"]');

// Verify focus moved into the modal
await page.keyboard.press('Tab');
const focused = await page.evaluate(() => document.activeElement?.closest('[role="dialog"]'));
// focused should be non-null

// Verify Tab doesn't escape the modal (press Tab N times, focus stays inside)
for (let i = 0; i < 10; i++) {
  await page.keyboard.press('Tab');
  const isInsideModal = await page.evaluate(() =>
    document.activeElement?.closest('[role="dialog"]') !== null
  );
  // isInsideModal should remain true
}

// Close modal (Escape or close button)
await page.keyboard.press('Escape');

// Verify focus returned to trigger
await expect(page.locator('[data-trigger="modal"]')).toBeFocused();
```

### Pattern 2: Heading Hierarchy Audit via DOM Inspection

**What:** Use `page.evaluate()` to collect all heading elements, extract their levels and text, then analyze for: single h1, no skipped levels, logical nesting.

**When to use:** Every page audit.

**Example:**
```javascript
// Source: page.evaluate + querySelectorAll pattern
const headings = await page.evaluate(() => {
  return Array.from(document.querySelectorAll('h1, h2, h3, h4, h5, h6')).map(h => ({
    level: parseInt(h.tagName[1]),
    text: h.textContent?.trim().substring(0, 50)
  }));
});

// Check: single h1
const h1s = headings.filter(h => h.level === 1);
// h1s.length should equal 1

// Check: no level skipping (e.g., h1 → h3 skips h2)
for (let i = 1; i < headings.length; i++) {
  const levelDiff = headings[i].level - headings[i-1].level;
  // levelDiff should not be > 1
}
```

### Pattern 3: Landmark Regions via getByRole

**What:** Use `page.getByRole()` to verify presence of required landmark regions.

**When to use:** Every page audit.

**Example:**
```javascript
// Source: Playwright locators (playwright.dev/docs/locators)

// Required landmarks per WCAG
await expect(page.getByRole('main')).toBeVisible();
await expect(page.getByRole('banner')).toBeVisible();        // <header>
await expect(page.getByRole('contentinfo')).toBeVisible();   // <footer>
await expect(page.getByRole('navigation')).toBeVisible();    // at least one <nav>
```

### Pattern 4: Touch Target Size Measurement

**What:** Get bounding boxes for all interactive elements, flag any below 44x44px.

**When to use:** Mobile viewport (375px) specifically. Already integrated into multi-viewport checks from Phase 3.

**Example:**
```javascript
// Source: locator.boundingBox() (playwright.dev/docs/api/class-locator)

const interactiveSelectors = 'a, button, [role="button"], input, select, textarea, [tabindex]';
const elements = await page.locator(interactiveSelectors).all();

const undersizedTargets = [];
for (const el of elements) {
  const box = await el.boundingBox();
  if (box && (box.width < 44 || box.height < 44)) {
    const label = await el.evaluate(e =>
      e.getAttribute('aria-label') || e.textContent?.trim().substring(0, 30) || e.tagName
    );
    undersizedTargets.push({ label, width: box.width, height: box.height });
  }
}
// Report all undersizedTargets
```

### Pattern 5: Reduced Motion Emulation

**What:** Set `prefers-reduced-motion: reduce` via `emulateMedia`, then check whether animations/transitions are suppressed. Check by inspecting CSS computed styles for `animation-duration` and `transition-duration`.

**When to use:** Pages with animations, carousels, loading spinners.

**Example:**
```javascript
// Source: page.emulateMedia (playwright.dev/docs/api/class-page)

await page.emulateMedia({ reducedMotion: 'reduce' });
// Then check whether animated elements respect the preference
const animationDuration = await page.evaluate(() => {
  const el = document.querySelector('.animated-element');
  if (!el) return null;
  return window.getComputedStyle(el).animationDuration;
});
// animationDuration should be '0s' or very short when reduced motion is preferred
```

### Pattern 6: Zoom Simulation (200%)

**What:** Playwright does not have a native browser zoom API. The practical approach for automated testing is to halve the viewport width (simulating what the user sees at 200% text zoom) OR use `page.evaluate()` to set `document.documentElement.style.fontSize = '200%'` which simulates text-only zoom. Then check for horizontal overflow.

**When to use:** After rendering at "zoomed" state.

**Limitation:** This is NOT identical to real browser zoom. Real zoom increases both text and UI elements. The CSS font-size approach only tests text zoom (WCAG 1.4.4). The viewport-halving approach tests responsive layout. Both are reasonable proxies for automated testing.

**Example:**
```javascript
// Source: page.evaluate CSS approach / viewport resize approach

// Option A: Text-only zoom simulation (WCAG 1.4.4 Resize Text)
await page.evaluate(() => {
  document.documentElement.style.fontSize = '200%';
});

// Check for horizontal overflow
const hasHorizontalOverflow = await page.evaluate(() => {
  return document.documentElement.scrollWidth > document.documentElement.clientWidth;
});
// hasHorizontalOverflow should be false

// Option B: Narrow viewport (simulates responsive behavior at high zoom)
await page.setViewportSize({ width: 640, height: 800 }); // 1280 / 2
// Then check that content is still accessible
```

### Pattern 7: Skip Link Verification

**What:** Skip links are typically the first focusable element on a page. Press Tab once to focus them, then activate (Enter/click) to verify focus jumps to the target anchor.

**When to use:** Every page audit.

**Example:**
```javascript
// Source: Playwright keyboard + focus pattern

// Tab to first focusable element (should be skip link)
await page.keyboard.press('Tab');

// Check if focused element is a skip link
const isSkipLink = await page.evaluate(() => {
  const el = document.activeElement;
  return el?.tagName === 'A' &&
    (el?.textContent?.toLowerCase().includes('skip') ||
     el?.getAttribute('href')?.startsWith('#'));
});

if (isSkipLink) {
  await page.keyboard.press('Enter');
  // Verify focus moved to target
  const targetId = await page.evaluate(() => {
    const el = document.querySelector('a[href^="#"]');
    return el?.getAttribute('href')?.substring(1);
  });
  if (targetId) {
    await expect(page.locator(`#${targetId}`)).toBeFocused();
  }
}
```

### Pattern 8: aria-live Region Verification

**What:** Playwright cannot directly observe screen reader announcements. The practical test is to verify: (1) aria-live regions are present in the DOM, (2) dynamic content is injected into them, (3) the region has correct `aria-live` value (`polite` or `assertive`).

**When to use:** Status messages, form errors, toast notifications, real-time updates.

**Example:**
```javascript
// Source: page.evaluate + DOM inspection

// Check aria-live regions exist
const liveRegions = await page.evaluate(() => {
  return Array.from(document.querySelectorAll('[aria-live]')).map(el => ({
    role: el.getAttribute('role'),
    live: el.getAttribute('aria-live'),
    id: el.id,
    hasContent: el.textContent?.trim().length > 0
  }));
});

// After triggering a dynamic event (form error, status change):
// Verify the live region now has content
await page.click('#submit-button');
const statusContent = await page.evaluate(() =>
  document.querySelector('[aria-live]')?.textContent?.trim()
);
// statusContent should be non-empty after submission
```

### Pattern 9: lang Attribute Check

**What:** Simple DOM inspection of the html element's lang attribute.

**Example:**
```javascript
// Source: page.evaluate
const lang = await page.evaluate(() => document.documentElement.lang);
// lang should be non-empty (e.g., 'en', 'en-US', 'fr')
```

### Anti-Patterns to Avoid

- **Tab-spamming without checking:** Pressing Tab 20+ times to detect escape from a modal is flaky. Instead, press Tab 5-10 times and verify focus stays inside; if it does, the trap is likely working.
- **Treating animation test as binary pass/fail:** Animation behavior varies by framework. Flag as Low confidence when `animation-duration` isn't `0s` but the page visually looks static — the property may be on the wrong element.
- **Using page.evaluate() to click things:** Always prefer `locator.click()` for interactions. Use `page.evaluate()` only for reading DOM state.
- **Skipping zoom check on mobile viewport:** The 200% zoom check should be done at desktop viewport (1280px), not mobile. The responsive design at mobile already represents a zoomed-down view.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Detecting focused element | Custom JS loop | `locator.toBeFocused()` + `page.evaluate(() => document.activeElement)` | Playwright's assertion handles timing and retries |
| Checking element role | Manual attribute parsing | `page.getByRole()` | Handles both explicit role= and implicit roles from semantic HTML |
| Page title assertion | `await page.evaluate(() => document.title)` | `await expect(page).toHaveTitle(...)` | toHaveTitle auto-waits for SPA title updates |
| ARIA tree inspection | Custom DOM walker | `expect(locator).toMatchAriaSnapshot()` | Built-in ARIA tree representation with partial matching |
| Emulating user preferences | Browser launch flags | `page.emulateMedia()` | Can change preferences at runtime per test |

**Key insight:** Playwright's role-based locators (`getByRole`) compute the accessibility name and role the same way assistive technology does — they resolve both explicit ARIA attributes and implicit HTML semantics. This makes them more reliable than CSS selectors or attribute queries for accessibility testing.

---

## Common Pitfalls

### Pitfall 1: Focus Trap False Positives (A11Y-03)

**What goes wrong:** Tab trapping test passes because there happen to be enough focusable elements inside the modal that the test never hits the boundary. But the trap isn't actually enforced — if the user has more Tab presses, focus escapes.

**Why it happens:** Testing only a few Tab presses doesn't prove trapping.

**How to avoid:** Press Tab enough times to cycle through all focusable elements in the modal AT LEAST TWICE. If Tab count > (visible focusable elements in modal + 2) and focus stays inside, the trap is verified. Alternatively, check that Tab from the last focusable element in the modal moves focus to the first focusable element in the modal (cycle detection).

**Warning signs:** Modal has only 1-2 focusable elements (easy to miss the trap boundary).

### Pitfall 2: Reduced Motion Check Targets Wrong Element (A11Y-12)

**What goes wrong:** The test checks `animation-duration` on the container element, but the animation is applied to a child. Reports "reduced motion respected" incorrectly.

**Why it happens:** CSS animations can be on any element in the subtree.

**How to avoid:** Check the computed styles on the visually animated element (spinner, transition, slide), not its parent. Take a screenshot before and after emulating reduced motion to visually confirm behavior change.

**Warning signs:** Page has visible animations but `animation-duration` query returns `0s`.

### Pitfall 3: Heading Level Audits Flagging False Gaps (A11Y-06)

**What goes wrong:** An h4 appears before any h3 on the page, triggering a "skipped level" warning, but the h4 is inside a component with its own internal hierarchy (e.g., a sidebar widget).

**Why it happens:** WCAG heading hierarchy should be document-level, but components can have internal heading structures that look like gaps when viewed in isolation.

**How to avoid:** Flag heading gaps as Medium confidence, not High. Note the context of where the gap occurs (main content vs. widget/aside). Pure document-level heading issues (e.g., no h1, multiple h1s) are High confidence.

**Warning signs:** Page uses component libraries (Material UI, Shadcn, etc.) which often have their own internal heading structures.

### Pitfall 4: Touch Target False Pass at Invisible Elements (A11Y-10)

**What goes wrong:** `locator.boundingBox()` returns `null` for hidden or collapsed elements (e.g., mobile nav items that are hidden at desktop viewport). The test skips them, recording no failures. But at mobile viewports those same elements are visible and may be undersized.

**Why it happens:** `boundingBox()` returns null when the element is not visible.

**How to avoid:** Touch target size checks MUST run at mobile viewport (375x812), not desktop. Align with Phase 3's viewport orchestration — run at mobile viewport specifically. `boundingBox()` returning null at mobile means the element is actually hidden at that size, which is fine to skip.

**Warning signs:** Running touch target check at desktop and getting zero failures when the site has dense mobile navigation.

### Pitfall 5: Skip Link Target Gets Focus but Link Test Fails (A11Y-05)

**What goes wrong:** The skip link exists and the href target element exists, but the target element doesn't have `tabindex="-1"` and so it can't receive programmatic focus via `#anchor`. The test falsely reports "skip link broken."

**Why it happens:** Only natively focusable elements (links, buttons, inputs) accept focus by default. `<main id="content">` needs `tabindex="-1"` to be focusable via skip link.

**How to avoid:** When skip link activation doesn't move focus as expected, check whether the target element has `tabindex="-1"` (or is a natively focusable element). This is a Medium confidence finding — the link destination may still be correct even if focus doesn't move to it precisely.

### Pitfall 6: Zoom Test Doesn't Reflect Real Zoom Behavior (A11Y-13)

**What goes wrong:** The CSS font-size injection test passes cleanly, but real browser zoom at 200% breaks the layout. The two are not equivalent.

**Why it happens:** Real browser zoom scales ALL elements (including fixed-width containers), while CSS font-size only scales text. `rem`-based layouts respond to font-size changes; `px`-based layouts do not.

**How to avoid:** Frame the automated test as a signal/proxy, not a guarantee. The font-size injection test catches text overflow issues in `rem`-based layouts. The viewport-halving test catches responsive breakpoint issues. Combine both. Flag any failures as High confidence. If both pass, report as "zoom proxy tests passed — manual verification at 200% browser zoom recommended for final validation."

---

## Code Examples

Verified patterns from Playwright documentation:

### Get element bounding box (for touch target size)
```javascript
// Source: playwright.dev/docs/api/class-locator#locator-bounding-box
const box = await page.getByRole('button').boundingBox();
// Returns: { x: number, y: number, width: number, height: number } or null
```

### Emulate reduced motion preference
```javascript
// Source: playwright.dev/docs/api/class-page#page-emulate-media
await page.emulateMedia({ reducedMotion: 'reduce' });
// Also supported: 'no-preference' | null
// Can also set: colorScheme, forcedColors, contrast, media
```

### Assert page title (with auto-wait for SPA)
```javascript
// Source: playwright.dev/docs/api/class-pageassertions
await expect(page).toHaveTitle('Expected Page Title');
// Or regex:
await expect(page).toHaveTitle(/Dashboard/);
```

### Locate by landmark role
```javascript
// Source: playwright.dev/docs/locators
await expect(page.getByRole('main')).toBeVisible();
await expect(page.getByRole('banner')).toBeVisible();     // header
await expect(page.getByRole('contentinfo')).toBeVisible(); // footer
await expect(page.getByRole('navigation')).toBeVisible(); // nav
```

### Assert element is focused
```javascript
// Source: playwright.dev/docs/api/class-locatorassertions
await expect(page.getByRole('button', { name: 'Close' })).toBeFocused();
// Or via evaluate:
const isFocused = await element.evaluate(el => el === document.activeElement);
```

### ARIA snapshot partial matching for heading structure
```javascript
// Source: playwright.dev/docs/aria-snapshots
await expect(page.locator('main')).toMatchAriaSnapshot(`
  - heading "Page Title" [level=1]
  - heading /Section/ [level=2]
`);
// Partial: heading level 3 with any text:
await expect(page.locator('main')).toMatchAriaSnapshot(`
  - heading [level=3]
`);
```

### Check lang attribute
```javascript
// Source: page.evaluate pattern
const lang = await page.evaluate(() => document.documentElement.getAttribute('lang'));
// Should be non-empty: 'en', 'en-US', 'fr-CA', etc.
```

### Detect horizontal overflow
```javascript
// Source: page.evaluate + scrollWidth/clientWidth comparison
const hasHorizontalOverflow = await page.evaluate(() =>
  document.documentElement.scrollWidth > document.documentElement.clientWidth
);
```

### Check aria-invalid on field
```javascript
// Source: page.evaluate / locator attribute
const isInvalid = await page.getByRole('textbox', { name: 'Email' })
  .evaluate(el => el.getAttribute('aria-invalid'));
// Should be 'true' after form submission with errors
```

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|------------------|--------|
| Only testing with axe-core for a11y | Playwright-native APIs first, axe-core as opt-in enhancement | Phase 4 stays self-contained, no new dependencies |
| `page.$(selector)` for DOM queries | `page.getByRole()` + `locator.*` APIs | More readable, handles ARIA semantics automatically |
| `ElementHandle.getBoundingClientRect()` | `locator.boundingBox()` | Returns same data, but locator API is the current recommended pattern |
| Manual focus state check via `document.activeElement` | `locator.toBeFocused()` assertion | Assertion retries automatically, more reliable |

**Deprecated/outdated:**
- `page.$eval()` / `page.$$eval()`: Not deprecated but the recommended pattern is now `locator.evaluate()` for single-element operations and `page.evaluate()` for page-level DOM inspection. Both work; use `page.evaluate()` for batch DOM inspection (heading audit, etc.).

---

## Implementation Plan Shape

Based on research, Phase 4 fits cleanly as **2 sub-plans** following Phase 3's model:

**Plan 04-01: Focus Management + Page Structure**
Covers A11Y-01 through A11Y-09:
- Focus management: modal open/close/trap, aria-live, skip links (A11Y-01 to A11Y-05)
- Page structure: heading hierarchy, landmark regions, page title, lang attribute (A11Y-06 to A11Y-09)

**Plan 04-02: Interactive Elements + Form Accessibility**
Covers A11Y-10 through A11Y-17:
- Interactive elements: touch targets, color as sole indicator, reduced motion, zoom (A11Y-10 to A11Y-13)
- Form accessibility: error summary, field links, aria-invalid, focus on first error (A11Y-14 to A11Y-17)

Both plans modify only `agents/qa-tester.md` and `skills/qa-run.md`.

---

## Open Questions

1. **Color as sole indicator for state (A11Y-11)**
   - What we know: "Color is not the sole indicator" requires checking that state changes (hover, active, selected, error) are communicated by more than color alone — also text, icon, underline, border, etc.
   - What's unclear: Playwright cannot compute color values or compare visual states without screenshots. Testing this programmatically is difficult without axe-core.
   - Recommendation: The agent should visually inspect (via screenshot) interactive states and use judgment. Frame as "check that state changes have a non-color indicator (text label, icon, underline, border change)." This makes it a Medium confidence observation that relies on agent judgment, not a mechanical assertion. Keep scope narrow and note the limitation.

2. **aria-live announcement verification depth (A11Y-04)**
   - What we know: Playwright cannot intercept screen reader announcements. We can verify the structural presence of aria-live regions and that they contain content after events.
   - What's unclear: Whether "is announced" should require verifying that the content is actually spoken aloud (requires screen reader — out of scope).
   - Recommendation: Scope to structural verification: live region exists, has correct `aria-live` value, and contains content after the dynamic event. Note in methodology that actual announcement requires manual screen reader testing.

---

## Sources

### Primary (HIGH confidence)
- `playwright.dev/docs/api/class-page#page-emulate-media` — emulateMedia API with reducedMotion parameter verified
- `playwright.dev/docs/api/class-locator#locator-bounding-box` — boundingBox() return structure verified
- `playwright.dev/docs/aria-snapshots` — ARIA snapshot format and toMatchAriaSnapshot usage verified
- `playwright.dev/docs/input` — keyboard.press(), locator.press() APIs verified
- `playwright.dev/docs/accessibility-testing` — confirmed no built-in a11y audit; relies on @axe-core/playwright for full audits

### Secondary (MEDIUM confidence)
- `thisdot.co/blog/testing-accessibility-features-with-playwright` — keyboard navigation and focus testing patterns, verified with Playwright docs
- `boia.org/blog/zoom-testing-for-web-accessibility-a-quick-guide` — WCAG 1.4.4 zoom testing guidance, confirmed 200% requirement
- WebSearch: Playwright `page.title()` and `toHaveTitle()` auto-wait behavior, cross-verified with docs

### Tertiary (LOW confidence)
- WebSearch: focus trap detection approaches — general patterns confirmed but exact Tab count recommendation is judgment-based
- WebSearch: heading hierarchy audit via `querySelectorAll` — standard DOM pattern, not Playwright-specific documentation

---

## Metadata

**Confidence breakdown:**
- Playwright APIs (keyboard, boundingBox, emulateMedia, title): HIGH — verified from official docs
- Implementation patterns (focus trap loop, heading audit): MEDIUM — patterns derived from Playwright APIs + accessibility knowledge
- Zoom simulation approach: MEDIUM — known Playwright limitation with no native zoom API, proxy approach is pragmatic
- Color-as-sole-indicator: LOW — no clean automated approach; agent judgment required

**Research date:** 2026-03-10
**Valid until:** 2026-06-10 (Playwright APIs are stable; 90 days reasonable)
