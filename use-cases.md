# Use Cases

Each entry shows what authors are instructed to do today to make a common UI pattern accessible, and what it collapses to with ARIA properties in CSS.
Links point to [current implementation guidance](https://github.com/GoogleChrome/guidance) for each pattern, as evidence of present practice; JS snippets are taken from it, trimmed to the ARIA-syncing parts.

## Manual ARIA boilerplate

Static attributes authors must remember to add by hand, on every instance, even though the fact they express is already a CSS fact.

### [Skeleton](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/skeleton/guide.md)

**Current:** Must remember to add `aria-busy="true"` to every `class="skeleton"`.

```css
.skeleton {
	aria-busy: true;
}
```

Newer [skeleton instructions](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/skeleton/guides/ui-components/skeleton/guide.md) key all the CSS on `.skeleton[aria-busy="true"]` and tell authors to "toggle `aria-busy`, `inert` and `aria-hidden` together" when loading finishes, so ARIA is the source of truth and CSS reads it.
The same `aria-busy` toggle recurs for [slow view loads](https://github.com/GoogleChrome/guidance/blob/2106-guided-navigation/guides/user-experience/guided-navigation/guide.md).

### Scrollable regions

#### Code blocks

**Current:** For [scrollable code blocks](https://github.com/GoogleChrome/guidance/blob/main/guides/html/html/guide.md), authors are instructed to:
> Ensure that scrollable code blocks are accessible by adding `role="region"` to the `<pre>`, along with an accessible name using either `aria-labelledby` (referencing the ID of the element that titles the block) or `aria-label`.

Note that this is very hard for content processed via Markdown, as code blocks often are.

#### Responsive tables

To [create responsive tables](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-atoms/responsive-table/guide.md) authors are instructed to manually add `role="region"` + `aria-labelledby` to the table wrapper.
Whether the element scrolls depends on content and viewport, so the static attribute is wrong whenever it doesn't.

With ARIA in CSS, this is just a scrollable container query + `aria-label`:

```css
pre:has(> code) {
	role: if(scroll-state(scrollable: none): region; else: revert-rule);
	aria-label: if(scroll-state(scrollable: none): "Code block"; else: revert-rule);
}
```

### Decorative elements (`aria-hidden="true"` / `role="none"`)

**Current:** Authors are instructed to add it by hand in around eight different patterns, and routinely forget:
1. Blanket rule that every `<svg>` needs `aria-hidden` or a name (which also sweeps in zero-size SVGs holding `<defs>`, as a hedge against inconsistent exposure of unnamed inline SVG).
1. A muted background `<video>` (`role="none"`/`aria-hidden`),

More often than not, the decorativeness is already keyed to a CSS state, for example:

```css
.icon:not([aria-label], [role]) {
	aria-hidden: true;
}

svg:not([aria-label], [role]) {
	aria-hidden: true;
}

video[muted][autoplay]:not([controls]) {
	role: none;
}
```

### Polyfills and workarounds

Authors often have to duplicate `role` and `aria-*` attributes to bridge browser support or work around incomplete/buggy implementations.

Examples:
- Adding `role=switch` to `<input type="checkbox" switch>` (the [switch polyfill](https://github.com/tomayac/input-switch-polyfill/blob/main/input-switch-polyfill.js#L29) does this from JS for every matching input)
- `<search role="search">`
- `<ul role="list">` to fix Safari removing list semantics when `list-style: none` (e.g. [swipe-to-remove lists](https://github.com/IanPouncey/modern-web-guidance-src/blob/swipe-to-remove-accessibility-update/guides/ui-behaviors/swipe-to-remove/guide.md))

The latter is particularly illuminating: the CSS creates the problem; but the fix must live in HTML or JS.

With ARIA in CSS, this duplication can be avoided:
```css
.SwipeableList {
	list-style: none;
	role: list; /* Restore list semantics for Safari */
}

input[type="checkbox"][switch] {
	role: switch;
}

search {
	role: search;
}
```


### Aligning `aria-orientation` with styling

For [toolbars](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/toolbar/guides/ui-components/toolbar/guide.md), sliders, tablists, menus, treeviews, and many other components, it is CSS that changes their orientation, but ARIA must manually mirror it.

This also means that for cases where orientation varies e.g. by available space, this adds an unnecessary JS dependency.

```css
[role="toolbar"].vertical {
	flex-direction: column;
	aria-orientation: vertical;

	[role="separator"] {
		aria-orientation: horizontal;
	}
}
```



### [Breadcrumbs](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/breadcrumbs/guides/ui-components/breadcrumbs/guide.md) (`aria-current`)

**Current:** Authors are instructed to mark "the current page, the last item" with `aria-current="page"`,
although it's always the last item.

With ARIA in CSS:

```css
.breadcrumbs li:last-child > a {
	aria-current: page;
}
```

### Dialog labels (`aria-labelledby`)

**Current:** Every dialog must be named by its heading: [modal and non-modal dialogs](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/dialog/guides/ui-components/dialog/guide.md), [light-dismiss dialogs](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/light-dismiss-a-dialog/guide.md), and [app tour steps](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/persistent-app-tours/guide.md) (`role="dialog"` plus `aria-labelledby`).
That means a unique id on each heading and a matching reference on each container, by hand, per instance:

```html
<dialog id="confirm" aria-labelledby="confirm-title">
	<h2 id="confirm-title">Delete this item?</h2>
	…
</dialog>
```

With tree-scoped references, the rule is written once and every dialog labels itself by its own heading, no ids involved:

```css
dialog {
	ref-scope: --title;
	aria-labelledby: --title;
}

dialog > h2 {
	ref-name: --title;
}
```

## JS to sync ARIA state

Use cases where JS exists solely to sync ARIA attributes with CSS state.

### [Scrollspy](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/scrollspy/guide.md) (`aria-current`)

**Current:** Scrollspy currently needs manual sync of `aria-current`:

```js
const syncAriaCurrent = () => {
	const currentLink = document.querySelector('nav a:target-current');
	document.querySelectorAll('nav a').forEach(link => {
		link.setAttribute('aria-current', link === currentLink ? 'true' : 'false');
	});
};
// Update the aria on initial page load. Ensure the DOM content is loaded, either by using `<script type="module">` or an event listener for `DOMContentLoaded`.
syncAriaCurrent();
// Update the aria after scroll ends
document.addEventListener('scrollend', syncAriaCurrent);
```

With ARIA in CSS:

```css
nav a:target-current {
	aria-current: true;
}
```

### Scroll-snap current item (`aria-current`)

**Current:** To sync [gallery thumbnails](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/scroll-snap-realtime-feedback/guide.md) or a [table of contents](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/scroll-snap-state-sync/guide.md) to a scroll-snapping container, authors are instructed to listen to `scrollsnapchange`, map the snap target back to an index, clear `aria-current` on every thumbnail or link, and set it on one.
Plus a one-off geometric check on load, because no event fires for restored scroll positions.
An [anchor-positioned tab underline](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/anchor-positioning-tab-underline/guide.md) carries the same "visual only, you must also set `aria-current`/`aria-selected`" caveat.

```js
container.addEventListener('scrollsnapchange', (event) => {
	const index = [...items].indexOf(event.snapTargetInline);
	if (index === -1 || !thumbnails[index]) return;

	thumbnails.forEach((thumb) => {
		thumb.classList.remove('pending', 'active');
		thumb.removeAttribute('aria-current');
	});
	thumbnails[index].classList.add('active');
	thumbnails[index].setAttribute('aria-current', 'true');
});

// Neither scrollsnapchange nor scroll fires for a restored scroll position,
// so authors are also given a ~30 line geometric closest-item check to run on load.
```

In a [carousel](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/carousel/guides/ui-components/carousel/guide.md), native `::scroll-marker:target-current` covers the visual, but the fallback `<button>` markers and a `data-current` attribute on slides are synced from a settled-scroll handler, same as above.
For a [navbar](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/navbar/guides/ui-components/navbar/guide.md), `aria-current="page"` is "set at build time or with JavaScript".

With ARIA in CSS:

```css
.gallery-item:snapped {
	aria-current: true;
}
```

Covers initial state for free.
Same relational gap as in disclosure triggers below when `aria-current` belongs on a *different* element (thumbnail, TOC link) than the snapped one; `:has()` only helps if they are in the same subtree.

### Form validation (`aria-invalid`)

**Current:** The instructions for [announcing errors accessibly](https://github.com/GoogleChrome/guidance/blob/main/guides/accessibility/accessible-error-announcement/guide.md) exist *solely* to teach a JS bridge from `:user-invalid` to `aria-invalid`.
The instructions for [required field feedback](https://github.com/GoogleChrome/guidance/blob/main/guides/forms/required-field-feedback/guide.md) repeat the same bridge, and two more patterns point to it.
Visual and programmatic error state can drift.

```js
const updateAriaState = (event) => {
	const input = event.target;
	if (!input.matches?.('input, textarea, select')) return;

	if (input.matches(':user-invalid')) {
		input.setAttribute('aria-invalid', 'true');
	} else {
		input.removeAttribute('aria-invalid');
	}
};

// 'blur' and 'focus' do not bubble, so we must use the capture phase (true).
document.addEventListener('blur', updateAriaState, true);
document.addEventListener('focus', updateAriaState, true);

// Also update on input if we've already shown the error,
// so the error clears immediately when fixed.
document.addEventListener('input', (event) => {
	if (event.target.getAttribute?.('aria-invalid') === 'true') {
		updateAriaState(event);
	}
});
```

Two later corrections ([one](https://github.com/rajhodedara/modern-web-guidance-src/blob/fix/1639-required-field-aria-sync/guides/forms/required-field-feedback/guide.md), [two](https://github.com/rajhodedara/modern-web-guidance-src/blob/fix/1638-invalid-submit-sync/guides/accessibility/accessible-error-announcement/guide.md)) show the bridge accreting edge cases:
- Gate on `:user-invalid`, not raw `checkValidity()`: tabbing through an untouched required field must NOT set `aria-invalid`.
- Sync *every* invalid field on blocked submission, not just the one the browser focuses:

```js
// Constraint validation fires 'invalid' on every control that fails when
// the form is submitted, so synchronize all of them, not only the first
// field the browser focuses.
document.addEventListener('invalid', updateAriaState, true);
```

Both are bugs a one-line CSS rule cannot have:

```css
:user-invalid {
	aria-invalid: true;
}
```

Guaranteed same-moment sync, and the entire set of instructions collapses to one rule.
`aria-errormessage` / `aria-describedby` stay static HTML (IDREFs), which is fine.

### Disclosure triggers (`aria-expanded`)

**Current:** The most common manual sync across all patterns surveyed.
Browser only sets `aria-expanded` for `popovertarget` / `commandfor`; every other trigger is JS:

- [Navigation drawer](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/navigation-drawer/guide.md) (a scroll-snap driven `popover="manual"`): an `IntersectionObserver` callback sets `aria-expanded` when the sheet enters or leaves the viewport.
- [Menu](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/menu/guide.md): a `toggle` listener on each popover syncs the trigger's `aria-expanded`.
- [Search hidden content](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/search-hidden-content/guide.md) (`hidden="until-found"` accordions): a `beforematch` handler loops over buttons to set `aria-expanded` from `aria-controls`.
- [Animated collapse](https://github.com/GoogleChrome/guidance/blob/main/guides/css/animate-to-intrinsic-sizes/guide.md) and general accessibility instructions: click handlers toggle `aria-expanded` by hand.
- [Declarative dialog/popover control](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-behaviors/declarative-dialog-popover-control/guide.md): the polyfill doesn't set `aria-expanded`, authors are told to do it themselves.

```js
// navigation-drawer: open/closed is decided by an IntersectionObserver on the sheet
function onDrawerOpened() {
	document.querySelector('main').inert = true;
	openBtn.setAttribute('aria-expanded', 'true');
	sheet.focus();
}

function onDrawerClosed() {
	drawer.hidePopover();
	document.querySelector('main').inert = false;
	openBtn.setAttribute('aria-expanded', 'false');
}

const observer = new IntersectionObserver((entries) => {
	const entry = entries.at(-1);
	if (entry.intersectionRatio < visibleThreshold) onDrawerClosed();
	// …
});

// menu, toolbar: popover toggle event
menu.addEventListener('toggle', ({ newState }) => {
	trigger.setAttribute('aria-expanded', String(newState === 'open'));
});

// search-hidden-content: beforematch reveals a panel, buttons follow
accordion.addEventListener('beforematch', (e) => {
	accordion.querySelectorAll('button').forEach((btn) => {
		const controls = btn.getAttribute('aria-controls');
		btn.setAttribute('aria-expanded', controls === e.target.id ? 'true' : 'false');
	});
});

// animated collapse: click handler
triggerBtn.addEventListener('click', () => {
	const isExpanded = triggerBtn.getAttribute('aria-expanded') === 'true';
	triggerBtn.setAttribute('aria-expanded', !isExpanded);
});
```

More of the same:
- A [toolbar](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/toolbar/guides/ui-components/toolbar/guide.md) syncs its overflow menu's `aria-expanded` from the popover `toggle` event.
- [tabs](https://github.com/GoogleChrome/guidance/blob/knowler/tabs/guides/ui-components/tabs/guide.md) sync `aria-selected` in JS, and again in `beforematch` when panels use `hidden="until-found"`.
- [searchable hidden content](https://github.com/joelamyman/modern-web-guidance-src/blob/search-hidden-content-accessibility-updates/guides/ui-behaviors/search-hidden-content/guide.md) adds `aria-disabled` next to `aria-expanded` in `beforematch`.

With ARIA in CSS:

```css
/* menu: submenu popover is the next sibling of its trigger */
[aria-haspopup="menu"]:has(+ [popover]:popover-open) {
	aria-expanded: true;
}

/* search-hidden-content: panel follows its button */
.custom-accordion:has(#panel-2:not([hidden])) #btn-2 {
	aria-expanded: true;
}

/* navigation-drawer: the drawer popover precedes <main>, which holds the trigger */
#drawer:popover-open ~ main #drawer-open {
	aria-expanded: true;
}
```

**Gap:** every one of these is relational.
`:has()` works for siblings/descendants, but the generic case is "the element my `aria-controls` / `commandfor` points to is open".
The [reference combinator](https://drafts.csswg.org/selectors-5/#idref-combinators) in Selectors 5 covers it, through `:has()`:

```css
[commandfor]:has(/commandfor/ :popover-open),
[aria-controls]:has(/aria-controls/ :not([hidden])) {
	aria-expanded: true;
}
```

Until it ships, authors still need per-ID rules or JS.

### Toggle buttons (`aria-pressed`)

A [toolbar](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/toolbar/guides/ui-components/toolbar/guide.md) is instructed to use the AT state as the styling hook: style *from* `[aria-pressed="true"]` so visual and AT state cannot diverge.
That only works when ARIA is the sole copy of the state.
It breaks as soon as the state already lives in the DOM, which is the common case: `:checked` or `[type]` is the state, and `aria-pressed` is a second copy that JS keeps in sync.

#### Checkbox styled as a button

**Current:** The canonical CSS-only widget: `:checked` is the pressed state, Space toggles it, no script needed.
[ARIA in HTML](https://www.w3.org/TR/html-aria/#el-input-checkbox) permits `role="button"` on a checkbox input "if used with `aria-pressed`", but the browser does not derive `aria-pressed` from checkedness once the role is overridden.
So a widget that needs no JS for anything else needs JS for this alone:

```js
toggle.addEventListener('change', () => {
	toggle.setAttribute('aria-pressed', toggle.checked);
});
```

With ARIA in CSS:

```css
.toggle {
	role: button;
	aria-pressed: false;
}

.toggle:checked {
	aria-pressed: true;
}
```

#### Password visibility toggle

**Current:** A [password visibility toggle](https://github.com/GoogleChrome/guidance/blob/main/guides/forms/forms/guide.md) flips `input[type]` between `password` and `text`.
`aria-pressed` on the button is a second copy of state that already has a selector:

```html
<div class="password-toggle">
	<input id="password" type="password">
	<button type="button" commandfor="password" command="--toggle-password"
	        data-show-label="Show password" data-hide-label="Hide password"></button>
</div>
```

```js
// command does not bubble, so delegate in the capture phase
document.addEventListener('command', e => {
	if (e.command !== '--toggle-password') return;
	const show = e.target.type === 'password';
	e.target.type = show ? 'text' : 'password';
	e.source.setAttribute('aria-pressed', show);
	e.source.setAttribute('aria-label', show ? 'Hide password' : 'Show password');
}, { capture: true });
```

With ARIA in CSS, the JS only flips the type:

```js
document.addEventListener('command', e => {
	if (e.command === '--toggle-password') {
		e.target.type = e.target.type === 'password' ? 'text' : 'password';
	}
}, { capture: true });
```

```css
.password-toggle button {
	aria-pressed: false;
	aria-label: attr(data-show-label);
}

.password-toggle:has(input[type="text"]) button {
	aria-pressed: true;
	aria-label: attr(data-hide-label);
}
```

### Tabs (`role="tab"`, `aria-selected`)

**Current:** For [tabbed interfaces](https://github.com/GoogleChrome/guidance/blob/knowler/tabs/guides/ui-components/tabs/guide.md), authors are instructed to use `focusgroup=tablist` for roving focus and `tab` semantics, then add `aria-selected` on the selected tab, `role=tabpanel` plus a name on each panel, `aria-controls` from each tab to its panel, and `hidden` on inactive panels.
`aria-selected` is synced in JS, and again in a `beforematch` handler when panels use `hidden="until-found"`.
The CSS-only version of the widget is a group of radio inputs: `:checked` is the selected tab, arrow keys move between tabs natively, and only the checked one sits in the tab order.
What AT hears is a radio group.

With ARIA in CSS, and `focusgroup` and `interactivity` alongside it:

```css
.tabs {
	focusgroup: tablist; /* roving focus, tablist and tab semantics */
}

.tabs > input[type="radio"]:checked {
	aria-selected: true;
}

.tabs > .panel {
	role: tabpanel;
	interactivity: focusable; /* a panel is a tab stop */
}
```

`tab` is not currently a permitted role on radio inputs.
This relies on the companion ARIA in HTML change the proposal asks for, the same loosening checkbox inputs received for `button` and `switch`.
The tab-to-panel relationship is per pair, which tree-scoped references do not express, so `aria-controls` stays an IDREF.

### [Slider](https://github.com/GoogleChrome/guidance/blob/main/guides/forms/slider/guide.md) (`aria-valuetext`)

**Current:** If the slider value requires a formatted label or unit, authors must update `aria-valuetext` when changed. The minimal bridge is:

```js
const syncValueText = () => {
	slider.setAttribute('aria-valuetext', `${slider.value}%`);
};
syncValueText(); // the listener alone leaves the initial value unlabeled
slider.addEventListener('input', syncValueText);
```

With ARIA in CSS:

```css
input[type=range].percentage {
	aria-valuetext: control-value() "%";
}
```

The code is about the same size either way; the difference is when it is true.
Until the script has loaded and run, AT reads a bare number.
Setting `slider.value` programmatically fires no event, so the label silently goes stale.
The CSS version has no initialization step and no listener: it holds from the moment the stylesheet applies, for every slider that matches, however the value changes.

### Collapsed / inactive content (`aria-hidden`)

**Current:**

- [SPA view switching](https://github.com/GoogleChrome/guidance/blob/main/guides/performance/faster-spa-view-transitions/guide.md): `switchToView()` loops all views setting `aria-hidden="true"`, then `"false"` on the active one.
  (Arguably redundant since `content-visibility: hidden` already drops contents from the AT tree, but authors are required to do it.)
- [General performance advice](https://github.com/GoogleChrome/guidance/blob/main/guides/performance/performance/guide.md): "`content-visibility: auto` keeps elements in the DOM for screen readers… manage `aria-hidden` manually."

```js
function switchToView(viewId) {
	document.querySelectorAll('.spa-view').forEach(view => {
		view.classList.add('inactive');
		view.setAttribute('aria-hidden', 'true');
	});

	const activeView = document.getElementById(viewId);
	activeView.classList.remove('inactive');
	activeView.setAttribute('aria-hidden', 'false');
	activeView.focus();
}
```

With ARIA in CSS:

```css
.spa-view.inactive {
	content-visibility: hidden;
	aria-hidden: true;
}
```

### Custom element semantics (`ElementInternals`)

**Current:** Component authors are instructed to set role and ARIA state through [`ElementInternals`](https://github.com/GoogleChrome/guidance/blob/main/guides/html/accessible-web-components/guide.md), so the semantics are defaults the component owns rather than attributes a consumer can clobber:

```js
class ToggleSwitch extends HTMLElement {
	#internals;
	constructor() {
		super();
		this.#internals = this.attachInternals();
		this.#internals.role = 'switch';
		this.#internals.ariaChecked = 'false';
		this.addEventListener('click', () => this.toggle());
	}
	toggle() {
		const on = this.#internals.ariaChecked !== 'true';
		this.#internals.ariaChecked = String(on);
	}
}
```

Here ARIA *is* the state: the component has no other record of being on.
The moment it does, a `checked` attribute so consumers can set it, or a custom state so CSS can style it, the mixin becomes one more copy to update imperatively in every code path that changes the state.

With ARIA in CSS, the component's own stylesheet declares the semantics once, from whichever source of truth it already exposes:

```css
:host {
	role: switch;
	aria-checked: false;
}

:host([checked]),
:host(:state(checked)) {
	aria-checked: true;
}
```

This is also more composable than the mixin.
Custom attributes, custom states, and consumer overrides all go through the same cascade, and a consumer can extend the semantics (`toggle-switch[readonly] { aria-readonly: true }`) without the component having anticipated it.

## Inability to apply ARIA to pseudo-elements

Pseudo-elements have no markup, so there is no attribute to set and no JS bridge to write.
Generated content can at least be named or hidden through `content: … / "alt"`; nothing equivalent exists for roles, states or relationships, on any pseudo-element.
Today these are simply blocked.

### [Customizable select with grid picker](https://github.com/GoogleChrome/guidance/blob/main/guides/forms/custom-select-picker-layouts/guide.md)

**Current:** Grid layouts for `<select>` pickers (color swatches, emoji, product variants) can be styled, but not described:

> The `<select>` and its `::picker()` still have their default roles (`menu` or `listbox`), and these cannot be overridden, nor are they allowed to contain an element with `role="grid"`.
> Nesting a wrapper with `role="grid"` inside the picker works in practice, but is not valid, and may surprise AT users.
> […] it may be ultimately necessary to switch to a custom solution using the popover API, anchor positioning, and `role="grid"` to achieve the desired UX for screen reader users.

Authors are blocked on exactly the thing CSS can see and HTML cannot reach: `role` and `focusgroup` on `::picker(select)`.

```css
select.swatches::picker(select) {
	display: grid;
	grid-template-columns: repeat(6, 1fr);
	role: grid;
	focusgroup: grid;
}
```

### Accordion panels (`::details-content`)

**Current:** A `<details>`-based accordion renders its panel in `::details-content`, which has no element to carry `role="region"` or a label.
Authors who want the panel exposed as a labelled region must wrap the content in an extra `<div role="region" aria-labelledby="…">`, with a unique id on every summary.

With ARIA in CSS and tree-scoped references, each panel labels itself by its own summary:

```css
details {
	ref-scope: --summary;

	> summary {
		ref-name: --summary;
	}

	&::details-content {
		role: region;
		aria-labelledby: --summary;
	}
}
```

## Markup you cannot edit

CSS is often the only injection point into markup an author does not control: Markdown output, CMS templates, third-party embeds, components.
[Dave Rupert's case](https://daverupert.com/2022/10/use-case-for-cascading-attribute-sheets/) is a Markdown-powered blog where foreign-language snippets need `lang` and `dir`, without which the text "makes it inaccessible".
Adding them means breaking out of Markdown or threading conditionals through every template: "One is easy to write and the other is correct."
`lang` and `dir` are outside this proposal, but the same shape recurs with ARIA, and there CSS can also *repair* semantics.
A third-party navigation widget marks a plain list of links up as `role="menu"`/`role="menuitem"`, a misuse the [dropdown instructions](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-atoms/viewport-aware-dropdowns/guide.md) warn against because it promises a keyboard model the widget does not implement.
The embedding page cannot touch the markup, but it can restyle it.
The same applies across a shadow boundary.
A component author can make a wrapping `<label>` reach the inner control, by making the element form-associated and forwarding the name through element reflection, or by adopting Reference Target.
When they have not, the consumer has no recourse today; with `::part()` the relationship can be repaired from the outside.

```css
.vendor-nav [role="menu"] {
	role: list;
}

.vendor-nav [role="menuitem"] {
	role: listitem;
}
```
