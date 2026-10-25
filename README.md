# ARIA in CSS

<!-- [css-ui] Proposal: `aria-*` and `role` properties, with tree-scoped element references -->

**TL;DR:** `role` and `aria-*` as CSS properties, with tree-scoped element references.
First part of a broader program on completing CSS-driven interaction: CSS can create it today, but cannot make it accessible or operable.

- [Problem statement \& user needs](#problem-statement--user-needs)
	- [Use cases](#use-cases)
	- [Why CSS?](#why-css)
	- [Cross-root ARIA](#cross-root-aria)
	- [What about CAS (Cascading Attribute Sheets)?](#what-about-cas-cascading-attribute-sheets)
- [Proposal](#proposal)
	- [Summary](#summary)
	- [Proposed properties](#proposed-properties)
		- [Supporting existing patterns: ARIA](#supporting-existing-patterns-aria)
		- [Encouraging more patterns: focusability](#encouraging-more-patterns-focusability)
	- [Precedence of CSS properties over ARIA attributes](#precedence-of-css-properties-over-aria-attributes)
	- [Element references: `ref-scope` and `ref-name`](#element-references-ref-scope-and-ref-name)
- [Processing model](#processing-model)
- [Guardrails](#guardrails)
- [Prior art](#prior-art)
	- [Attributes from CSS](#attributes-from-css)
	- [ARIA via CSS](#aria-via-css)
	- [CSS and the a11y tree](#css-and-the-a11y-tree)
- [Anticipated objections](#anticipated-objections)
	- [Layer and venue](#layer-and-venue)
	- [Misuse](#misuse)
	- [Focus and behavior](#focus-and-behavior)
	- [Semantics](#semantics)


## Problem statement & user needs

An increasing number of common interaction patterns can be implemented via HTML and CSS alone, but making them accessible requires manual HTML boilerplate (tedious and error-prone), or flimsy JS to keep UI state in sync with ARIA attributes.

Additionally, for **CSS pseudo-elements** such as `::picker()`, `::details-content`, `::part()`, `::slotted()` etc there is _no_ way to convey their accessibility semantics to the accessibility tree.

As a result, accessibility is often skipped, buggy, pushed to "later", or deemed out of scope altogether.

Authors are often discouraged from simpler, faster to load HTML & CSS patterns on the basis of accessibility concerns, and steered into heavy markup and JS solutions as the solution.
A CSS framework that [shelved its tabs component](https://github.com/cirthcss/cirth/issues/61) for exactly this reason put it plainly:

> CSS can select and present a state, but it cannot manage focus or update the corresponding ARIA attributes. Therefore, correct ARIA tabs currently require script.

But what if instead of discouraging authors from doing what feels natural (always an uphill battle), we paved the cowpaths, and _made_ the CSS patterns _actually_ accessible?

The benefits are quite significant, for both developers and end-users:
- Lower friction to make experiences accessible directly translates to **more accessible experiences**.
- **Semantics move up a level of abstraction.** Today the only unit that can carry ARIA once for all its uses is a JS component; a class, a CSS framework or the UA stylesheet cannot. In CSS, whoever defines a pattern specifies it once, and consumers get it automatically (e.g. by using the class).
- **Co-location resists drift.** The dominant scripted-ARIA failure is state desync; deriving semantics from the same selector that drives presentation makes them a single expression.
- **Fault tolerance.** Scripted ARIA fails whole: In JS _any_ uncaught error breaks the entire file. CSS is fault-tolerant by design. An incorrect declaration is dropped without affecting the rest of the sheet.
- The resulting decrease in complexity has the potential to **improve speed of Web UI as a whole**. Besides Built-in elements are often wrapped in components _precisely_ to encapsulate accessibility logic (which requires JS).
The ability to handle accessibility semantics in CSS allows many of these wrappers to become a simple CSS class on a built-in element, with the downstream effects compounding.

### Use cases

The [full list](./use-cases.md) is drawn from current implementation guidance and falls into five buckets:

- **[Manual ARIA boilerplate](./use-cases.md#manual-aria-boilerplate)**: facts CSS already knows, added by hand on every instance: `aria-hidden="true"` on every decorative icon, `aria-current="page"` on the current nav link, a unique id on every dialog heading so `aria-labelledby` can point at it, etc.
- **[JS that exists only to sync CSS state into ARIA](./use-cases.md#js-to-sync-aria-state)**: `aria-invalid` from `:user-invalid`, `aria-current` from `:target-current`, `aria-pressed` or `aria-selected` from `:checked` (toggle buttons, tabs, `<select>` listboxes restyled as checkbox groups), etc.
- **[Pseudo-elements that cannot carry ARIA at all](./use-cases.md#pseudo-elements)**: `::picker()` laid out as a grid, `::details-content` as a labelled region, `::slotted()` / `::part()` for semantics across a shadow boundary, from either side, etc.
- **[Markup the author cannot edit](./use-cases.md#markup-you-cannot-edit)**: Markdown output, third-party widgets, components that did not forward labels, etc.
- **[Built-in browser magic, made visible](./use-cases.md#built-in-browser-magic-made-visible)**: implicit roles, `::tooltip`'s role, the `aria-hidden` corrections browsers apply silently, all as UA stylesheet rules that can be inspected.

### Why CSS?

CSS is the only web platform language built around _reactive invariants_:
authors write rules declaring what applies to which (pseudo-)elements under which conditions, and the UA combines whatever matches and takes care of the hard bits: invalidation and state sync.
There are no initialization steps, listeners, or side effect cleanup.
This matters because invalidation is _hard_, and its difficulty grows with the number of conditions.
Intent as simple as "apply X to buttons, except when hovered or focused" takes two lines of CSS, but ~50 lines of (much buggier) JS.
Add media queries to the mix, and the complexity of keeping everything in sync grows even further.

Specifying logic via invariants is not a good fit for everything; some things are inherently one-shot.
But accessible semantics are _exactly_ this kind of invariant, and the few one-shot cases are already JS APIs (e.g. `ariaNotify()`).
The [use cases](./use-cases.md) demonstrate this very clearly.

Additionally:
- **Pseudo-elements have no HTML counterpart**, and often nesting (e.g. a `<div>` wrapping the contents of `::picker()` or `::details-content`) produces incorrect accessibility semantics.
- CSS is often applied to markup that **cannot be modified directly**, such as third-party Web UI, components, CMS-generated content, content rendered from Markdown, etc. This means ARIA via CSS could even be used to *fix* broken accessibility semantics in otherwise uneditable content.

### Cross-root ARIA

ARIA relationships are IDREFs, and IDREFs do not cross shadow boundaries, in either direction.
Both sides of a component are stuck today, for different reasons.

**Inside the component.** `ElementInternals` lets a component put a role and ARIA state on its own host, and nothing else.
The light-DOM children a consumer slots in are off limits, short of writing attributes into markup the consumer owns.
It is also imperative, so every code path that changes state must remember to update it, and it does not compose: a `checked` attribute, a custom state and a consumer override each need their own plumbing into the mixin.
With ARIA properties, the component stylesheet declares all of it once, for the host and for slotted content alike:

```css
:host {
	role: tablist;
}

::slotted([slot="tab"]) {
	role: tab;
}

::slotted([slot="tab"][selected]) {
	aria-selected: true;
}
```

**Outside the component.** Every existing path across the boundary (form-associated custom elements, element reflection, [Reference Target](https://github.com/WICG/webcomponents/blob/gh-pages/proposals/reference-target-explainer.md)) requires the component's cooperation.
Where a selector reaches both ends of the relation, a consumer can fix a component that did not:

```css
label:has(> fancy-input) {
	ref-name: --label;
	ref-scope: --label;
}

label > fancy-input::part(input) {
	aria-labelledby: --label;
}
```

**Relation to Reference Target.** Reference Target lets a shadow host nominate an element inside its shadow root as the target of any IDREF pointing at the host, so `<label for>`, `aria-labelledby`, `popovertarget` and the rest work across the boundary with no change on the referencing side.
That fixes the reference primitive itself, for every IDREF in HTML, and this proposal builds on it rather than duplicating it: `<string>` references in ARIA properties resolve exactly as the attribute would, retargeting included.
The two address different gaps.
Reference Target is declared by the component, so it is the right fix wherever the component author is in the loop, and nothing here touches non-ARIA references at all.
ARIA properties cover what remains: components that never adopted it, slotted content the component must describe but does not own, and relations that exist only in some states.

### What about CAS (Cascading Attribute Sheets)?

CAS (Cascading Attribute Sheets) were [first proposed by Tab Atkins in 2012](https://www.xanthir.com/blog/b4K_0) as a way to extend the cascading nature of CSS to HTML attributes.
Brian Kardell also wrote a [polyfill](https://github.com/bkardell/cas) in 2013.
See also: [Dave Rupert: A Use Case for CAS](https://daverupert.com/2022/10/use-case-for-cascading-attribute-sheets/) (2022)

The [proposal was updated in 2025](https://gist.github.com/tabatkins/4074abaf486af5b3f7ac289737e216e4) to a pared down version, which removes reactivity and makes the syntax essentially a syntax abstraction over the equivalent JS. However, restricted invalidation kills the reactivity that is the point for these use cases.

Additionally, a new platform language, with a new MIME type, new inclusion mechanisms, etc, is a lift no individual use case can justify, which makes shipping unlikely.

Instead, piggybacking on CSS may be [theoretically impure](https://www.w3.org/TR/design-principles/#priority-of-constituencies), but it allows **incremental expansion** (start from the pressing use cases, not every attribute; most CAS problems like security-sensitive attributes and collisions vanish once we stop importing wholesale), lets each property grow CSS-native syntax beyond its attribute grammar, and inherits all existing and future CSS machinery: CSSOM, Typed OM, inline styles, animations, transitions, `@supports`, media queries, etc.

## Proposal

### Summary

- CSS properties mirroring ARIA attributes that feed **only the accessibility tree computation** — never reflected to the DOM.
- Properties named after the attribute (`role`, not `aria-role`), but value space designed to fit CSS syntax and semantics.
- Relationship properties take both ids (for compatibility) but also take tree-scoped `<dashed-ident>` references (the `anchor-name` model), also mitigating the [IDREF DX problem](https://github.com/whatwg/html/issues/10143) without attempting to solve it completely.
- The initial value is `auto`, which defers to the corresponding ARIA attribute or implicit semantics if the attribute is not present.
- `none` makes the state/property absent even if the attribute is present.
- ARIA/host-language conflict resolution, unknown-token fallback, and ARIA-in-HTML constraints all apply at used-value time. CSS changes where values come from, not which combinations are conformant.

This is the general form of the [pseudo-element-only `aria-*` properties](https://github.com/w3c/csswg-drafts/issues/14534) proposed in CSSWG in September 2026, which it contains as a subset.

Later:
- CQ/`if()` condition to match on computed `role` (since `:role()` cannot match these)


### Proposed properties

All initial `auto`, not inherited, not animatable, excluded from `all`, elements-only at Level 1.

#### Supporting existing patterns: ARIA

- `role: auto | none | <ident>+`
- Token states/properties, each mirroring its attribute's grammar as `auto | none | <tokens>`: `aria-expanded`, `aria-pressed`, `aria-checked`, `aria-selected`, `aria-current`, `aria-hidden`, `aria-disabled`, `aria-invalid`, `aria-busy`, `aria-sort`, `aria-orientation`, `aria-haspopup`, `aria-modal`, `aria-readonly`, `aria-required`, `aria-multiselectable`, `aria-live`, `aria-atomic`.
- Numeric/string: `aria-level`, `aria-posinset`, `aria-setsize`, `aria-value{min,max,now,text}`, `aria-label`, `aria-description`, `aria-roledescription`, `aria-keyshortcuts`, `aria-placeholder`.
- Relationships: `aria-labelledby`, `aria-describedby`, `aria-details`, `aria-controls`, `aria-errormessage`, `aria-flowto` take `<dashed-ident>+`; `aria-activedescendant` takes one. (`aria-owns` deferred: interacts with `reading-flow` focus scopes.)

#### Encouraging more patterns: focusability

ARIA CSS properties support making CSS widgets that are _already_ functional, actually accessible.

If we want to go even further, and actually _encourage_ the creation of accessible CSS widgets (for all the performance & robustness benefits mentioned earlier), we could also properties to control focusability, such as:
- A property to make elements focusable, such as `interactivity: focusable` (like `tabindex="0"` with more specific keywords for the type of focusability) or just `tab-index: 0 | -1` to mirror the HTML concept (suboptimal design, but compatible).
- A `focusgroup` property, for the same reason

When [previously proposed](https://github.com/w3c/csswg-drafts/issues/13040), pushback centered around focusability not being enough for accessible interactions.
But when focusability is a counterpart to all of the above, that argument doesn't hold.

### Precedence of CSS properties over ARIA attributes

Do these properties override the corresponding ARIA attributes for the purpose of accessibility tree computation?
There are two conflicting precedents:
1. Like SVG geometry properties and HTML presentational attributes, the CSS properties would _override_ the corresponding ARIA attribute values
2. Like inline styles, the CSS properties would have lower precedence than the corresponding attributes

The HTML attributes are higher locality, which would point towards 2.
`!important` overridding them gives us the best of both worlds.

Either way, selectors keep matching the markup attribute, so no cycles.

Note: A propdef column stating whether a corresponding attribute exists and whether the property defaults or overrides it could make SVG geometry and presentational attributes first-class platform concepts rather than legacy warts.

### Element references: `ref-scope` and `ref-name`

IDREF attributes (e.g. `aria-labelledby`) take two types of references:
- `<string>` which corresponds to an ID ref, just like an attribute would
- `<dashed-ident>` which are tree-scoped references modeled on `anchor-name`/`anchor-scope`.

Strawman:

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

Open questions:
- Bare `<dashed-ident>` vs a `ref()` function resolving to a new `<element>` type that all IDREF properties take; the Typed OM could then resolve it to actual element references. Relationships whose scope does not follow the tree could use `ident()` to generate refs. Either way, `id` itself should not become CSS-settable (security).
- `ref-*` vs unifying with `anchor-name`/`anchor-scope`
- name visibility across shadow boundaries.

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

Platform precedent for CSS properties mirroring markup attributes:
- SVG geometry attributes
- HTML presentational attributes (now largely defined in terms of CSS)
- CSS `direction` vs HTML `dir`

### ARIA via CSS

- [`aria-*` properties for pseudo-elements only](https://github.com/w3c/csswg-drafts/issues/14534) (Tab Atkins, September 2026): properties matching every `aria-*` attribute's name and value space, restricted to pseudo-elements. Notes that the general version has been "discussed a few times" and rejected over layering and AT interaction, and argues those objections do not apply to pseudo-elements. Brought to the ARIA WG as [aria#2921](https://github.com/w3c/aria/issues/2921) for TPAC 2026.
- [`:target-current` implies `aria-current`](https://github.com/w3c/csswg-drafts/issues/12659): the ARIA WG [resolved](https://github.com/w3c/aria/issues/2710#issuecomment-3819696897) in January 2026 that the active scroll marker maps to `aria-current="true"` in the platform tree, with no attribute injection, and an author-set `aria-current` winning. A per-feature instance of this proposal's processing model: computed semantics, no DOM reflection, markup wins.
- [CSS-AAM](https://github.com/w3c/css-aam), newly created, is the home for "this CSS state maps to this platform semantics"; `:target-current` is its first entry, and the resolutions this proposal needs would live there.
- `aria-busy` by default during view transitions ([#8447](https://github.com/w3c/csswg-drafts/issues/8447), closed) and a pseudo-class for a range input's current value ([#6354](https://github.com/w3c/csswg-drafts/issues/6354)): smaller requests for the a11y tree to follow CSS state.

### CSS and the a11y tree

CSS already affects the a11y tree, often destructively and by accident:
- `display: contents` ([#3040](https://github.com/w3c/csswg-drafts/issues/3040))
- `list-style: none` in WebKit (strips `list` / `listitem` roles from unstyled `<ul>`/`<ol>`, [WebKit bug 170179](https://bugs.webkit.org/show_bug.cgi?id=170179), [write-up](https://www.scottohara.me/blog/2019/01/12/lists-and-safari.html)).

CSS already controls focusability, as a side effect of other features:
- `overflow: auto` / `scroll` makes a scroll container focusable, and only while there is something to scroll ([demo](https://codepen.io/leaverou/pen/PwPpVBJ)).
- [`image-animation: controlled`](https://drafts.csswg.org/css-image-animation-1/) makes an image focusable so its UA play/pause control can be operated, and only when the image is actually animated; the explainer adds that "screen readers should announce the element distinctly".

CSS already creates interaction that is *not* keyboard accessible, which is the real footgun (collected in [#13040](https://github.com/w3c/csswg-drafts/issues/13040)):
- `resize` adds interactive UI that manipulates CSS properties, with no keyboard path.
- `:hover` makes interactive UI trivial to build and impossible to make keyboard accessible without HTML changes or JS. Authors should not have to choose between DX and accessibility.
- `img:hover, img:focus { image-animation: running }` is the spec's own example, and the spec concedes that "it is not possible to directly make an element focusable in CSS, proper keyboard navigation will require element attributes to be set as well."
- Content that is interactive only under some conditions (a breakpoint, an overflow) has no attribute that can follow the condition.

Similarly, HTML creates inaccessible tooltips via the `title` attribute, that authors are [actively encouraged to avoid](https://html.spec.whatwg.org/multipage/dom.html#the-title-attribute), in favor of much more markup-heavy [interest invokers](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/interest-triggered-tooltips/guide.md).

Where deliberate:
- `reading-flow`/`reading-order` (exposure order, with ARIA WG review in [aria#2241](https://github.com/w3c/aria/issues/2241)),
- `content: url() / "alt"`,
- `::scroll-marker`/`::scroll-button()` and the `image-animation: controlled` control: CSS-generated interactive UI with UA-assigned semantics,
- `interactivity` (shipped, though over sustained a11y objections in [#10711](https://github.com/w3c/csswg-drafts/issues/10711) — cited as fact, not endorsement), and `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040)). That proposal already anticipated this one: making generated content focusable "would obliterate a lot of specialized pseudo-elements being proposed (especially if down the line we can control ARIA roles too)", and it named the false-sense-of-security risk of focusability without semantics, concluding that "the way to solve this is to provide control over those as well, not to make it harder to make a UI keyboard accessible." It also floated expressing a UA default as a UA stylesheet rule (`[tabindex=0] { interactivity: focusable !important }`), the same move this proposal makes for `::tooltip { role: tooltip }`. The objection on [#13040](https://github.com/w3c/csswg-drafts/issues/13040) — that focusability alone amplifies the "focusable = accessible" author mistake — is an argument for completing the contract; this proposal is that completion, and the two should be evaluated as one program.

## Anticipated objections

Most of these were raised against `interactivity: inert` ([#10711](https://github.com/w3c/csswg-drafts/issues/10711), shipped in Chrome 135) and `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040), its 2018 precursor [#3379](https://github.com/w3c/csswg-drafts/issues/3379), the [APA WG review](https://github.com/w3ctag/design-reviews/issues/1055#issuecomment-2725956455), and Alice Boxhall's [threat model post](https://alice.boxhall.au/articles/a-threat-model-for-accessibility-on-the-web/#css-carousels-and-interactivity)).

Objections specific to inertness (un-inerting, steering away from `<dialog>`, set size with inert items) do not transfer and are omitted.

### Layer and venue

- **Wrong layer: CSS is for styling** (Tim Nguyen, Teh, Boxhall, Rivoal 2018, Higley, Yasskin). Semantics, inertness and focusability are content and behavior, not presentation, and CSS's association with presentation makes semantics-blind use *more* likely. Boxhall rejects the "CSS already affects the a11y tree" argument: `display: none` affects it *because* it changes presentation; the a11y tree mirrors the visible experience, and a property that changes semantics without changing visuals is a different category. Higley: CSS may be easier at authoring time but is more fragile and harder to debug.
  CSS has not been only about styling for a very long time: SVG geometry properties, aural properties, `user-select`, `touch-action`, `interactivity`, `resize`, `pointer-events`, `scroll-behavior`/`overscroll-behavior`, `content`/`quotes`, `direction`/`unicode-bidi`, `reading-flow`/`reading-order`, Opera's old `nav-*`. That ship has sailed, and `reading-flow` is the direct counterexample to Boxhall's line: it changes exposure order without changing a pixel, and went through ARIA WG review ([aria#2241](https://github.com/w3c/aria/issues/2241)).
  The actual user need was never "separation of concerns"; it is local (one-off) vs global (rule-based): O(1) author effort, a single source of truth, and no side effects or invalidation to manage. That is exactly why reactive frameworks are popular, and CSS is the platform's only fully reactive language. Theoretical purity sits at the bottom of the priority of constituencies; this has very clear benefits for both end users and developers, who sit at the top.
- **Wrong venue** (Higley, Atkins, APA). CSSWG has fewer a11y participants than Open UI or HTML; `focusgroup` and `focusNext` came out better *because* a11y people were involved early.
  The semantics here are not CSS's to design: every property mirrors an ARIA attribute one to one and feeds the existing ARIA/HTML-AAM computation unchanged, so ARIA WG owns the meaning and CSSWG owns only the syntax and the cascade, the same split `reading-flow` used, and CSS-AAM now exists as the joint venue for exactly these mappings. The companion ARIA in HTML change goes through html-aria, not here.

### Misuse

- **No visual impact, so misuse is invisible** (Boxhall, O'Hara, Suzanne, APA). Authors assume content stays as perceivable to everyone as it is to them; rules setting the property drift out of sync with the rules that change appearance; copy-pasted CSS leaves subtrees inert unnoticed. `user-select: none` and `pointer-events: none` show how such properties get abused. Non-interactive ≠ non-critical.
  `inert` does affect non-AT users, so that objection was overstated even for `interactivity`. More to the point, applying ARIA via HTML or JS has exactly the same property: ARIA attributes have no visual impact either, and that is the baseline being compared against. The "CSS applies it in aggregate" worry does not hold: a JS loop also applies it in aggregate, with less visibility. At least CSS keeps the semantics next to the styling they describe, and the Guardrails ask inspectors to surface rule provenance, which makes a CSS-originated role *more* traceable than a `setAttribute` buried in a handler.
- **"The `aria-hidden` of CSS"** (O'Hara). Browsers had to whittle `aria-hidden` down (ignored on `html`/`body`, undone when focus enters) because of pervasive misuse. No mitigation for author error was designed here, and once shipped it cannot be scoped down without "breaking the web".
  These properties are inputs to the same computation, so every mitigation that exists or is later added to the attribute applies to the property unchanged. And the syntax lends itself to a very simple, very visible mitigation in the UA stylesheet:
  ```css
  html, body { aria-hidden: false !important }
  ```
- **No guardrails; nothing encourages correct use** (Teh, Boxhall; the neglect and threat-model objection). HTML `inert` got normative guidance on inappropriate use after the same concerns; a note in a CSS spec will be ignored. Authors learn by trial and error, and their trials don't include AT or keyboard testing. Consider careless authors and AI-generated code, not just well-intentioned ones.
  True of ARIA attributes and a11y-touching JS today too. But under neglect, the easiest version is what ships, and today the accessible version of CSS-driven UI costs a JS subsystem while the inaccessible one costs nothing. This feature's purpose is making the accessible version the one-declaration version. The Guardrails section is the correct-use encouragement whose absence was the core complaint about `interactivity`, and unlike a spec note, the operability guardrail is enforced by the UA.
- **Cascade/third-party CSS deciding what things are.** Third-party CSS can already destroy semantics wholesale (`display: none`/`contents`, `interactivity: inert`); it just can't repair anything. Auditing moves to the computed tree, which is what AT users actually get, and what inspectors and axe-style runners already evaluate.

### Focus and behavior

- **Focusable ≠ accessible** (Higley). The dominant mistake of developers without a11y understanding is toggling focusability and thinking they are done; it degrades screen reader and voice control users. "An alarmingly large footgun that would increase the frequency of this mistake."
  The other attributes address this: that is the whole point of proposing the two as one program. And the operability guardrail enforces the converse, no widget role without focusability, so the half-done state is not expressible.
- **Focusability couples to behavior, so the switch belongs in JS** (Rivoal 2018, Higley, Yasskin). Focus alone is rarely useful; the handler that reacts to it is JS, so keeping the switch in JS avoids desync. Rivoal: *removing* focusability from CSS is fine; *adding* it is the suspect direction, acceptable only for cases with no attached behavior.
  See the layer argument above, and the next bullet: for most of the property set there is no behavior to desync from.
- **Semantics without behavior.** This objection only bites for widget roles; most of the property set is pure state, where no behavioral contract exists to be missing: `aria-current` from `:target`/`scroll-state(snapped)` (scrollspy, active carousel slide — CSS is the only layer that knows), `aria-expanded` mirroring `:popover-open`/`[open]`/`:checked`/`:focus-within`, `aria-orientation` when `flex-direction` flips at a breakpoint (orientation's source of truth *is* the stylesheet), `aria-hidden` on off-canvas or clamped content, `aria-busy` on skeletons. Where roles are involved, the dominant cases relabel fully working native behavior within ARIA in HTML's existing allowances: checkbox-backed toggles already toggle on Space (`role: button; aria-pressed` or `role: switch` from `:checked` — allowed on checkbox inputs since [html-aria#6](https://github.com/w3c/html-aria/issues/6), though Chrome maps native checkedness onto `switch`, whose state is still `aria-checked`, and not onto `button`'s `aria-pressed`, which is precisely the gap these properties fill), popovers already dismiss and manage focus (`role: tooltip`; `aria-haspopup` on the invoker). Radio-hack tabs have arrow-key traversal natively but `tab` is not currently an allowed role on radio inputs — a relabel this proposal makes worth permitting via a companion ARIA-in-HTML change, the same loosening path checkbox→`button`/`switch` took. These widgets work for keyboard users today and misreport to AT users — the interaction contract is complete, the description is wrong. The residue (`role: button` on a dead `div`) is what the operability guardrail refuses, and invoker commands supply declarative behavior to compose with.
- **CSS can't know the condition** (Chan). A scroller that is only sometimes scrollable also depends on content amount; the browser should make such things focusable itself rather than hand authors another footgun.
  Browsers should address the cases they can, and this provides an escape hatch when their heuristics are wrong. But CSS already knows a lot of conditions that are hard or impossible to detect reliably from JS: `scroll-state()`, `:snapped`, `:target-current`, `:user-invalid`, container and media queries.
- **Granularity** (Oriol). `tabindex=-1` (click-only) is common; if sequential focusability is exposed, so should order; pseudo-element focus is undefined (`activeElement`).
  Pseudo-element focus already needs solving, since we now have plenty of focusable pseudo-elements. The ARIA properties themselves do not touch focus order at all.
- **Focus management is already "wonky and magical"** (Kardell 2018, Boxhall). Don't add new complexity until the existing mess is dealt with.
  This proposal adds no focus management: the properties are write-only inputs to the accessibility tree and never move focus.

### Semantics

- **Conditional semantics.** When the UI is conditional on MQs, semantics that can't follow are wrong, not stable; the failure exists today with reversed polarity. `auto` + markup remain for anything that shouldn't vary.
- **Non-CSS consumers / failed loads.** `auto` keeps markup semantics as the baseline, so a missing stylesheet degrades to today's behavior, where a missing or broken script leaves every synced attribute stale. Consumers that discard CSS already discard `reading-flow` and every layout-dependent aspect of meaning.
- **Which pseudo-elements need author ARIA at all?** (Higley, on [#14534](https://github.com/w3c/csswg-drafts/issues/14534)). Interactive pseudo-elements like the interest invoker touch button would not benefit from customization, so interactivity alone does not imply a need for custom ARIA in CSS.
  Agreed that it does not imply it; the [use cases](./use-cases.md) show where the need is concrete: `::picker()` as a grid, `::details-content` as a labelled region, `::tooltip` carrying its role, and the carousel pseudo-elements' labels. And the element half of this proposal does not depend on the pseudo-element half.
- **Exclude roles; specify each pseudo-element's semantics in its own module** (Crissov, on [#14534](https://github.com/w3c/csswg-drafts/issues/14534)). Most pseudo-elements should be `none`, the few meaningful roles belong in the CSS module that defines the pseudo-element, not in author stylesheets.
  Both, not either: per-module UA defaults are exactly what a UA stylesheet rule like `::tooltip { role: tooltip }` expresses, and an author override through the same property is what the `::picker()` case needs. Author-overridable UA defaults are how every other property works.
- **Many of these attributes only apply to certain elements.** So do the SVG geometry properties, `object-fit`/`object-position`, `appearance`, `field-sizing` and `frame-sizing`, and nothing broke. None of these inherit.
