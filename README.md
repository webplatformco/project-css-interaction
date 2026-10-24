# Accessible CSS Widgets

<!-- [css-ui] Proposal: `aria-*` and `role` properties, with tree-scoped element references -->

## Problem statement / user needs

An increasing number of common interaction patterns can be implemented via HTML and CSS alone, but making them accessible requires manual HTML boilerplate (tedious and error-prone), or flimsy JS to keep UI state in sync with ARIA attributes.

Additionally, for CSS pseudo-elements such as `::picker()` or `::details-content`, there is _no_ way to convey their accessibility semantics to the accessibility tree.

As a result, accessibility is often skipped, buggy, pushed to "later", or deemed out of scope altogether.

Authors are often discouraged from simpler, faster to load CSS patterns on the basis of accessibility concerns, and steered into heavy markup and JS solutions as the solution.
A CSS framework that [shelved its tabs component](https://github.com/cirthcss/cirth/issues/61) for exactly this reason put it plainly:

> CSS can select and present a state, but it cannot manage focus or update the corresponding ARIA attributes. Therefore, correct ARIA tabs currently require script.

But what if instead of discouraging authors from doing what feels natural (an uphill battle), we could pave the cowpaths, and _make_ the CSS patterns accessible by design?

The benefits are quite significant, for both developers and end-users:
- Lower friction to make experiences accessible directly translates to **more accessible experiences**.
- **Co-location resists drift.** The dominant scripted-ARIA failure is state desync; deriving semantics from the same selector that drives presentation makes them a single expression.
- The resulting decrease in complexity has the potential to **improve speed of Web UI as a whole**. Besides Built-in elements are often wrapped in components _precisely_ to encapsulate accessibility logic (which requires JS).
The ability to handle accessibility semantics in CSS allows many of these wrappers to become a simple CSS class on a built-in element, with the downstream effects compounding.

### Why CSS?

CSS is the only web platform language that facilitates _reactive aggregation_:
authors specify invariants — what applies to (pseudo-)elements that fit what criteria — and the UA takes care of the hard bits: invalidation and state sync.
There are no initialization steps, listeners, or side effect cleanup.
This matters, as invalidation is _hard_ and increases with the number of conditions.
Intent as simple as "apply X to buttons, except when hovered or focused" takes two lines of CSS, but ~50 lines of (much buggier) JS.
Add media queries into the mix, and the complexity of keeping everything in sync grows even further.

While specifying logic via invariants is not a good fit for everything (some things are inherently one-shot), accessible semantics are _exactly_ this kind of invariant, and the few that are one-shot are already JS APIs (e.g. `ariaNotify()`). The [use cases](./use-cases.md) demonstrate that very clearly.

Additionally:
- **Pseudo-elements have no HTML counterpart**, and often nesting produces incorrect accessibility semantics.
- CSS is often applied to markup that **cannot be modified directly**, such as third-party Web UI, components, CMS-generated content, content rendered from Markdown, etc. This means ARIA via CSS could even be used to *fix* broken accessibility semantics in otherwise uneditable content.
- **Consumer-side repair for cross-root ARIA:** Component authors already have paths across the boundary (form-associated custom elements plus element reflection, or Reference Target), but all of them require the component's cooperation. Where there is a selector to both ends of the relation (`::part()`, `::slotted()`), ARIA via CSS lets the consumer fix a component that did not.

## Proposal

### Summary

- CSS properties mirroring ARIA attributes that feed **only the accessibility tree computation** — never reflected to the DOM.
- Property names the same as the attribute (i.e. `role`, not `aria-role`), but value space designed to fit CSS syntax and semantics.
- Relationship properties also take tree-scoped `<dashed-ident>` references (the `anchor-name` model) instead of IDREFs (also mitigating the [IDREF DX problem](https://github.com/whatwg/html/issues/10143) without attempting to solve it completely).

Later:
- CQ/`if()` condition to match on computed `role` (since `:role()` cannot match these)

### Proposed properties

All initial `auto`, not inherited, not animatable, excluded from `all`, elements-only at Level 1.

- `role: auto | none | <ident>+`
- Token states/properties, each mirroring its attribute's grammar as `auto | none | <tokens>`: `aria-expanded`, `aria-pressed`, `aria-checked`, `aria-selected`, `aria-current`, `aria-hidden`, `aria-disabled`, `aria-invalid`, `aria-busy`, `aria-sort`, `aria-orientation`, `aria-haspopup`, `aria-modal`, `aria-readonly`, `aria-required`, `aria-multiselectable`, `aria-live`, `aria-atomic`.
- Numeric/string: `aria-level`, `aria-posinset`, `aria-setsize`, `aria-value{min,max,now,text}`, `aria-label`, `aria-description`, `aria-roledescription`, `aria-keyshortcuts`, `aria-placeholder`.
- Relationships: `aria-labelledby`, `aria-describedby`, `aria-details`, `aria-controls`, `aria-errormessage`, `aria-flowto` take `<dashed-ident>+`; `aria-activedescendant` takes one. (`aria-owns` deferred: interacts with `reading-flow` focus scopes.)

### Value semantics

- Like SVG geometry properties, the CSS properties would _override_ the corresponding ARIA attribute values for the purpose of accessibility tree computation.
- The initial value is `auto`, which defers to the corresponding ARIA attribute or implicit semantics if the attribute is not present.
- `none` makes the state/property absent even if the attribute is present.
- Anything else replaces the attribute value as *input* to the existing computation: ARIA/host-language conflict resolution, unknown-token fallback, and ARIA-in-HTML constraints all apply at used-value time. CSS changes where values come from, not which combinations are conformant.


### Element references

IDREF attributes (e.g. `aria-labelledby`) take two types of references:
- `<string>` which corresponds to an ID ref, just like an attribute would
- `<dashed-ident>` which are tree-scoped references modeled on `anchor-name`/`anchor-scope`. Strawman:

```
ref-name:  auto | none | <dashed-ident>#
ref-scope: auto | none | all | <dashed-ident>#
```

Multi-valued properties resolve each ident to all in-scope elements exposing it, in tree order; singular ones to the last in tree order (anchor precedent).

```css
.card {
	ref-scope: --title;
	role: group;
	aria-labelledby: --title;
}

.card > h3 {
	ref-name: --title;
}
```

Every instance labels itself by its own heading; no IDs, no uniqueness discipline.

Note that, as a side effect, this also [alleviates the IDREF problem](https://github.com/whatwg/html/issues/10143) without touching the much harder issue of solving the HTML API side.

## Processing model

- **Write-only, downstream.** Inputs to the a11y tree computation only. No DOM reflection: attributes are unchanged, `el.role`/`el.ariaExpanded` keep reflecting attributes, MutationObservers see nothing; `Element.computedRole` and AT see the result.
- **No circularity by construction.** Selectors match source semantics only; nothing matches this computation's output. Requires retracting the unimplemented `:role()` resolution ([#3596](https://github.com/w3c/csswg-drafts/issues/3596) — minuted as the price of the opposite choice: "we can't do both"). The identical trade was made in [#13040](https://github.com/w3c/csswg-drafts/issues/13040) re `:focusable` ([#7269](https://github.com/w3c/csswg-drafts/issues/7269)).
- Updates propagate like attribute mutation; AT already handles dynamic semantics.


## Guardrails

- **Widget roles require operability.** `button`, `tab`, `checkbox`, etc. don't take effect on elements that aren't focusable by some mechanism (native, `tabindex`, `interactivity: focusable`) — no announcing controls nobody can operate. Structural/landmark roles are exempt.
- Existing ARIA validity machinery applies unchanged, at used-value time.
- Implementations should surface rule provenance for CSS-originated semantics in a11y inspectors.


## Prior art

### Attributes from CSS

Proposals:
- [Tab Atkins' Cascading Attribute Sheets](https://gist.github.com/tabatkins/4074abaf486af5b3f7ac289737e216e4) (2012, rough sketch updated 2025)
- [Brian Kardell's polyfill](https://github.com/bkardell/cas) (2013)
- [Dave Rupert: A Use Case for CAS](https://daverupert.com/2022/10/use-case-for-cascading-attribute-sheets/) (2022)
- Daniel Glazman's earlier proposal (?)

Platform precedent for CSS properties mirroring markup attributes:
- SVG geometry attributes
- HTML presentational attributes (now largely defined in terms of CSS)
- CSS `direction` vs HTML `dir`

### CSS and the a11y tree

CSS already affects the a11y tree, often destructively and by accident:
- `display: contents` ([#3040](https://github.com/w3c/csswg-drafts/issues/3040))
- `list-style: none` in WebKit (strips `list` / `listitem` roles from unstyled `<ul>`/`<ol>`, [WebKit bug 170179](https://bugs.webkit.org/show_bug.cgi?id=170179), [write-up](https://www.scottohara.me/blog/2019/01/12/lists-and-safari.html)).

Where deliberate:
- `reading-flow`/`reading-order` (exposure order, with ARIA WG review in [aria#2241](https://github.com/w3c/aria/issues/2241)),
- `content: url() / "alt"`,
- `::scroll-marker`/`::scroll-button()` (CSS-generated interactive UI with UA-assigned semantics),
- `interactivity` (shipped, though over sustained a11y objections in [#10711](https://github.com/w3c/csswg-drafts/issues/10711) — cited as fact, not endorsement), and `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040)). The objection on [#13040](https://github.com/w3c/csswg-drafts/issues/13040) — that focusability alone amplifies the "focusable = accessible" author mistake — is an argument for completing the contract; this proposal is that completion, and the two should be evaluated as one program.

## Objections raised against `interactivity`

Collected from [#10711](https://github.com/w3c/csswg-drafts/issues/10711) (`interactivity: inert`, shipped in Chrome 135), [#13040](https://github.com/w3c/csswg-drafts/issues/13040) (`interactivity: focusable`), its 2018 precursor [#3379](https://github.com/w3c/csswg-drafts/issues/3379), the [APA WG review](https://github.com/w3ctag/design-reviews/issues/1055#issuecomment-2725956455), and Alice Boxhall's [threat model post](https://alice.boxhall.au/articles/a-threat-model-for-accessibility-on-the-web/#css-carousels-and-interactivity). Expect every one of these to be raised against this proposal.

### Against `interactivity: inert` ([#10711](https://github.com/w3c/csswg-drafts/issues/10711))

- **No visual impact, so misuse is invisible** (Boxhall, O'Hara, Suzanne, APA). Authors assume content stays as perceivable to everyone as it is to them; rules setting the property drift out of sync with the rules that change appearance; copy-pasted CSS leaves subtrees inert unnoticed. `user-select: none` / `pointer-events: none` show how such properties get abused. Non-interactive ≠ non-critical: static content made inert vanishes for AT users only, with no recourse.
- **"The `aria-hidden` of CSS"** (O'Hara). Browsers had to whittle `aria-hidden` down (ignored on `html`/`body`, undone when focus enters) because of pervasive misuse. No mitigation for author error was designed here, and once shipped it cannot be scoped down without "breaking the web".
- **Wrong layer** (Tim Nguyen, Teh, Boxhall). Inertness is content/semantics, not presentation, and CSS's association with presentation makes semantics-blind use *more* likely. Boxhall explicitly rejects the "CSS already affects the a11y tree" argument: `display: none` affects it *because* it changes presentation; the a11y tree is meant to mirror the visible experience. A property that changes semantics without changing visuals is a different category.
- **No guardrails** (Teh, Boxhall). HTML `inert` got normative guidance on inappropriate use after the same concerns; a note in a CSS spec (or any article) will be ignored. Authors learn by trial and error, and their trials don't include AT or keyboard testing. Consider careless authors and AI-generated code, not just well-intentioned ones.
- **Scope untethered from the motivating case** (Suzanne, O'Hara, Boxhall, APA). Titled "offscreen content" but applies anywhere; explained only as part of carousels with no standalone explainer; authors already read it as "remove any element from the tab order", where `tabindex=-1` or `disabled` is the right tool. Tailored solutions beat a one-size-fits-all primitive. APA: which use cases need CSS inert over the attribute at all?
- **Un-inerting** (Boxhall, APA). Escaping inertness in CSS invites specificity battles, is inconsistent with HTML `inert`, and is invisible to whoever inerted the subtree. The only real use case is modal UI, which is a top-layer question. Resolved May 2025: not inherited, cannot undo inertness.
- **Steers authors away from `<dialog>` and popover**, which were designed carefully (Boxhall). A low-level invisible primitive puts responsibility *back* on authors while making the wrong thing easier than the right one (Suzanne).
- **Process** (Boxhall, Lauke, Kardell, Mayank, Suzanne). Shipped in time for Google I/O with TAG/APA review pending; pushback treated as a debate to be rebutted rather than feedback; "developers want this" / "there are use cases" offered as sufficient justification. Boxhall: "exhausting and demoralising", and she regrets `inert` itself.
- **Smaller points.** Does set size / position report correctly when some items are inert (Grenier)? Every option risks content reachable only by scrolling (Kizu). Firefox's tab bar wants hidden-but-in-tree, so inertness is the wrong tool (Emilio). Any `opacity`-based heuristic would break visually-hidden-but-AT-visible content (O'Hara).

### Against `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040), [#3379](https://github.com/w3c/csswg-drafts/issues/3379))

- **Focusable ≠ accessible** (Higley). The dominant mistake of developers without a11y understanding is toggling focusability and thinking they are done; it degrades screen reader and voice control users. "An alarmingly large foot-gun that would increase the frequency of this mistake." New problems felt by users, in service of pain felt by developers.
- **Focusability couples to semantics and behavior, not presentation** (Rivoal 2018, Higley, Yasskin). Focus alone is rarely useful; the handler that reacts to it is JS, so the switch belongs in JS to avoid desync. Rivoal: *removing* focusability from CSS is fine; *adding* it is the suspect direction, acceptable only if there are real cases with no attached behavior. Higley: CSS may be easier at authoring time but is more fragile, harder to debug and maintain.
- **Wrong venue** (Higley, Atkins). Focus DX is not CSS-specific; CSSWG has fewer a11y participants than Open UI or HTML; `focusgroup` / `focusNext` were designed better *because* a11y folks were involved early. APA: "doesn't seem like the right layer", concerned about easy footguns.
- **`inert` is not precedent** (Kardell, Stearns). The opposition to `interactivity` was never resolved; don't hold it up as done or dig the hole deeper.
- **CSS can't know the condition** (Chan). E.g. a scroller that is only sometimes scrollable also depends on content amount; the browser should make such things focusable itself (Interop proposal) rather than hand authors another footgun.
- **Granularity** (Oriol). `tabindex=-1` (click-only) is common; if sequential focusability is exposed, so should order, suggesting `tab-index: none | <integer>` instead. Pseudo-element focus is undefined (`activeElement`).
- **Focus management is already "wonky and magical"** (Kardell 2018, Boxhall +1). Don't add significant new complexity until the existing mess is dealt with.
- **Process** (Yasskin, Higley, Stearns). Solution-first issue with no explainer, use cases, or evidence of severity, same pattern as [html#10143](https://github.com/whatwg/html/issues/10143); removed from the CSSWG agenda pending use cases and alternatives (Cascading Attribute Sheets suggested by Warlow).

## Anticipated objections

- **Conditional semantics.** When the UI is conditional on MQs, semantics that can't follow are wrong, not stable; the failure exists today with reversed polarity. `auto` + markup remain for anything that shouldn't vary.
- **Non-CSS consumers / failed loads.** `auto` keeps markup semantics as the baseline; consumers that discard CSS already discard `reading-flow` and every layout-dependent aspect of meaning.
- **Cascade/third-party CSS deciding what things are.** Third-party CSS can already destroy semantics wholesale (`display: none`/`contents`, `interactivity: inert`); it just can't repair anything. Auditing moves to the computed tree — what AT users actually get, and what inspectors and axe-style runners already evaluate.
- **Semantics without behavior.** This objection only bites for widget roles; most of the property set is pure state, where no behavioral contract exists to be missing: `aria-current` from `:target`/`scroll-state(snapped)` (scrollspy, active carousel slide — CSS is the only layer that knows), `aria-expanded` mirroring `:popover-open`/`[open]`/`:checked`/`:focus-within`, `aria-orientation` when `flex-direction` flips at a breakpoint (orientation's source of truth *is* the stylesheet), `aria-hidden` on off-canvas or clamped content, `aria-busy` on skeletons. Where roles are involved, the dominant cases relabel fully working native behavior within ARIA in HTML's existing allowances: checkbox-backed toggles already toggle on Space (`role: button; aria-pressed` or `role: switch` from `:checked` — allowed on checkbox inputs since [html-aria#6](https://github.com/w3c/html-aria/issues/6), though Chrome maps native checkedness onto `switch`, whose state is still `aria-checked`, and not onto `button`'s `aria-pressed`, which is precisely the gap these properties fill), popovers already dismiss and manage focus (`role: tooltip`; `aria-haspopup` on the invoker). Radio-hack tabs have arrow-key traversal natively but `tab` is not currently an allowed role on radio inputs — a relabel this proposal makes worth permitting via a companion ARIA-in-HTML change, the same loosening path checkbox→`button`/`switch` took. These widgets work for keyboard users today and misreport to AT users — the interaction contract is complete, the description is wrong. The residue (`role: button` on a dead `div`) is what the operability guardrail refuses, and invoker commands supply declarative behavior to compose with.
- **Wrong layer: CSS is for styling.** CSS has not been only about styling for a very long time: SVG geometry properties, aural properties, `user-select`, `touch-action`, `interactivity`, `resize`, `pointer-events`, `scroll-behavior`/`overscroll-behavior`, `content`/`quotes`, `direction`/`unicode-bidi`, `reading-flow`/`reading-order`, Opera's old `nav-*`. The actual user need was never "separation of concerns"; it is local (one-off) vs global (rule-based): O(1) author effort, a single source of truth, and no side effects or invalidation to manage. That is exactly why reactive frameworks are popular, and CSS is the platform's only fully reactive language. None of that is about styling; a separate attribute language would be an artificial boundary from the author's point of view.
- **Many of these attributes only apply to certain elements.** So do the SVG geometry properties, and nothing broke. None of these inherit.
- **Nothing encourages correct use; misuse is invisible to the author** (the neglect/threat-model objection). True of ARIA attributes and a11y-touching JS today too. But under neglect, the easiest version is what ships — and today the accessible version of CSS-driven UI costs a JS subsystem while the inaccessible one costs nothing. This feature's purpose is making the accessible version the one-declaration version; the Guardrails are the correct-use encouragement whose absence was the core complaint about `interactivity`.

## Alternatives considered

- **Cascading Attribute Sheets:** heavy platform complexity whose restricted invalidation kills the reactivity that is the point; DOM mutation has side effects this design avoids. A new language and `text/cas` MIME type is a lift no individual use case can justify, so it is unlikely to ever ship, and it forks into a parallel set of APIs that can diverge. Piggybacking on CSS may be theoretically impure, but it allows **incremental expansion** (start from the pressing use cases, not every attribute; most CAS problems like security-sensitive attributes and collisions vanish once we stop importing wholesale), lets each property grow CSS-native syntax beyond its attribute grammar, and inherits all existing and future CSS machinery: CSSOM, Typed OM, inline styles, animations, transitions, `@supports`.
- **Reference Target:** solves IDREF-across-shadow only; no cascade, conditionality, or repair.
- **AOM / ElementInternals:** right for component authors; unusable by component consumers and plain markup.
- **Do nothing:** CSS may only remove or scramble semantics, and every CSS-expressible pattern keeps its JS tax.

## Open questions

- Bare attribute names (`role`) vs a prefix.
- `auto`-deference vs modeling the attributes as presentational hints into the cascade. The alternative model: the properties set attribute *defaults*, and the markup attribute overrides at inline-style precedence (unlike SVG geometry and HTML presentational attributes, which lose to author CSS for legacy reasons). Either way, selectors keep matching the markup attribute, so no cycles. A propdef column stating whether a corresponding attribute exists and whether the property defaults or overrides it could make SVG geometry and presentational attributes first-class platform concepts rather than legacy warts.
- Invalid `role` token fallback: `auto` or `none`?
- `ref-*` vs unifying with `anchor-name`/`anchor-scope`; name visibility across shadow boundaries.
- Bare `<dashed-ident>` vs a `ref()` function resolving to a new `<element>` type that all IDREF properties take; the Typed OM could then resolve it to actual element references. Relationships whose scope does not follow the tree could use `ident()` to generate refs. Either way, `id` itself should not become CSS-settable (security).
- Per-property design is deliberate (new ARIA attribute → new property, mechanically); stated to preempt "design the generic attribute mechanism first."
