# ARIA in CSS

<!-- [css-ui] Proposal: `aria-*` and `role` properties, with tree-scoped element references -->

## Summary

CSS properties mirroring ARIA attributes that feed **only the accessibility tree computation** — never reflected to the DOM.
Relationship properties take tree-scoped `<dashed-ident>` references (the `anchor-name` model) instead of IDREFs.

## Primary Use cases

### Accessible CSS-only widgets

**Pure-CSS interaction patterns are inaccessible by construction.**
CSS can express and style interaction state (`:checked`, `:popover-open`, `[open]`, `:state()`) but not expose it.
Every "CSS-only" pattern ships either broken semantics or JS whose sole job is copying state CSS already has into attributes.

```css
.disclosure input[type=checkbox] { role: button; aria-expanded: false; }
.disclosure input[type=checkbox]:checked { aria-expanded: true; }
```

(Conforming: ARIA in HTML permits `button`, `switch`, `menuitemcheckbox`, and `option` on checkbox inputs, and those constraints apply to these properties at used-value time — CSS can relabel exactly where the attribute could.)

### Responsive semantics

Breakpoints routinely change what a widget *is* (tabs → accordion, toolbar → menu). The interaction truth is already conditional on MQs; semantics can't follow without resize-observing JS, so the a11y tree lies at every unhandled breakpoint.

### Markup you can't edit

CSS is often the only injection point into third-party components, CMS output, and embeds. You can restyle a semantically broken widget but not repair it.

## Secondary use cases

These do not drive the design, but most solutions solve or improve them as a side effect.

### Reducing the IDREF problem

