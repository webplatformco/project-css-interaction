# ARIA in CSS

<!-- [css-ui] Proposal: `aria-*` and `role` properties, with tree-scoped element references -->

> [!IMPORTANT]
> This is a work in progress and still very rough around the edges, but any feedback is welcome!

**TL;DR:** `role` and `aria-*` as CSS properties, with tree-scoped element references.
Two tiers: first, make the CSS-driven interaction authors already build describable to AT; second, encourage more of it by also letting CSS control focusability.

- [Problem statement \& user needs](#problem-statement--user-needs)
	- [Use cases](#use-cases)
	- [Why CSS?](#why-css)
	- [Cross-root ARIA](#cross-root-aria)
- [CSS and the a11y tree](#css-and-the-a11y-tree)
- [Proposal](#proposal)
	- [Summary](#summary)
	- [Element references: `ref-scope` and `ref-name`](#element-references-ref-scope-and-ref-name)
		- [Cross-root references](#cross-root-references)
	- [Tier 1: Supporting existing patterns](#tier-1-supporting-existing-patterns)
		- [ARIA properties](#aria-properties)
	- [Tier 2: Encouraging more patterns](#tier-2-encouraging-more-patterns)
	- [Precedence of CSS properties over ARIA attributes](#precedence-of-css-properties-over-aria-attributes)
- [Processing model](#processing-model)
- [Guardrails](#guardrails)
- [Alternatives considered](#alternatives-considered)
	- [Cascading Attribute Sheets (CAS)](#cascading-attribute-sheets-cas)
	- [`ElementInternals`](#elementinternals)
	- [Reference Target alone](#reference-target-alone)
	- [Pseudo-element-only `aria-*` properties](#pseudo-element-only-aria--properties)
	- [Per-feature mappings in CSS-AAM](#per-feature-mappings-in-css-aam)
- [Prior art](#prior-art)
	- [Attributes from CSS](#attributes-from-css)
	- [ARIA via CSS](#aria-via-css)
- [Anticipated objections](#anticipated-objections)
	- [Layer](#layer)
	- [Misuse](#misuse)
	- [Tooling](#tooling)
	- [Semantics](#semantics)


## Problem statement & user needs

An increasing number of common interaction patterns can be implemented via HTML and CSS alone, but making them accessible requires manual HTML boilerplate (tedious and error-prone), or flimsy JS to keep UI state in sync with ARIA attributes.

Additionally, for **CSS pseudo-elements** such as `::picker()`, `::details-content`, `::part()`, `::slotted()` etc., there is _no_ way to convey their accessibility semantics to the accessibility tree.

As a result, accessibility is often skipped, buggy, pushed to "later", or deemed out of scope altogether.

Authors are often discouraged from simpler, faster to load HTML & CSS patterns on the basis of accessibility concerns, and steered into heavy markup and JS solutions as the solution.
A CSS framework that [shelved its tabs component](https://github.com/cirthcss/cirth/issues/61) for exactly this reason put it plainly:

> CSS can select and present a state, but it cannot manage focus or update the corresponding ARIA attributes. Therefore, correct ARIA tabs currently require script.

Take the simplest such widget: a checkbox styled as a toggle button.
Space toggles it, `:checked` styles it, no script.
But AT hears a checkbox, and making it hear a toggle button means a script that copies `checked` into `aria-pressed` on load and on every `change`, for every instance, forever.
With this proposal, the same fact is stated once, next to the rule that already styles it:

```css
.toggle {
	role: button;
	aria-pressed: false;
}

.toggle:checked {
	aria-pressed: true;
}
```

What if instead of discouraging authors from doing what feels natural (always an uphill battle), we paved the cowpaths, and _made_ the CSS patterns _actually_ accessible?

The benefits are quite significant, for both developers and end-users:
- Lower friction to make experiences accessible directly translates to **more accessible experiences**.
- **Semantics move up a level of abstraction.** Today the only unit that can carry ARIA once for all its uses is a JS component; a class, a CSS framework or the UA stylesheet cannot. In CSS, whoever defines a pattern specifies it once, and consumers get it automatically (e.g. by using the class).
- **Co-location resists drift.** The dominant scripted-ARIA failure is state desync; deriving semantics from the same selector that drives presentation makes them a single expression.
- **Fault tolerance.** Scripted ARIA fails whole: In JS _any_ uncaught error breaks the entire file. CSS is fault-tolerant by design. An incorrect declaration is dropped without affecting the rest of the sheet.
- **Fewer wrappers.** Built-in elements are often wrapped in components _precisely_ to encapsulate accessibility logic, which today requires JS. Handling accessibility semantics in CSS lets many of these wrappers become a CSS class on a built-in element, and the decrease in complexity compounds downstream.
Of [Web Awesome](https://github.com/shoelace-style/webawesome)'s 73 components, 39 (53%) could have been a simple CSS class if it weren't for JS syncing state to ARIA (7 of them would also need `focusgroup`); see the [component library audit](./component-libraries.md).

### Use cases

The [full list](./use-cases.md) falls into five buckets:

- **[Manual ARIA boilerplate](./use-cases.md#manual-aria-boilerplate)**: facts CSS already knows, added by hand on every instance: `aria-hidden="true"` on every decorative icon, `aria-current="page"` on the current nav link, a unique id on every dialog heading so `aria-labelledby` can point at it, etc.
- **[JS that exists only to sync CSS state into ARIA](./use-cases.md#js-to-sync-aria-state)**: `aria-invalid` from `:user-invalid`, `aria-current` from `:target-current`, `aria-pressed` or `aria-selected` from `:checked` (toggle buttons, tabs, `<select>` listboxes restyled as checkbox groups), etc.
- **[Pseudo-elements that cannot carry ARIA at all](./use-cases.md#pseudo-elements)**: `::picker()` laid out as a grid, carousel pseudo-elements with UA-assigned roles and no way to name them, `::details-content` as a labelled region or a menu, whether `::checkmark` is announced, `::slotted()` / `::part()` for semantics across a shadow boundary, from either side, etc.
- **[Markup the author cannot edit](./use-cases.md#markup-you-cannot-edit)**: Markdown output, third-party widgets, components that did not forward labels, etc.
- **[Built-in browser magic, made visible](./use-cases.md#built-in-browser-magic-made-visible)**: implicit roles, `::tooltip`'s role, the `aria-hidden` corrections browsers apply silently, all as UA stylesheet rules that can be inspected.

### Why CSS?

CSS is the only web platform language built around _reactive invariants_:
authors write rules declaring what applies to which (pseudo-)elements under which conditions, and the UA combines whatever matches and takes care of the hard bits: invalidation and state sync.
There are no initialization steps, listeners, or side effect cleanup.
This matters because invalidation is _hard_, and its difficulty grows with the number of conditions.
Intent as simple as "apply X to buttons, except when hovered or focused" takes two lines of CSS, but ~50 lines of (much buggier) JS.
Add media queries to the mix, and the complexity of keeping everything in sync grows even further.
The Chrome team made the same point from the other direction in the TAG review of `interactivity: inert` ([design-reviews#1055](https://github.com/w3ctag/design-reviews/issues/1055#issuecomment-2762905148)): which nodes, at which screen size, obscured by what, "CSS does that with selectors", "with media queries", and "is already orchestrating those moments".

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

**Outside the component.** Every existing path across the boundary (form-associated custom elements, element reflection, [Reference Target](https://github.com/WICG/webcomponents/blob/gh-pages/proposals/reference-target-explainer.md)) requires the component's cooperation.
A consumer cannot fix a component that did not describe its parts, even when a selector can reach both ends of the relation.

Selectors already cross the boundary in both directions (`::slotted()`, `::part()`, `:host`), so a stylesheet is the one place where both sides can be described.
See [Cross-root references](#cross-root-references) for how.

## CSS and the a11y tree

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
- the `links` / `tabs` keyword of [`scroll-marker-group`](https://drafts.csswg.org/css-overflow-5/#scroll-marker-group-property), which picks the roles of `::scroll-marker-group` and `::scroll-marker`: `navigation` + `link`, or `tablist` + `tab` with a focusgroup and `tabpanel` on the originating elements.
- `interactivity` (shipped, though over sustained a11y objections in [#10711](https://github.com/w3c/csswg-drafts/issues/10711), cited as fact, not endorsement), and `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040)). That proposal already anticipated this one: making generated content focusable "would obliterate a lot of specialized pseudo-elements being proposed (especially if down the line we can control ARIA roles too)", and it named the false-sense-of-security risk of focusability without semantics, concluding that "the way to solve this is to provide control over those as well, not to make it harder to make a UI keyboard accessible." It also floated expressing a UA default as a UA stylesheet rule (`[tabindex=0] { interactivity: focusable !important }`), the same move this proposal makes for `::tooltip { role: tooltip }`. The objection on [#13040](https://github.com/w3c/csswg-drafts/issues/13040), that focusability alone amplifies the "focusable = accessible" author mistake, is an argument for completing the contract; this proposal is that completion, and the two should be evaluated as one program.

## Proposal

### Summary

- CSS properties mirroring ARIA attributes that feed **only the accessibility tree computation**, never reflected to the DOM.
- Properties named after the attribute (`role`, not `aria-role`), but value space designed to fit CSS syntax and semantics.
- Relationship properties take ids (for compatibility) but also tree-scoped `<dashed-ident>` references (the `anchor-name` model), also mitigating the [IDREF DX problem](https://github.com/whatwg/html/issues/10143) without attempting to solve it completely.
- The initial value is `auto`, which defers to the corresponding ARIA attribute, or to implicit semantics if the attribute is not present.
- An explicit absence value makes the state/property absent even if the attribute is present. **Issue:** `none` is the natural CSS spelling, but is also a valid value of `role`, `aria-sort` and `aria-autocomplete`. See [ARIA properties](#aria-properties).
- Attributes are assumed to behave like presentational hints: beaten by any author declaration, winning over the UA stylesheet. See [Precedence](#precedence-of-css-properties-over-aria-attributes).
- ARIA/host-language conflict resolution, unknown-token fallback, and ARIA-in-HTML constraints all apply at used-value time. CSS changes where values come from, not which combinations are conformant, with one deliberate exception (see [Guardrails](#guardrails)).

The proposal comes in two tiers.
**Tier 1** describes interaction CSS can already build, so that widgets which work for keyboard users today stop misreporting to AT users.
**Tier 2** extends CSS's control over focusability, so that more such widgets can be built in the first place.
Tier 1 stands on its own; Tier 2 is sketched here because the two should be evaluated together (see [Tier 2](#tier-2-encouraging-more-patterns)).

This is the general form of the [pseudo-element-only `aria-*` properties](https://github.com/w3c/csswg-drafts/issues/14534) proposed in CSSWG in September 2026, which it contains as a subset.

Later:
- CQ/`if()` condition to match on computed `role` (since `:role()` cannot match these)

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

#### Cross-root references

The [cross-root problem](#cross-root-aria) has two sides, and the same properties serve both.

**Inside the component**, the component stylesheet declares semantics once, for the host and for slotted content alike:

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

**Outside the component**, where a selector reaches both ends of the relation, a consumer can fix a component that did not:

```css
label:has(> fancy-input) {
	ref-name: --label;
	ref-scope: --label;
}

label > fancy-input::part(input) {
	aria-labelledby: --label;
}
```

Note that `::part()` is an opt-in to styling, not to semantics; whether exposing a part should also expose its semantics to the consumer is an open question.

**Relation to Reference Target.** Reference Target lets a shadow host nominate an element inside its shadow root as the target of any IDREF pointing at the host, so `<label for>`, `aria-labelledby`, `popovertarget` and the rest work across the boundary with no change on the referencing side.
That fixes the reference primitive itself, for every IDREF in HTML, and this proposal builds on it rather than duplicating it: `<string>` references in ARIA properties resolve exactly as the attribute would, retargeting included.
The two address different gaps.
Reference Target is declared by the component, so it is the right fix wherever the component author is in the loop, and nothing here touches non-ARIA references at all.
ARIA properties cover what remains: components that never adopted it, slotted content the component must describe but does not own, and relations that exist only in some states.

### Tier 1: Supporting existing patterns

All properties: initial `auto`, not inherited, not animatable, excluded from `all`, apply to elements and pseudo-elements.

#### ARIA properties

- `role: auto | none | <ident>+`
- Token states/properties, each mirroring its attribute's grammar as `auto | none | <tokens>`: `aria-expanded`, `aria-pressed`, `aria-checked`, `aria-selected`, `aria-current`, `aria-hidden`, `aria-disabled`, `aria-invalid`, `aria-busy`, `aria-sort`, `aria-orientation`, `aria-haspopup`, `aria-modal`, `aria-readonly`, `aria-required`, `aria-multiselectable`, `aria-live`, `aria-atomic`.
- Numeric/string: `aria-level`, `aria-posinset`, `aria-setsize`, `aria-value{min,max,now,text}`, `aria-label`, `aria-description`, `aria-roledescription`, `aria-keyshortcuts`, `aria-placeholder`.
- Relationships: `aria-labelledby`, `aria-describedby`, `aria-details`, `aria-controls`, `aria-errormessage`, `aria-flowto` take `<dashed-ident>+`; `aria-activedescendant` takes one. (`aria-owns` deferred: interacts with `reading-flow` focus scopes.)

Open question, the absence keyword: `none` above means "no value, even if the attribute has one", but `role="none"` (a synonym of `presentation`), `aria-sort="none"` and `aria-autocomplete="none"` are valid attribute values, so `role: none` is ambiguous.
Options: a distinct keyword for absence (e.g. `omit`), or keep `none` as absence and require the `presentation` spelling for the role, since the other colliding values are equivalent to absence anyway.


### Tier 2: Encouraging more patterns

Tier 1 makes CSS widgets that are _already_ operable describable to AT.
Tier 2 is about operability itself: if we want to _encourage_ accessible CSS widgets (for all the performance and robustness benefits above), CSS also needs to control focusability.
Candidates:
- A property to make elements focusable, such as `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040), like `tabindex="0"` with more specific keywords for the type of focusability), or a `tab-index: 0 | -1` property mirroring the HTML concept (suboptimal design, but compatible).
- A `focusgroup` property, for the same reason.

This document only sketches Tier 2, and nothing in Tier 1 depends on it.
But the two should be evaluated as one program.
When focusability was [previously proposed](https://github.com/w3c/csswg-drafts/issues/13040) on its own, pushback centered on focusability not being enough for accessible interactions, and that is correct: focusable ≠ accessible.
Tier 1 is the missing half of that contract, and its [operability guardrail](#guardrails) enforces the converse.
With both, the half-done state is not expressible.

### Precedence of CSS properties over ARIA attributes

Do these properties override the corresponding ARIA attributes for the purpose of accessibility tree computation?
There are two conflicting precedents:

1. The attributes could be treated like inline styles with the same precedence rules, with a conflict resolution policy for when _both_ are specified. I think this makes more sense from a locality perspective, and is a model authors are familiar with.
2. Like SVG geometry properties, HTML presentational attributes and `dir`, the CSS properties would _override_ the corresponding ARIA attribute values. This was primarily done for compatibility reasons, and is less intuitive.

**This document assumes 1:** the attribute behaves like a presentational hint, exactly as `dir` does for `direction`. It beats the UA stylesheet and loses to any author declaration, and `auto` (the initial value) defers to it.
This is what the use cases need:
- UA-stylesheet defaults such as `::tooltip { role: tooltip }` or `h1 { role: heading }` stay overridable by markup.
- Author CSS can repair markup it cannot edit.
- The absence value works without `!important`.
- A UA `!important` rule (e.g. `html, body { aria-hidden: false !important }`) can still shut down known abuse.

The argument for 2 is locality: the attribute sits on the element, like `style` does.
But under 2, repair of uneditable markup, framework-level defaults and the absence value all require `!important`, which turns the escape hatch into the common path.

Either way, selectors keep matching the markup attribute, so no cycles.

Note: A propdef column stating whether a corresponding attribute exists and whether the property defaults or overrides it could make SVG geometry and presentational attributes first-class platform concepts rather than legacy warts.

## Processing model

- **Write-only, downstream.** Inputs to the a11y tree computation only. No DOM reflection: attributes are unchanged, `el.role`/`el.ariaExpanded` keep reflecting attributes, MutationObservers see nothing; `Element.computedRole` and AT see the result.
- **No circularity by construction.** Selectors match source semantics only; nothing matches this computation's output. Requires retracting the unimplemented `:role()` resolution ([#3596](https://github.com/w3c/csswg-drafts/issues/3596), minuted as the price of the opposite choice: "we can't do both"). The identical trade was made in [#13040](https://github.com/w3c/csswg-drafts/issues/13040) re `:focusable` ([#7269](https://github.com/w3c/csswg-drafts/issues/7269)), and it is how `:dir()` already relates to `direction`.
- Updates propagate like attribute mutation; AT already handles dynamic semantics.


## Guardrails

- **Widget roles require operability.** `button`, `tab`, `checkbox`, etc. don't take effect on elements that are neither focusable by some mechanism (native, `tabindex`, `interactivity: focusable`) nor managed by a focusable composite (`tablist`, `listbox` with `aria-activedescendant`, a `focusgroup`). No announcing controls nobody can operate. Structural/landmark roles are exempt.
  This is the one place where the CSS property is deliberately stricter than the attribute: the attribute predates any way to enforce it, and the property's purpose is to make the accessible version the one-declaration version, not to make the broken one cheaper.
- Existing ARIA validity machinery applies unchanged, at used-value time.
- Implementations should surface rule provenance for CSS-originated semantics in a11y inspectors.


## Alternatives considered

### Cascading Attribute Sheets (CAS)

CAS (Cascading Attribute Sheets) were [first proposed by Tab Atkins in 2012](https://www.xanthir.com/blog/b4K_0) as a way to extend the cascading nature of CSS to HTML attributes.
Brian Kardell also wrote a [polyfill](https://github.com/bkardell/cas) in 2013.
See also: [Dave Rupert: A Use Case for CAS](https://daverupert.com/2022/10/use-case-for-cascading-attribute-sheets/) (2022)

The [proposal was updated in 2025](https://gist.github.com/tabatkins/4074abaf486af5b3f7ac289737e216e4) to a pared down version, which removes reactivity and makes the syntax essentially a syntax abstraction over the equivalent JS. However, restricted invalidation kills the reactivity that is the point for these use cases.

Additionally, a new platform language, with a new MIME type, new inclusion mechanisms, etc., is a lift no individual use case can justify, which makes shipping unlikely.

Piggybacking on CSS is less pure, but [purity ranks last](https://www.w3.org/TR/design-principles/#priority-of-constituencies), and it allows **incremental expansion** (start from the pressing use cases, not every attribute; most CAS problems like security-sensitive attributes and collisions vanish once we stop importing wholesale), lets each property grow CSS-native syntax beyond its attribute grammar, and inherits all existing and future CSS machinery: CSSOM, Typed OM, inline styles, animations, transitions, `@supports`, media queries, etc.

### `ElementInternals`

The ARIA mixin on `ElementInternals` is the platform's designated path for component semantics, and it is imperative, host-only and component-only.
It cannot describe slotted content, cannot be used by a consumer, cannot follow a CSS condition, and every state change must remember to update it.
The [custom element use cases](./use-cases.md#custom-element-semantics-elementinternals) show the resulting double write at scale.
Extending it would not reach the other four use-case buckets.

### Reference Target alone

Reference Target fixes IDREFs across the shadow boundary for every attribute, and this proposal builds on it.
But it must be declared by the component, so it cannot help a consumer repair a component that did not adopt it, cannot describe slotted content, and does not express relations that exist only in some states.
See [Relation to Reference Target](#cross-root-references).

### Pseudo-element-only `aria-*` properties

[#14534](https://github.com/w3c/csswg-drafts/issues/14534) restricts the same properties to pseudo-elements, on the grounds that the layering and AT-interaction objections to the general version do not apply there.
It is a strict subset of this proposal, with the same processing model and the same cascade cost.
Stopping there leaves the four other use-case buckets unaddressed, and leaves the cross-root cases half-solved: `::slotted()` and `::part()` select real elements, so a pseudo-element-only rule already sets ARIA on elements, just not on the ones the author owns.
It also leaves the headline pseudo-element case half-solved: a `::picker()` exposed as a `grid` needs its `<option>`s exposed as `gridcell`s, and the options are elements whose role has to switch with the same media query as the layout ([grid picker](./use-cases.md#customizable-select-with-grid-picker)).

### Per-feature mappings in CSS-AAM

`:target-current` now maps to `aria-current` by [ARIA WG resolution](https://github.com/w3c/aria/issues/2710#issuecomment-3819696897), and CSS-AAM is the home for such mappings.
Each one is a WG resolution and an implementation per feature, covering only the pattern the UA anticipated.
This proposal is the general form: the same processing model, with authors able to declare the mapping for their own patterns.
The two are complementary, and UA-level mappings can be expressed as UA stylesheet rules in these properties.

## Prior art

### Attributes from CSS

Platform precedent for CSS properties mirroring markup attributes:
- **`dir` / `direction`** is the closest precedent on every axis. `dir` is semantic (bidi), not visual. HTML maps it through UA stylesheet rules (`[dir]:dir(rtl) { direction: rtl }`), so any author declaration wins, which is the precedence model assumed here. And `:dir()` matches the markup-derived directionality, never the computed `direction`, which is the no-circularity split this proposal makes.
- SVG geometry attributes: presentation attributes, beaten by any author declaration.
- HTML presentational attributes, now defined as presentational hints: author-origin declarations of zero specificity.

### ARIA via CSS

- [`aria-*` properties for pseudo-elements only](https://github.com/w3c/csswg-drafts/issues/14534) (Tab Atkins, September 2026): properties matching every `aria-*` attribute's name and value space, restricted to pseudo-elements. Notes that the general version has been "discussed a few times" and rejected over layering and AT interaction, and argues those objections do not apply to pseudo-elements. Brought to the ARIA WG as [aria#2921](https://github.com/w3c/aria/issues/2921) for TPAC 2026.
- [`:target-current` implies `aria-current`](https://github.com/w3c/csswg-drafts/issues/12659): the ARIA WG [resolved](https://github.com/w3c/aria/issues/2710#issuecomment-3819696897) in January 2026 that the active scroll marker maps to `aria-current="true"` in the platform tree, with no attribute injection, and an author-set `aria-current` winning. A per-feature instance of this proposal's processing model: computed semantics, no DOM reflection, markup wins over the UA mapping.
- [CSS-AAM](https://github.com/w3c/css-aam), newly created, is the home for "this CSS state maps to this platform semantics"; `:target-current` is its first entry, and the resolutions this proposal needs would live there.
- `aria-busy` by default during view transitions ([#8447](https://github.com/w3c/csswg-drafts/issues/8447), closed) and a pseudo-class for a range input's current value ([#6354](https://github.com/w3c/csswg-drafts/issues/6354)): smaller requests for the a11y tree to follow CSS state.


## Anticipated objections

Many of these were raised against `interactivity: inert` ([#10711](https://github.com/w3c/csswg-drafts/issues/10711), shipped in Chrome 135) and `interactivity: focusable` ([#13040](https://github.com/w3c/csswg-drafts/issues/13040), its 2018 precursor [#3379](https://github.com/w3c/csswg-drafts/issues/3379), the [APA WG review](https://github.com/w3ctag/design-reviews/issues/1055#issuecomment-2725956455), and a [threat model post](https://alice.boxhall.au/articles/a-threat-model-for-accessibility-on-the-web/#css-carousels-and-interactivity)), and transfer to any CSS that touches the accessibility tree.
Objections specific to focusability or inertness (focusable ≠ accessible, focus order, un-inerting, set size with inert items) belong to Tier 2 and are omitted here.

### Layer

- **Wrong layer: CSS is for styling** ([#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2306101100), [#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2765041902), [#13040 (comment)](https://github.com/w3c/csswg-drafts/issues/13040#issuecomment-3487928582), [#13040 (comment)](https://github.com/w3c/csswg-drafts/issues/13040#issuecomment-3503856989)). Semantics, inertness and focusability are content and behavior, not presentation, and CSS's association with presentation makes semantics-blind use *more* likely. A [response on #10711](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2359573310) rejects the "CSS already affects the a11y tree" argument: `display: none` affects it *because* it changes presentation; the a11y tree mirrors the visible experience, and a property that changes semantics without changing visuals is a different category. [Also raised](https://github.com/w3c/csswg-drafts/issues/13040#issuecomment-3487928582): CSS may be easier at authoring time but is more fragile and harder to debug.
  CSS has not been only about styling for a very long time: SVG geometry properties, aural properties, `user-select`, `touch-action`, `interactivity`, `resize`, `pointer-events`, `scroll-behavior`/`overscroll-behavior`, `content`/`quotes`, `direction`/`unicode-bidi`, `reading-flow`/`reading-order`, Opera's old `nav-*`. That ship has sailed, and `reading-flow` is the direct counterexample to that line: it changes exposure order without changing a pixel, and went through ARIA WG review ([aria#2241](https://github.com/w3c/aria/issues/2241)).
  The actual user need was never "separation of concerns"; it is local (one-off) vs global (rule-based): O(1) author effort, a single source of truth, and no side effects or invalidation to manage. That is exactly why reactive frameworks are popular, and CSS is the platform's only fully reactive language. Theoretical purity sits at the bottom of the [priority of constituencies](https://www.w3.org/TR/design-principles/#priority-of-constituencies); this has very clear benefits for both end users and developers, who sit at the top.
<!-- - **`ElementInternals` is the designated path for declaring semantics outside markup.** The ARIA mixin exists precisely so that semantics are set by the code that owns the behavior, and AOM deliberately did not add a general write path.
  `ElementInternals` is host-only, component-only and imperative; it cannot describe slotted content, cannot be used by a consumer, and cannot follow a condition. It covers one of the five use-case buckets, and the [custom element use cases](./use-cases.md#custom-element-semantics-elementinternals) show the double write it forces even there. This proposal keeps its processing model (computed semantics, no DOM reflection) and extends it to where selectors already reach. -->
- **Fix HTML instead.** `focusgroup`, invoker commands, Reference Target and `<input switch>` are closing the same gaps declaratively, in the layer that owns semantics.
  They are complementary, and this proposal depends on them for behavior. What HTML cannot do is follow a condition: a breakpoint, an overflow, a `:checked`, a `:popover-open`. Every synced attribute in the [use cases](./use-cases.md#js-to-sync-aria-state) is HTML that was already declared correctly for one state and is wrong in the other. HTML also has no way to reach pseudo-elements at all.

### Misuse

- **No visual impact, so misuse is invisible** ([#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2765041902), [#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2777119047), [#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-3358267254), the [APA WG review](https://github.com/w3ctag/design-reviews/issues/1055#issuecomment-2725956455)). Authors assume content stays as perceivable to everyone as it is to them; rules setting the property drift out of sync with the rules that change appearance; copy-pasted CSS leaves subtrees inert unnoticed. `user-select: none` and `pointer-events: none` show how such properties get abused. Non-interactive ≠ non-critical.
  `inert` does affect non-AT users, so that objection was overstated even for `interactivity`. More to the point, applying ARIA via HTML or JS has exactly the same property: ARIA attributes have no visual impact either, and that is the baseline being compared against. The "CSS applies it in aggregate" worry does not hold: a JS loop also applies it in aggregate, with less visibility. At least CSS keeps the semantics next to the styling they describe, and the Guardrails ask inspectors to surface rule provenance, which makes a CSS-originated role *more* traceable than a `setAttribute` buried in a handler.
- **"The `aria-hidden` of CSS"** ([#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2777119047)). Browsers had to whittle `aria-hidden` down (ignored on `html`/`body`, undone when focus enters) because of pervasive misuse. No mitigation for author error was designed here, and once shipped it cannot be scoped down without "breaking the web".
  These properties are inputs to the same computation, so every mitigation that exists or is later added to the attribute applies to the property unchanged. And the syntax lends itself to a very simple, very visible mitigation in the UA stylesheet:
  ```css
  html, body { aria-hidden: false !important }
  ```
- **No guardrails; nothing encourages correct use** ([#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2306101100), [#10711 (comment)](https://github.com/w3c/csswg-drafts/issues/10711#issuecomment-2765041902), the [threat model post](https://alice.boxhall.au/articles/a-threat-model-for-accessibility-on-the-web/#css-carousels-and-interactivity)). HTML `inert` got normative guidance on inappropriate use after the same concerns; a note in a CSS spec will be ignored. Authors learn by trial and error, and their trials don't include AT or keyboard testing. Consider careless authors and AI-generated code, not just well-intentioned ones.
  True of ARIA attributes and a11y-touching JS today too. But under neglect, the easiest version is what ships, and today the accessible version of CSS-driven UI costs a JS subsystem while the inaccessible one costs nothing. This feature's purpose is making the accessible version the one-declaration version. The Guardrails section is the correct-use encouragement whose absence was the core complaint about `interactivity`, and unlike a spec note, the operability guardrail is enforced by the UA.
- **Cascade/third-party CSS deciding what things are.** Third-party CSS can already destroy semantics wholesale (`display: none`/`contents`, `interactivity: inert`); it just can't repair anything. Auditing moves to the computed tree, which is what AT users actually get, and what inspectors and axe-style runners already evaluate.
- **Semantics churn.** State driven by `:hover`, `:focus-within` or a scroll-state query flips as often as the condition does, so AT users get a stream of announcements, and `aria-live` from a stylesheet could create announcement storms.
  A JS mirror of the same condition produces the same stream; the churn is a property of the condition, not of the language expressing it. The properties are not animatable and do not transition, so there is no CSS-specific amplification. Which conditions make sense as sources of which states is exactly the guidance CSS-AAM exists to give, and it can give it once rather than per framework.

### Tooling

- **Everything that reasons about ARIA reads the DOM.** Validators (ARIA in HTML conformance), linters, axe-style runners in CI, and `getByRole` in Testing Library and Playwright compute roles from attributes, so CSS-set semantics are invisible to them, or worse, flagged as missing.
  Already true for `ElementInternals` roles, UA mappings like `:target-current`, and every implicit role of a pseudo-element, which is why `Element.computedRole`/`computedLabel`, the DevTools accessibility protocol and Playwright's a11y snapshots exist. The computed tree is the right source of truth because it is what AT users get. This is a real migration cost for the testing ecosystem, and the provenance guardrail is what makes it tractable: a CSS-originated role must be attributable to a rule.
- **Strings in stylesheets resist localization** (`aria-label: "Close"`). Translation pipelines process markup and message catalogs, not CSS.
  The same holds for `content: "…"` and `content: … / "alt"`, which the WG accepted with the same caveat. Most of the property set is tokens and relationships, not strings; the string cases in the use cases come from `attr()`, counters, or `aria-labelledby`, which keep the text in markup. Guidance: prefer `attr()` and relationships over literal strings.
- **Which rule won?** Specificity, `!important` and origin make the resulting role harder to reason about than an attribute on the element.
  This is the same question as for any property, and inspectors already answer it: the winning rule, its source, and the overridden ones. The Guardrails ask for the same view in the accessibility pane. An attribute set by one of several JS code paths has no equivalent.

### Semantics

- **Semantics without behavior.** This objection only bites for widget roles; most of the property set is pure state, where no behavioral contract exists to be missing: `aria-current` from `:target`/`scroll-state(snapped)` (scrollspy, active carousel slide — CSS is the only layer that knows), `aria-expanded` mirroring `:popover-open`/`[open]`/`:checked`/`:focus-within`, `aria-orientation` when `flex-direction` flips at a breakpoint (orientation's source of truth *is* the stylesheet), `aria-hidden` on off-canvas or clamped content, `aria-busy` on skeletons. Where roles are involved, the dominant cases relabel fully working native behavior within ARIA in HTML's existing allowances: checkbox-backed toggles already toggle on Space (`role: button; aria-pressed` or `role: switch` from `:checked` — allowed on checkbox inputs since [html-aria#6](https://github.com/w3c/html-aria/issues/6), though Chrome maps native checkedness onto `switch`, whose state is still `aria-checked`, and not onto `button`'s `aria-pressed`, which is precisely the gap these properties fill), popovers already dismiss and manage focus (`role: tooltip`; `aria-haspopup` on the invoker). Radio-hack tabs have arrow-key traversal natively but `tab` is not currently an allowed role on radio inputs — a relabel this proposal makes worth permitting via a companion ARIA-in-HTML change, the same loosening path checkbox→`button`/`switch` took. These widgets work for keyboard users today and misreport to AT users — the interaction contract is complete, the description is wrong. The residue (`role: button` on a dead `div`) is what the operability guardrail refuses, and invoker commands supply declarative behavior to compose with.
- **Conditional semantics.** When the UI is conditional on MQs, semantics that can't follow are wrong, not stable; the failure exists today with reversed polarity. `auto` + markup remain for anything that shouldn't vary.
- **Non-CSS consumers / failed loads.** `auto` keeps markup semantics as the baseline, so a missing stylesheet degrades to today's behavior, where a missing or broken script leaves every synced attribute stale. Before stylesheets load, the tree is the markup's, exactly as before a script runs; and since the a11y tree already depends on style resolution for `display`, `visibility` and `content`, there is no new moment at which it could be computed without CSS. Consumers that discard CSS already discard `reading-flow` and every layout-dependent aspect of meaning.
- **Which pseudo-elements need author ARIA at all?** ([#14534 (comment)](https://github.com/w3c/csswg-drafts/issues/14534#issuecomment-5961706551)). Interactive pseudo-elements like the interest invoker touch button would not benefit from customization, so interactivity alone does not imply a need for custom ARIA in CSS.
  Agreed that it does not imply it; the [use cases](./use-cases.md) show where the need is concrete: `::picker()` as a grid, `::details-content` as a labelled region, `::tooltip` carrying its role, and the carousel pseudo-elements' labels. And the element half of this proposal does not depend on the pseudo-element half.
- **Exclude roles; specify each pseudo-element's semantics in its own module** ([#14534 (comment)](https://github.com/w3c/csswg-drafts/issues/14534#issuecomment-5968643406)). Most pseudo-elements should be `none`, the few meaningful roles belong in the CSS module that defines the pseudo-element, not in author stylesheets.
  Both, not either: per-module UA defaults are exactly what a UA stylesheet rule like `::tooltip { role: tooltip }` expresses, and an author override through the same property is what the `::picker()` case needs. Author-overridable UA defaults are how every other property works.
  [#14316](https://github.com/w3c/csswg-drafts/issues/14316) is the live case: the CSSWG made the per-module decision for `::checkmark` and `::picker-icon`, and it is contested precisely because a hard-coded exclusion is one "an author or user cannot override"; the compromise being sought is a default plus an opt-out, which is a UA rule ([Built-in browser magic](./use-cases.md#built-in-browser-magic-made-visible)).
- **Many of these attributes only apply to certain elements.** So do the SVG geometry properties, `object-fit`/`object-position`, `appearance`, `field-sizing` and `frame-sizing`, and nothing broke. None of these inherit.
- **Performance.** Every style recalc can now dirty the accessibility tree, and the tree depends on full style resolution for every element.
  It already does: `display`, `visibility`, `content`, `reading-flow` and `interactivity` are all style inputs to the tree, and engines already recompute it on style change. The new properties are not inherited and are excluded from `all`, so a change invalidates one node, not a subtree.