There is general consensus that [IDREFs are currently painful](https://github.com/whatwg/html/issues/10143), but the right HTML solution is nonobvious.
A CSS ARIA API, with anchor positioning–like semantics has the power to alleviate the problem.

### Cross-root ARIA

By making it possible to specify ARIA via CSS, we can solve a good chunk of the problem, at least for cases where there is a selector to reference both parts of the relation (`::part()`, `::slotted()`, etc).

## Prior art

CSS already affects the a11y tree, often destructively and by accident:
- `display: contents` (#3040)
- `list-style: none` in WebKit.

Where deliberate:
- `reading-flow`/`reading-order` (exposure order, with ARIA WG review in aria#2241),
- `content: url() / "alt"`,
- `::scroll-marker`/`::scroll-button()` (CSS-generated interactive UI with UA-assigned semantics),
- `interactivity` (shipped, though over sustained a11y objections in #10711 — cited as fact, not endorsement), and `interactivity: focusable` (#13040). The objection on #13040 — that focusability alone amplifies the "focusable = accessible" author mistake — is an argument for completing the contract; this proposal is that completion, and the two should be evaluated as one program.

## Proposed properties

All initial `auto`, not inherited, not animatable, excluded from `all`, elements-only at Level 1.

- `role: auto | none | <ident>+`
- Token states/properties, each mirroring its attribute's grammar as `auto | none | <tokens>`: `aria-expanded`, `aria-pressed`, `aria-checked`, `aria-selected`, `aria-current`, `aria-hidden`, `aria-disabled`, `aria-invalid`, `aria-busy`, `aria-sort`, `aria-orientation`, `aria-haspopup`, `aria-modal`, `aria-readonly`, `aria-required`, `aria-multiselectable`, `aria-live`, `aria-atomic`.
- Numeric/string: `aria-level`, `aria-posinset`, `aria-setsize`, `aria-value{min,max,now,text}`, `aria-label`, `aria-description`, `aria-roledescription`, `aria-keyshortcuts`, `aria-placeholder`. Strings accept `attr()`.
- Relationships: `aria-labelledby`, `aria-describedby`, `aria-details`, `aria-controls`, `aria-errormessage`, `aria-flowto` take `<dashed-ident>+`; `aria-activedescendant` takes one. (`aria-owns` deferred: interacts with `reading-flow` focus scopes.)

Value semantics: `auto` defers to attribute/implicit semantics — pages without these declarations behave identically to today. `none` makes the state/property absent even if the attribute is present. Anything else replaces the attribute value as *input* to the existing computation: ARIA/host-language conflict resolution, unknown-token fallback, and ARIA-in-HTML constraints all apply at used-value time. CSS changes where values come from, not which combinations are conformant.

### References

```
ref-name:  none | <dashed-ident>#
ref-scope: none | all | <dashed-ident>#
```

Generic (not ARIA-specific), modeled on `anchor-name`/`anchor-scope`. Multi-valued properties resolve each ident to all in-scope elements exposing it, in tree order; singular ones to the last in tree order (anchor precedent).

```css
.card { ref-scope: --title; role: group; aria-labelledby: --title; }
.card > h3 { ref-name: --title; }
```

Every instance labels itself by its own heading; no IDs, no uniqueness discipline.

## Processing model

- **Write-only, downstream.** Inputs to the a11y tree computation only. No DOM reflection: attributes are unchanged, `el.role`/`el.ariaExpanded` keep reflecting attributes, MutationObservers see nothing; `Element.computedRole` and AT see the result.
- **No circularity by construction.** Selectors match source semantics only; nothing matches this computation's output. Requires retracting the unimplemented `:role()` resolution (#3596 — minuted as the price of the opposite choice: "we can't do both"). The identical trade was made in #13040 re `:focusable` (#7269).
- Updates propagate like attribute mutation; AT already handles dynamic semantics.

## Guardrails

- **Widget roles require operability.** `button`, `tab`, `checkbox`, etc. don't take effect on elements that aren't focusable by some mechanism (native, `tabindex`, `interactivity: focusable`) — no announcing controls nobody can operate. Structural/landmark roles are exempt.
- Existing ARIA validity machinery applies unchanged, at used-value time.
- **Co-location resists drift.** The dominant scripted-ARIA failure is state desync; deriving semantics from the same selector that drives presentation makes them a single expression.
- Implementations should surface rule provenance for CSS-originated semantics in a11y inspectors.

## Anticipated objections

- **Conditional semantics.** When the UI is conditional on MQs, semantics that can't follow are wrong, not stable; the failure exists today with reversed polarity. `auto` + markup remain for anything that shouldn't vary.
- **Non-CSS consumers / failed loads.** `auto` keeps markup semantics as the baseline; consumers that discard CSS already discard `reading-flow` and every layout-dependent aspect of meaning.
- **Cascade/third-party CSS deciding what things are.** Third-party CSS can already destroy semantics wholesale (`display: none`/`contents`, `interactivity: inert`); it just can't repair anything. Auditing moves to the computed tree — what AT users actually get, and what inspectors and axe-style runners already evaluate.
- **Semantics without behavior.** This objection only bites for widget roles; most of the property set is pure state, where no behavioral contract exists to be missing: `aria-current` from `:target`/`scroll-state(snapped)` (scrollspy, active carousel slide — CSS is the only layer that knows), `aria-expanded` mirroring `:popover-open`/`[open]`/`:checked`/`:focus-within`, `aria-orientation` when `flex-direction` flips at a breakpoint (orientation's source of truth *is* the stylesheet), `aria-hidden` on off-canvas or clamped content, `aria-busy` on skeletons. Where roles are involved, the dominant cases relabel fully working native behavior within ARIA in HTML's existing allowances: checkbox-backed toggles already toggle on Space (`role: button; aria-pressed` or `role: switch` from `:checked` — allowed on checkbox inputs since html-aria#6, though the browser doesn't map native checkedness onto the overridden role's state, which is precisely the gap these properties fill), popovers already dismiss and manage focus (`role: tooltip`; `aria-haspopup` on the invoker). Radio-hack tabs have arrow-key traversal natively but `tab` is not currently an allowed role on radio inputs — a relabel this proposal makes worth permitting via a companion ARIA-in-HTML change, the same loosening path checkbox→`button`/`switch` took. These widgets work for keyboard users today and misreport to AT users — the interaction contract is complete, the description is wrong. The residue (`role: button` on a dead `div`) is what the operability guardrail refuses, and invoker commands supply declarative behavior to compose with.
- **Nothing encourages correct use; misuse is invisible to the author** (the neglect/threat-model objection). True of ARIA attributes and a11y-touching JS today too. But under neglect, the easiest version is what ships — and today the accessible version of CSS-driven UI costs a JS subsystem while the inaccessible one costs nothing. This feature's purpose is making the accessible version the one-declaration version; the Guardrails are the correct-use encouragement whose absence was the core complaint about `interactivity`.

## Alternatives considered

- **Cascading Attribute Sheets:** heavy platform complexity whose restricted invalidation kills the reactivity that is the point; DOM mutation has side effects this design avoids.
- **Reference Target:** solves IDREF-across-shadow only; no cascade, conditionality, or repair.
- **AOM / ElementInternals:** right for component authors; unusable by component consumers and plain markup.
- **Do nothing:** CSS may only remove or scramble semantics, and every CSS-expressible pattern keeps its JS tax.

## Open questions

- Bare attribute names (`role`) vs a prefix.
- `auto`-deference vs modeling the attributes as presentational hints into the cascade.
- Invalid `role` token fallback: `auto` or `none`?
- `ref-*` vs unifying with `anchor-name`/`anchor-scope`; name visibility across shadow boundaries.
- Level 1 cut: `aria-owns`, the `aria-value*` group.
- Per-property design is deliberate (new ARIA attribute → new property, mechanically); stated to preempt "design the generic attribute mechanism first."
