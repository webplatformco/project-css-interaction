# Use Cases

Each entry shows what authors are instructed to do today to make a common UI pattern accessible, and what it collapses to with ARIA properties in CSS.

- [Manual ARIA boilerplate](#manual-aria-boilerplate)
	- [Skeleton](#skeleton)
	- [Loading buttons (`aria-busy`)](#loading-buttons-aria-busy)
	- [Scrollable regions](#scrollable-regions)
		- [Code blocks](#code-blocks)
		- [Responsive tables](#responsive-tables)
	- [Decorative elements (`aria-hidden="true"` / `role="none"`)](#decorative-elements-aria-hiddentrue--rolenone)
	- [Spinner (`role="progressbar"`)](#spinner-roleprogressbar)
	- [Avatar (`role="img"`)](#avatar-roleimg)
	- [Polyfills and workarounds](#polyfills-and-workarounds)
	- [Aligning `aria-orientation` with styling](#aligning-aria-orientation-with-styling)
	- [Breadcrumbs (`aria-current`)](#breadcrumbs-aria-current)
	- [Pagination (`aria-current`, `aria-disabled`)](#pagination-aria-current-aria-disabled)
	- [Stepper (`aria-current`, `aria-busy`)](#stepper-aria-current-aria-busy)
	- [Dialog labels (`aria-labelledby`)](#dialog-labels-aria-labelledby)
	- [Relationships that force templating (`aria-describedby`, `aria-errormessage`)](#relationships-that-force-templating-aria-describedby-aria-errormessage)
	- [Carousel slides (`aria-label` from position)](#carousel-slides-aria-label-from-position)
- [JS to sync ARIA state](#js-to-sync-aria-state)
	- [Scrollspy (`aria-current`)](#scrollspy-aria-current)
	- [Scroll-snap current item (`aria-current`)](#scroll-snap-current-item-aria-current)
	- [Form validation (`aria-invalid`)](#form-validation-aria-invalid)
	- [Disclosure triggers (`aria-expanded`)](#disclosure-triggers-aria-expanded)
	- [Menu items (`role="menuitemcheckbox"`, `aria-checked`, `aria-haspopup`)](#menu-items-rolemenuitemcheckbox-aria-checked-aria-haspopup)
	- [Tree view (`aria-level`, `aria-expanded`)](#tree-view-aria-level-aria-expanded)
	- [Toggle buttons (`aria-pressed`)](#toggle-buttons-aria-pressed)
		- [Checkbox styled as a button](#checkbox-styled-as-a-button)
		- [Password visibility toggle](#password-visibility-toggle)
	- [Tabs (`role="tab"`, `aria-selected`)](#tabs-roletab-aria-selected)
	- [Accordion to tabs by available space](#accordion-to-tabs-by-available-space)
	- [Select listboxes as other widgets (`role`, `aria-checked`)](#select-listboxes-as-other-widgets-role-aria-checked)
	- [Rating (`aria-readonly`)](#rating-aria-readonly)
	- [Combobox virtual focus (`aria-activedescendant`)](#combobox-virtual-focus-aria-activedescendant)
	- [Sortable tables (`aria-sort`)](#sortable-tables-aria-sort)
	- [Slider (`aria-valuetext`)](#slider-aria-valuetext)
	- [Values that draw the UI (`aria-valuenow`)](#values-that-draw-the-ui-aria-valuenow)
	- [Collapsed / inactive content (`aria-hidden`)](#collapsed--inactive-content-aria-hidden)
	- [Optional slots (`:has-slotted`)](#optional-slots-has-slotted)
	- [Shared relationship lists (`aria-labelledby`)](#shared-relationship-lists-aria-labelledby)
	- [Custom element semantics (`ElementInternals`)](#custom-element-semantics-elementinternals)
- [Pseudo-elements](#pseudo-elements)
	- [Customizable select with grid picker](#customizable-select-with-grid-picker)
	- [CSS carousels (`::scroll-button()`, `::scroll-marker`, `::column`)](#css-carousels-scroll-button-scroll-marker-column)
	- [Accordion panels and dropdown menus (`::details-content`)](#accordion-panels-and-dropdown-menus-details-content)
	- [Native tooltips (`::tooltip`)](#native-tooltips-tooltip)
	- [Slotted children (`::slotted()`)](#slotted-children-slotted)
	- [Repairing a component from outside (`::part()`)](#repairing-a-component-from-outside-part)
- [Markup you cannot edit](#markup-you-cannot-edit)
- [Built-in browser magic, made visible](#built-in-browser-magic-made-visible)


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
Even components forget: Web Awesome's [`<wa-skeleton>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/skeleton/skeleton.ts) sets no `aria-busy` at all.

### Loading buttons (`aria-busy`)

**Current:** Web Awesome's [`<wa-button>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/button/button.ts) renders a native `<button>` or `<a>` and binds three attributes to its own: `aria-busy` from `loading`, `aria-disabled` when a link button is `disabled`, and `role="button"` when it is not a link.
Web Awesome's native stylesheet already styles `button` and `a.wa-button`, so the class exists; the component is the ARIA.

```ts
role=${ifDefined(isLink ? undefined : 'button')}
aria-disabled=${ifDefined(isLink && this.disabled ? 'true' : undefined)}
aria-busy=${this.loading ? 'true' : 'false'}
```

With ARIA in CSS, the same class that draws the spinner and dims the link carries the state:

```css
.button.loading {
	aria-busy: true;
}

a.button.disabled {
	aria-disabled: true;
	pointer-events: none;
}
```

### Scrollable regions

#### Code blocks

**Current:** For [scrollable code blocks](https://github.com/GoogleChrome/guidance/blob/main/guides/html/html/guide.md), authors are instructed to:
> Ensure that scrollable code blocks are accessible by adding `role="region"` to the `<pre>`, along with an accessible name using either `aria-labelledby` (referencing the ID of the element that titles the block) or `aria-label`.

Note that this is very hard for content processed via Markdown, as code blocks often are.

#### Responsive tables

To [create responsive tables](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-atoms/responsive-table/guide.md) authors are instructed to manually add `role="region"` + `aria-labelledby` to the table wrapper.
Whether the element scrolls depends on content and viewport, so the static attribute is wrong whenever it doesn't.
Web Awesome's [`<wa-scroller>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/scroller/scroller.ts) is the component version of the same instructions: `role="region"` and a "Scrollable region" label unconditionally, and a `tabindex` that flips between `0` and `-1` from a `scrollHeight` measurement re-run on every scroll and resize.

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

Component libraries encode the same rule in a watcher.
Web Awesome's [`<wa-icon>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/icon/icon.ts) is `aria-hidden` unless it has a `label`, in which case it becomes `role="img"` with that label as its name; three `setAttribute`/`removeAttribute` pairs that re-run whenever `label` changes.
In CSS the attribute selector is the condition and `attr()` is the name:

```css
wa-icon:not([label]) {
	aria-hidden: true;
}

wa-icon[label] {
	role: img;
	aria-label: attr(label);
}
```

Sometimes the decorativeness is itself a CSS decision: a hero illustration that repeats the heading text only at narrow widths, a "matrix rain" of characters transformed into a backdrop, content layered behind a sheet.
`aria-hidden="true"` in markup is only right where the content is always decorative.
Today the only conditional path is `interactivity: inert`, which also blocks pointer interaction, text selection and find-in-page.
The Chrome team made the case for CSS owning the condition in the TAG review of that property ([design-reviews#1055](https://github.com/w3ctag/design-reviews/issues/1055#issuecomment-2762905148)), relaying what library and developer feedback kept coming back to:

> - Crawling a tree with JS to be picky about which nodes are inert or not? **CSS does that with selectors**
> - Adapting a tree crawl or UI to various screen sizes to manage whether or not items are offscreen inert or on screen and interactive? **CSS does that with media queries**
> - Knowledge of whether or not the inert item is actually off screen or just obscured in other UI layering creative design ways? **CSS is already orchestrating those moments** and knows best when to change it

The same reasoning applies to hiding non-interactive content, and the rule that makes the content decorative is the rule that hides it:

```css
@media (width < 600px) {
	.hero-illustration {
		aria-hidden: true;
	}
}
```

### Spinner (`role="progressbar"`)

The inverse of decorative elements: presentational markup that needs a role.

**Current:** Web Awesome's [`<wa-spinner>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/spinner/spinner.ts) is an SVG with `role="progressbar"` and a localized "Loading" label; the component exists to put those two attributes on it.

With ARIA in CSS, a spinner is a class:

```css
.spinner {
	role: progressbar;
	aria-label: "Loading";
}
```

### Avatar (`role="img"`)

**Current:** Web Awesome's [`<wa-avatar>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/avatar/avatar.ts) renders an image, initials or an icon, and gives whichever one it rendered `role="img"` and `aria-label` from its `label` attribute, in three separate template branches.

With ARIA in CSS, the same rule as the [icon](#decorative-elements-aria-hiddentrue--rolenone) above, on whatever the avatar holds:

```css
.avatar {
	role: img;
	aria-label: attr(label);
}
```

### Polyfills and workarounds

Authors often have to duplicate `role` and `aria-*` attributes to bridge browser support or work around incomplete/buggy implementations.

Examples:
- Adding `role=switch` to `<input type="checkbox" switch>` (the [switch polyfill](https://github.com/tomayac/input-switch-polyfill/blob/main/input-switch-polyfill.js#L29) does this from JS for every matching input)
- `<search role="search">`
- `<ul role="list">` to fix Safari removing list semantics when `list-style: none` (e.g. [swipe-to-remove lists](https://github.com/IanPouncey/modern-web-guidance-src/blob/swipe-to-remove-accessibility-update/guides/ui-behaviors/swipe-to-remove/guide.md))
- `aria-checked` synced by hand on a native checkbox input that carries `role="switch"` (Web Awesome's [`<wa-switch>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/switch/switch.ts)), because not every engine carries native checkedness over to the overridden role.
- `role="menu"` on the container and `role="menuitem"` on every button, written from JS only when `focusgroup` is unsupported ([menu](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/menu/guide.md)), because native `focusgroup` infers those roles and an explicit role blocks the inference; the fallback cannot be static markup.

The list case is particularly illuminating: the CSS creates the problem; but the fix must live in HTML or JS.

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

The `focusgroup` fallback is the one that needs a condition, and the condition is a feature test.
There is no `@supports` for an HTML attribute, but once `focusgroup` is a CSS property as well ([Tier 2](./README.md#tier-2-encouraging-more-patterns)), the roles it would have inferred come from the same place when it is missing:

```css
@supports not (focusgroup: menu) {
	.menu {
		role: menu;

		> button:not([role]) {
			role: menuitem;
		}
	}
}
```


### Aligning `aria-orientation` with styling

For [toolbars](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/toolbar/guides/ui-components/toolbar/guide.md), sliders, tablists, menus, treeviews, and many other components, it is CSS that changes their orientation, but ARIA must manually mirror it.

This also means that for cases where orientation varies e.g. by available space, this adds an unnecessary JS dependency.
Web Awesome mirrors its `orientation` attribute into `aria-orientation` in four components (divider, button group, radio group, slider), each with its own watcher, where the whole mapping is `[orientation="vertical"] { aria-orientation: vertical }`.

```css
[role="toolbar"].vertical {
	flex-direction: column;
	aria-orientation: vertical;

	[role="separator"] {
		aria-orientation: horizontal;
	}
}
```

The [menu](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/menu/guide.md) instructions have the same coupling one level up, on the role itself.
`role="menubar"` is correct only when the triggers are "styled as one continuous bar with no gaps between triggers"; visually separate buttons must drop the role and the arrow-key model, because "visually separate buttons signal one `Tab` stop each".
Whether a page header's triggers form a bar or stand apart can depend on available space, and the rule that lays them out already knows:

```css
@container (width > 40em) {
	.commands {
		display: flex;
		role: menubar;
	}
}
```



### [Breadcrumbs](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/breadcrumbs/guides/ui-components/breadcrumbs/guide.md) (`aria-current`)

**Current:** Authors are instructed to mark "the current page, the last item" with `aria-current="page"`,
although it's always the last item.
Web Awesome's [`<wa-breadcrumb>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/breadcrumb/breadcrumb.ts) automates exactly this: a `slotchange` handler loops over the items, sets `aria-current` on the last and removes it from the rest.

With ARIA in CSS:

```css
.breadcrumbs li:last-child > a {
	aria-current: page;
}
```

Step indicators are the same attribute from a class instead of a position.
The [multi-step form](https://github.com/GoogleChrome/guidance/blob/main/guides/forms/forms/guide.md) instructions ship `<li class="step-active" aria-current="step">`, the class for the styling and the attribute for AT, side by side on every step:

```css
.step-active {
	aria-current: step;
}
```

### Pagination (`aria-current`, `aria-disabled`)

**Current:** Web Awesome's [`<wa-pagination>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/pagination/pagination.ts) binds `aria-current="page"` on the current item and `aria-disabled` on the previous and next controls at either boundary, in four template branches (buttons or links, compact or full).
A static or server-rendered `<nav><ol>` carries the same attributes by hand, on every page.

```ts
aria-current=${ifDefined(isCurrent ? 'page' : undefined)}
aria-disabled=${isDisabled ? 'true' : 'false'}
```

With ARIA in CSS, the current page is the only state, and the boundary controls derive from it:

```css
.pagination .current > a {
	aria-current: page;
}

.pagination:has(.pages > li:first-child.current) .previous,
.pagination:has(.pages > li:last-child.current) .next {
	aria-disabled: true;
	pointer-events: none;
}
```

Generating the list from a total and a page size is templating either way; announcing a page change is one-shot and stays a JS API.

### Stepper (`aria-current`, `aria-busy`)

**Current:** The [step snippet](#breadcrumbs-aria-current) above is the simple case.
Web Awesome's [`<wa-step>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/step/step.ts) also has to decide *where* the attribute goes, because, as its source puts it, "`aria-current` belongs on whichever element actually receives focus: the inner `<button>` when the step is clickable, the host (a plain listitem) otherwise", and it adds `aria-busy` while a step is loading:

```ts
private syncAriaCurrent() {
	if (this.active && !this.clickable) {
		this.setAttribute('aria-current', 'step');
	} else {
		this.removeAttribute('aria-current');
	}
}
// plus aria-current=${this.active ? 'step' : nothing} on the <button> branch of the template,
// and a `loading` watcher for aria-busy
```

With ARIA in CSS, the placement is a selector:

```css
.stepper .active:not(:has(> button)) {
	aria-current: step;
}

.stepper .active > button {
	aria-current: step;
}

.stepper .loading {
	aria-busy: true;
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

Web Awesome's [`<wa-dialog>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dialog/dialog.ts) and drawer wire `aria-labelledby="title"` inside their shadow root; the fixed id is safe only because the component owns both ends of the relation, which is part of why a native `<dialog>` gets wrapped at all.

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

### Relationships that force templating (`aria-describedby`, `aria-errormessage`)

**Current:** Every relationship attribute takes an id, and ids must be unique on the page, so any repeated pattern with a relationship in it cannot be static markup.
A form field with a hint and an error message cannot be copy-pasted; it has to be generated, with the ids threaded through.
The pain is [acknowledged at the HTML level](https://github.com/whatwg/html/issues/10143), where the right fix for the attributes themselves remains an open question.
Meanwhile, frameworks ship a hook for exactly this.
React: "[`useId`](https://react.dev/reference/react/useId) is a React Hook for generating unique IDs that can be passed to accessibility attributes", because "a component may be rendered more than once on the page, but IDs have to be unique!"
Vue has the same [`useId()`](https://vuejs.org/api/composition-api-helpers.html#useid), "for accessibility attributes or form elements".
Component libraries solve it with a shadow root instead: twelve Web Awesome form controls (input, textarea, number input, select, checkbox, switch, slider, radio and checkbox groups, color picker, OTP and tag inputs) render a `<label for="input">`, a hint slot with a fixed id and `aria-describedby="hint"`, legal only because the ids are scoped to the component.
Web Awesome's native stylesheet already styles every one of those elements; the id plumbing is a large part of why they are wrapped.
The [error announcement](https://github.com/GoogleChrome/guidance/blob/main/guides/accessibility/accessible-error-announcement/guide.md) instructions show the hand-written version, one id per field and one per message:

```jsx
function Field({ label, hint, error, ...props }) {
	const id = useId();
	return (
		<div className="field">
			<label htmlFor={id}>{label}</label>
			<input id={id} aria-describedby={`${id}-hint`} aria-errormessage={`${id}-error`} {...props} />
			<p id={`${id}-hint`} className="hint">{hint}</p>
			<p id={`${id}-error`} className="error">{error}</p>
		</div>
	);
}
```

With tree-scoped references, the field is plain HTML again, with nothing unique in it:

```html
<div class="field">
	<label>Email <input type="email" required></label>
	<p class="hint">We never share it.</p>
	<p class="error">Please enter a valid email address.</p>
</div>
```

```css
.field {
	ref-scope: --hint, --error;

	> .hint {
		ref-name: --hint;
	}

	> .error {
		ref-name: --error;
	}

	input {
		aria-describedby: --hint;
		aria-errormessage: --error;
	}

	input:user-invalid {
		aria-invalid: true;
	}
}
```

The markup can be static, copy-pasted, hand-written or CMS-emitted, and the relationship holds for every field on the page and every one added later.
The same applies wherever this document uses a reference: a dialog and its heading, an accordion panel and its summary, a combobox and its active option.

### [Carousel slides](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/carousel/guides/ui-components/carousel/guide.md) (`aria-label` from position)

**Current:** Authors are instructed to give each slide `role="group"`, `aria-roledescription="slide"`, and "an `aria-label` that includes its position", typed out per slide, then announce "Slide N of M" from a status region after scrolling settles:

```html
<div class="carousel-slide" role="group" aria-roledescription="slide" aria-label="1 of 3">…</div>
<div class="carousel-slide" role="group" aria-roledescription="slide" aria-label="2 of 3">…</div>
<div class="carousel-slide" role="group" aria-roledescription="slide" aria-label="3 of 3">…</div>
```

Position and count are structural facts, and the labels go stale the moment a slide is added or removed.
With ARIA in CSS and the tree-counting functions, it is one rule that stays correct through any mutation:

```css
.carousel-slide {
	role: group;
	aria-roledescription: "slide";
	aria-label: sibling-index() " of " sibling-count();
}
```

The same shape covers every number ARIA derives from structure: `aria-posinset` and `aria-setsize` on list items, `aria-level` from nesting depth in a tree view built from nested `<details>`.
JS has to recompute all of them on every mutation; CSS never does.

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

The platform has since moved for one case: the ARIA WG [resolved](https://github.com/w3c/aria/issues/2710#issuecomment-3819696897) in January 2026 that a browser-determined active scroll marker maps to `aria-current="true"` with no attribute injection, and an author-set `aria-current` wins.
That is this proposal's processing model applied to one feature, and the general rule it generalizes to is:

```css
nav a:target-current {
	aria-current: true;
}
```

Chromium now implements the resolution behind a flag, with the value fixed to `true`.
ARIA also defines `page`, `step`, `location`, `date` and `time`; a table of contents is arguably `location`, and a multi-step flow is `step`.
The resolution lets an author-set `aria-current` win, but an attribute cannot follow `:target-current`, so the only way to pick the value is to go back to the script above.
With the property, choosing the value is the same rule:

```css
nav.toc a:target-current {
	aria-current: location;
}
```

Whether current AT distinguishes the values is a separate question; in the [2026-10-15 CSSWG discussion](https://github.com/w3c/csswg-drafts/issues/12269#issuecomment-6196438216), Matt King's read was that for most AT `page`, `true` and the others sound the same today, with support improving.

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
Web Awesome's [`<wa-carousel>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/carousel/carousel.ts) keeps three copies of "this slide is in view", a class, `inert` and `aria-hidden`, all written from one `IntersectionObserver` callback, plus `aria-busy` on the scroller while a scroll settles.
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
Web Awesome's select, tag input and known-date components bind `aria-invalid` from their own validity state in the template, the same bridge one level up.
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

### Menu items (`role="menuitemcheckbox"`, `aria-checked`, `aria-haspopup`)

**Current:** [Accordion panels and dropdown menus](#accordion-panels-and-dropdown-menus-details-content) below covers the container; the items are the same sync tax per entry.
The [menu](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/menu/guide.md) instructions make every checkable item a `<button>` with an explicit `role="menuitemcheckbox"` (or `menuitemradio`, inside a `role="group"`) and state that "script toggles `aria-checked`"; the check mark is then drawn from `[aria-checked="true"]::before`, so the attribute JS maintains is also the styling hook.
Every submenu trigger carries `aria-haspopup="menu"` and a JS-synced `aria-expanded` ([Disclosure triggers](#disclosure-triggers-aria-expanded)), and every menu is named with `aria-labelledby` pointing back at its trigger, which means a unique id on every trigger.
Web Awesome's [`<wa-dropdown-item>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dropdown-item/dropdown-item.ts) flips its `role` between `menuitem` and `menuitemcheckbox` from its `type`, syncs `aria-checked` from `checked`, and adds `aria-haspopup`/`aria-expanded` when a submenu is slotted ([Optional slots](#optional-slots-has-slotted)); [`<wa-dropdown>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dropdown/dropdown.ts) loops the items to write `aria-posinset`/`aria-setsize`, and writes `aria-haspopup` and `aria-expanded` into the consumer's slotted trigger button.

```ts
// dropdown-item
if (changedProperties.has('type')) {
	if (this.type === 'checkbox') {
		this.setAttribute('role', 'menuitemcheckbox');
		this.setAttribute('aria-checked', this.checked ? 'true' : 'false');
	} else {
		this.setAttribute('role', 'menuitem');
		this.removeAttribute('aria-checked');
	}
}

// dropdown
items.forEach((item, index) => {
	item.setAttribute('aria-posinset', String(index + 1));
	item.setAttribute('aria-setsize', String(items.length));
});
nativeButton.setAttribute('aria-haspopup', 'menu');
nativeButton.setAttribute('aria-expanded', this.open ? 'true' : 'false');
```

With ARIA in CSS, on the `<details>` menu from below: a checkable item is a checkbox input (`menuitemcheckbox` is already an allowed role on one), and a submenu is a nested `<details>`:

```css
.menu input[type="checkbox"] {
	role: menuitemcheckbox;
	aria-checked: false;

	&:checked {
		aria-checked: true;
	}
}

.menu > details > summary {
	role: menuitem;
	aria-haspopup: menu; /* aria-expanded follows [open] natively */
}
```

Nothing is written into the trigger, since `<summary>` already carries `aria-expanded`, and the position loop has no counterpart: ARIA has the UA compute `aria-posinset` and `aria-setsize` for sibling menu items.

On the popover-based layout from the menu instructions, the per-trigger ids go the same way as [Dialog labels](#dialog-labels-aria-labelledby).
The instructions already require each popover to be the next sibling of its trigger, and a singular reference resolves to the last in-scope name that precedes it in tree order (the anchor rule), so each menu names its own trigger:

```css
[role="menubar"],
[focusgroup~="menu"] {
	ref-scope: --trigger;
}

[aria-haspopup="menu"] {
	ref-name: --trigger;
}

[popover] > [focusgroup~="menu"] {
	aria-labelledby: --trigger;
}
```

Typeahead and submenu hover timing stay JS.

### Tree view (`aria-level`, `aria-expanded`)

**Current:** Web Awesome's [`<wa-tree>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tree/tree.ts) writes `role="tree"` and `aria-multiselectable` from its `selection` attribute; each [`<wa-tree-item>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tree-item/tree-item.ts) writes `role="treeitem"`, `aria-expanded` only while it has children, `aria-selected`, `aria-busy` while lazy-loading and `aria-disabled`, each from its own watcher, with the children in a `role="group"`.
Every one of those is the host element, which is the consumer's markup.

```ts
@watch('expanded', { waitUntilFirstUpdate: true })
handleExpandedChange() {
	if (!this.isLeaf) {
		this.setAttribute('aria-expanded', this.expanded ? 'true' : 'false');
	} else {
		this.removeAttribute('aria-expanded');
	}
}
```

With ARIA in CSS, the tree is nested `<details>`, and every state is already in the markup:

```css
.tree {
	role: tree;
	focusgroup: tree;
}

.tree[data-selection="multiple"] {
	aria-multiselectable: true;
}

.tree details {
	role: treeitem;
	counter-increment: level;
	aria-level: counter(level);
	aria-expanded: false;

	&[open] {
		aria-expanded: true;
	}

	&:not(:has(details)) {
		aria-expanded: none;
	}

	&.selected > summary {
		aria-selected: true;
	}
}
```

Two open questions: `treeitem` is not an allowed role on `<details>` or `<summary>` today, the same companion ARIA in HTML change [tabs](#tabs-roletab-aria-selected) need; and `aria-level` from nesting is a counter, which is a string, so the integer properties would need to accept one.
Multi-selection, indeterminate parents and lazy loading stay JS.

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
Web Awesome's [`<wa-tab>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tab/tab.ts) syncs `aria-selected` from `active`, `<wa-tab-panel>` syncs `aria-hidden` from the same, and `<wa-tab-group>` mints `wa-tab-N`/`wa-tab-panel-N` ids so it can write `aria-controls` and `aria-labelledby` per pair.
The CSS-only version of the widget is a group of radio inputs: `:checked` is the selected tab, arrow keys move between tabs natively, and only the checked one sits in the tab order.
What AT hears is a radio group.

The newer variant is a group of `<details name="…">` elements laid out as tabs, with `::details-content` as the panel.
Authors keep trying it and keep getting the same verdict:
- [Pure CSS tabs with details, grid and subgrid](https://css-tricks.com/pure-css-tabs-with-details-grid-and-subgrid/) (CSS-Tricks, 2025) drew the review that the result has no `tablist`/`tab` roles, no arrow-key or Home/End navigation, and announces no tab count or position: "a 'CSS-only' or 'pure CSS' widget won't have the same accessibility features as a built-for-purpose pattern."
- The Cirth CSS framework evaluated `<details>`, radio inputs and `:target`, then [deferred tabs indefinitely](https://github.com/cirthcss/cirth/issues/61) (2026): "CSS can select and present a state, but it cannot manage focus or update the corresponding ARIA attributes. Therefore, correct ARIA tabs currently require script."

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

The `<details>` variant is the same three rules, with the open state coming from `[open]` and the panel being a pseudo-element:

```css
.tabs {
	focusgroup: tablist;
}

.tabs > details[open] > summary {
	aria-selected: true;
}

.tabs > details::details-content {
	role: tabpanel;
	interactivity: focusable;
}
```

Neither `tab` on a radio input nor `tab` on a `summary` is currently a permitted role.
Both rely on the companion ARIA in HTML change the proposal asks for, the same loosening checkbox inputs received for `button` and `switch`.
Chrome already honors both overrides, but exposes neither a checked radio nor an open `details` as `aria-selected`, which is precisely the rule above.
The tab-to-panel relationship is per pair, which tree-scoped references do not express, so `aria-controls` stays an IDREF.

### Accordion to tabs by available space

**Current:** The same set of sections wants to be tabs where they fit in a row and an accordion where they do not.
The markup is identical either way; the semantics are not.
Tabs are a `tablist` of `tab`s with `aria-selected` and one focusable panel; an accordion is a set of disclosure buttons with `aria-expanded` and labelled regions.
Attributes cannot follow a breakpoint, so authors are told to pick one.
[Inclusive Components](https://inclusive-components.design/tabbed-interfaces/): "Some have noble attempts to reconfigure tabbed interfaces into accordion interfaces for small viewports. Given that accordions are structured, attributed, and operated completely differently to tabs, I would recommend against this."
Those who do it anyway ship [a script](https://www.makethingsaccessible.com/guides/responsive-and-accessible-tabbed-interfaces/) that starts from `<details>`/`<summary>`, listens to `matchMedia`, and rewrites every role and attribute on every switch, so that "there will be no incorrect roles present on either widget":

```js
const mq = window.matchMedia('(max-width: 767px)');
mq.addEventListener('change', () => {
	mq.matches ? createAccordions() : createTabs();
});
// createTabs(): role=tablist on the wrapper, role=tab + aria-selected + tabindex on every button,
// role=tabpanel + tabindex on every panel.
// createAccordions(): removeAttribute() for all of the above, then aria-expanded on every button
// and role=region on every panel. Plus bookkeeping to keep focus where it was across the switch.
```

With ARIA in CSS, the switch is a container query, and the narrow side needs no rules at all: a group of `<details name="…">` is already a correct exclusive accordion.
The wide side is the three rules from [Tabs](#tabs-roletab-aria-selected):

```css
main {
	container-type: inline-size;
}

@container (inline-size > 40em) {
	.sections {
		/* lay the summaries out in a row, the open panel below */
		focusgroup: tablist;
	}

	.sections > details[open] > summary {
		aria-selected: true;
	}

	.sections > details::details-content {
		role: tabpanel;
		interactivity: focusable;
	}
}
```

Nothing is removed on the way back: the rules stop matching and `auto` restores the disclosure semantics.
Focus needs no preserving, because no element is replaced.
[`aria-orientation`](#aligning-aria-orientation-with-styling) above is the one-property version of the same thing; a navigation list that becomes a disclosure below a breakpoint, or a toolbar whose overflow becomes a menu, are the same shape again.

### Select listboxes as other widgets (`role`, `aria-checked`)

A checkbox group, a rating, a cycle toggle and a tree view can all be implemented over a styled `<select>` listbox.
The appeal is the markup.
This is a checkbox group today:

```html
<fieldset>
	<legend>Toppings</legend>
	<label><input type="checkbox" name="toppings" value="Cheese"> Cheese</label>
	<label><input type="checkbox" name="toppings" value="Mushrooms"> Mushrooms</label>
	<label><input type="checkbox" name="toppings" value="Olives"> Olives</label>
</fieldset>
```

and this is the same control as a listbox:

```html
<select multiple name="toppings" class="checkboxes">
	<option>Cheese</option>
	<option>Mushrooms</option>
	<option>Olives</option>
</select>
```

`<select>` could be the **canonical multiple-choice widget on the web**.
Much higher signal-to-noise, much lower potential of error.
The former must be wrapped in a component to be made palatable, the latter can be used as-is.


What stands in the way is accessibility.
The keyboard side is already close: the customizable listbox moves focus with arrow keys and toggles with Space, which is the checkbox-group model, and the [older listbox model](https://github.com/openui/open-ui/issues/1484) is the radio-group one.
The semantics are not: whatever it looks like, AT hears a listbox of options with `aria-selected`, never a group of checkboxes with `aria-checked`.
ARIA in HTML permits no roles at all on `select[multiple]`, `select[size]` or `option`.
All three engines honor the overrides anyway on a customizable `<select>`, on the element and on its options.
(Safari keeps exposing the options of a *native* listbox as `option`, but a restyled widget opts into customizable select regardless, so that path is the one that matters.)
What no engine does is carry an option's selectedness over to the new role: `<option role="checkbox" selected>` is exposed as an unchecked checkbox.
So even the non-conforming route needs JS to copy selection into `aria-checked` on every change, the same sync tax as every entry in this section.

With ARIA in CSS, the relabel is a rule on the stylesheet that already does the restyling:

```css
select.checkboxes {
	role: group;

	> option {
		role: checkbox;
		aria-checked: false;

		&:checked {
			aria-checked: true;
		}
	}
}

select.segmented,
select.stars {
	role: radiogroup;

	> option {
		role: radio;
		aria-checked: false;

		&:checked {
			aria-checked: true;
		}
	}
}
```

Conformance-wise this relies on the companion ARIA in HTML loosening the proposal asks for, like tabs.
Implementation-wise the role half already works; the state half is exactly what these properties add.

### Rating (`aria-readonly`)

**Current:** Web Awesome's [`<wa-rating>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/rating/rating.ts) is `role="slider"` on the host, with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`, `aria-disabled`, `aria-readonly` and `aria-label` written in `connectedCallback` and rewritten by a watcher each, plus JS for keyboard, hover preview and half-star precision.

```ts
connectedCallback() {
	super.connectedCallback();
	this.setAttribute('aria-valuenow', String(this.value));
	this.setAttribute('aria-valuemin', '0');
	this.setAttribute('aria-valuemax', String(this.max));
	this.setAttribute('aria-disabled', this.disabled ? 'true' : 'false');
	this.setAttribute('aria-readonly', this.readonly ? 'true' : 'false');
	// …
}
```

The CSS-only widget is a radio group: `:checked ~ label` fills the stars, `label:has(~ :hover)` previews, half stars are more radios, arrow keys and "3 of 5" are native, and a `<fieldset>` names it.
It is announced *more* precisely than the slider, and needs none of the six attributes.
The one thing radios lack is a read-only state:

```css
.rating.readonly input {
	aria-readonly: true;
	pointer-events: none;
}
```

### Combobox virtual focus (`aria-activedescendant`)

**Current:** An autocomplete keeps real focus in the text input and moves a *virtual* focus through the suggestions.
The active suggestion is marked with a class so it can be styled; `aria-activedescendant` on the input is a copy of that class, updated on every arrow key, every filter change and every pointer hover, and it is the attribute most often found stale or pointing at a removed node.

```js
function setActive(option) {
	listbox.querySelector('.active')?.classList.remove('active');
	option.classList.add('active');
	if (!option.id) {
		option.id = `opt-${crypto.randomUUID()}`;
	}
	input.setAttribute('aria-activedescendant', option.id);
}
```

With ARIA in CSS and tree-scoped references, the class is the only state, and the id disappears:

```css
.combobox {
	ref-scope: --active;
}

.combobox .option.active {
	ref-name: --active;
}

.combobox input {
	role: combobox;
	aria-activedescendant: --active;
}
```

A singular reference resolves to the one in-scope element that carries the name, so when the class moves, the relationship moves with it, and when the active option is filtered out, the reference is simply empty.

### Sortable tables (`aria-sort`)

**Current:** The sort direction already lives in a class or data attribute, because the header's arrow is drawn from it.
`aria-sort` is a second copy, and both [Deque](https://dequeuniversity.com/library/aria/table-sortable) and [Adrian Roselli](https://adrianroselli.com/2021/04/sortable-table-columns.html) spell out the sync rule: remove it from the previously sorted header, set it on the new one, never leave two.

```js
function sortBy(header, direction) {
	for (const th of header.parentElement.children) {
		th.removeAttribute('aria-sort');
		delete th.dataset.sort;
	}
	header.dataset.sort = direction;
	header.setAttribute('aria-sort', direction);
	// … reorder the rows
}
```

With ARIA in CSS, the attribute that draws the arrow is the only state:

```css
th[data-sort="ascending"] {
	aria-sort: ascending;
}

th[data-sort="descending"] {
	aria-sort: descending;
}
```

As with the toolbar's `aria-pressed`, styling *from* `[aria-sort]` instead is the other way to keep the two in step, and it works as long as ARIA is the only copy.

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

### Values that draw the UI (`aria-valuenow`)

**Current:** A progress bar, a split panel and an image comparison each have exactly one number, and each writes it twice: once into the CSS that draws the control, once into `aria-valuenow`.
Web Awesome's [progress bar](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/progress-bar/progress-bar.ts) sets a `--percentage` custom property that its stylesheet uses as `width`, and the attribute next to it; the [split panel](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/split-panel/split-panel.ts) and [comparison](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/comparison/comparison.ts) do the same with `position`, into `grid-template-columns` and `clip-path` respectively.

```ts
this.style.setProperty('--percentage', `${clamp(this.value, 0, 100)}%`);
// …
html`<div role="progressbar" aria-valuemin="0" aria-valuemax="100" aria-valuenow=${this.value}>`
```

Inside a component the duplication is one line.
Outside one it is a wall: the HTML+CSS progress bar is `<div class="progress" style="--value: 40">`, and the style attribute is the *only* place the value exists, so making it accessible means adding `role` and `aria-valuenow="40"` by hand, on every instance, and keeping the two numbers equal forever.

With ARIA in CSS, the property that draws the bar is the value:

```css
.progress {
	role: progressbar;
	aria-valuemin: 0;
	aria-valuemax: 100;
	aria-valuenow: var(--value);

	&::before {
		width: calc(var(--value) * 1%);
	}
}
```

Where the value is a reflected attribute instead, `attr()` reads it: `wa-split-panel { aria-valuenow: attr(position type(<number>)) }`.
Web Awesome's [progress ring](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/progress-ring/progress-ring.ts) is the same number a third time, as a `--percentage` that drives a stroke offset; a `conic-gradient()` reading the same custom property is the whole drawing.

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

### Optional slots (`:has-slotted`)

**Current:** Whether a component exposes a label, a hint, a submenu or a disclosure depends on what the consumer slotted in, and components check that from JS.
Web Awesome's `HasSlotController` listens to `slotchange` in 24 components, and much of what it feeds is ARIA:
- Every form control (input, textarea, select, checkbox, switch, slider, number, time, OTP and tag inputs, radio and checkbox groups) renders its label and hint elements unconditionally and sets `aria-hidden` on each when its slot is empty.
- A [dropdown item](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dropdown-item/dropdown-item.ts) has `aria-haspopup="menu"` and `aria-expanded` only while a `submenu` is slotted.
- A [tree item](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tree-item/tree-item.ts) has `aria-expanded` only while it has child items.
- A dialog or drawer is `aria-labelledby` its title only while it has a header.

```ts
// dropdown-item
private handleSlotChange = () => {
	this.hasSubmenu = this.hasSlotController.test('submenu');

	if (this.hasSubmenu) {
		this.setAttribute('aria-haspopup', 'menu');
		this.setAttribute('aria-expanded', this.submenuOpen ? 'true' : 'false');
	} else {
		this.removeAttribute('aria-haspopup');
		this.removeAttribute('aria-expanded');
	}
};
```

The condition is structural, and CSS already selects it: `:has-slotted` from inside the shadow tree, `:has()` from outside.
Two components even grew SSR-only attributes (`with-label`, `with-header`) to work around, in their own words, "the lack of a `:has-slotted` CSS pseudo-class"; it has since shipped in every engine.
With ARIA in CSS the semantics go in the same rule that lays the slot out, and `none` expresses absence:

```css
/* form controls: the label element exists either way */
:host(:not([label])) .label:not(:has-slotted) {
	aria-hidden: true;
}

/* dropdown item: a submenu makes it a disclosure */
:host {
	role: menuitem;
}

:host(:has([slot="submenu"])) {
	aria-haspopup: menu;
	aria-expanded: false;
}

:host(:has([slot="submenu"]):state(submenu-open)) {
	aria-expanded: true;
}

/* tree item: leaves have no expanded state */
:host(:not(:has(wa-tree-item))) {
	aria-expanded: none;
}
```

### Shared relationship lists (`aria-labelledby`)

**Current:** A relationship attribute is a space-separated list, and more than one party may want to be in it.
Web Awesome's [tooltip](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tooltip/tooltip.ts) names the element its `for` attribute points at, so it has to add its own id to that element's `aria-labelledby` without clobbering whatever is already there, and remove exactly that id when the anchor changes or the tooltip disconnects:

```ts
private addToAriaLabelledBy(element: Element, id: string) {
	const currentLabel = element.getAttribute('aria-labelledby') || '';
	const labels = currentLabel.split(/\s+/).filter(Boolean);

	if (!labels.includes(id)) {
		labels.push(id);
		element.setAttribute('aria-labelledby', labels.join(' '));
	}
}
// plus the mirror image for removal, called from the `for` watcher and on disconnect
```

The anchor is the consumer's element, so this is also a component writing into markup it does not own.

With tree-scoped references, membership is computed rather than maintained.
A multi-valued property resolves to every in-scope element exposing the name, in tree order, so whatever names itself is in the list and whatever stops matching, or is removed, drops out:

```css
.field {
	ref-scope: --description;

	> wa-tooltip,
	> .hint {
		ref-name: --description;
	}

	> input {
		aria-describedby: --description;
	}
}
```

No add, no remove, no "only if not already present"; two sources naming the same element compose through the cascade's existing rules.
When the tooltip is not co-located with its anchor, `for` is an IDREF from the tooltip's side, and this remains a per-id rule, the same gap as in [Disclosure triggers](#disclosure-triggers-aria-expanded).

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

Web Awesome shows the double write at scale.
Five of its components (option, tree item, radio, dropdown item, step) publish the same fact twice in the same handler, once to CSS as a custom state and once to AT as an attribute:

```ts
// option
if (changedProperties.has('selected')) {
	this.setAttribute('aria-selected', this.selected ? 'true' : 'false');
	this.customStates.set('selected', this.selected);
}
```

The custom state exists so that CSS can select on it.
With ARIA in CSS that selector is where the semantics live (`:host(:state(selected)) { aria-selected: true }`) and the first line is gone.
The same holds for attributes the consumer sets, each of which is a watcher today and an attribute selector with these properties:

```css
wa-tree[selection="multiple"],
wa-tree[selection="leaf-multiple"] {
	aria-multiselectable: true;
}

wa-dropdown-item[type="checkbox"] {
	role: menuitemcheckbox;
}

wa-step[active]:not([clickable]) {
	aria-current: step;
}
```

## Pseudo-elements

Most pseudo-elements have no markup, so there is no attribute to set and no JS bridge to write.
Generated content can at least be named or hidden through `content: … / "alt"`; nothing equivalent exists for roles, states or relationships, on any pseudo-element.
Alt text is also a literal: string-management and translation tools do not see `content` values, which is why pointing at visible DOM text is the recommended practice ([#12269 (comment)](https://github.com/w3c/csswg-drafts/issues/12269#issuecomment-5054041459)), and a pseudo-element cannot point at anything.
Today these are simply blocked.
`::slotted()` and `::part()` are the other kind: they select real elements, but on the far side of a shadow boundary, where the stylesheet's author cannot set attributes either.

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

The pseudo-element is only half of it.
A `grid` needs `gridcell`s, and ARIA in HTML permits no role on `<option>`, so the options must be relabelled too, and they are elements.
An attribute cannot do it: the same picker is a list again below a breakpoint, or when the swatches are replaced by a text-only variant.
The option rule has to live where the layout rule lives, which is what a pseudo-element-only version of these properties ([#14534](https://github.com/w3c/csswg-drafts/issues/14534)) cannot express:

```css
select.swatches > option {
	role: gridcell;
}
```

A one-row picker is the lighter version of the same case.
`listbox` and `menu` both support `aria-orientation`, the layout that decides it is CSS, and the pseudo-element has nowhere to carry the attribute:

```css
select.swatches:open::picker(select) {
	display: flex;
	aria-orientation: horizontal;
}
```

Conformance-wise, `gridcell` on an `<option>` relies on the companion ARIA in HTML change the proposal asks for, like [tabs](#tabs-roletab-aria-selected) and [select listboxes](#select-listboxes-as-other-widgets-role-aria-checked).

### [CSS carousels](https://github.com/GoogleChrome/guidance/blob/guides/ui-components/carousel/guides/ui-components/carousel/guide.md) (`::scroll-button()`, `::scroll-marker`, `::column`)

**Current:** A CSS carousel is built one of two ways:
1. _Explicit pagination_: Slides are elements, as in [Carousel slides](#carousel-slides-aria-label-from-position) above, or
2. _Automatic pagination_: The scroller gets `columns: 1` and each page of items is a column box, reached through [`::column`](https://drafts.csswg.org/css-multicol-2/#column-pseudo).

Both get the same chrome via pseudo-elements:
- `::scroll-button()` for previous/next
- `::scroll-marker` inside `::scroll-marker-group` for the dots, generated per slide or per `::column`.

[CSS Overflow 5](https://drafts.csswg.org/css-overflow-5/) assigns their semantics:
- scroll buttons are focusable, activatable and `:disabled` at the end of the scroller
- the marker group is a `navigation` of `link`s, or a `tablist` of `tab`s with a focusgroup when `scroll-marker-group` says `tabs`.

The only semantic the author controls is the accessible name, through `content: … / "alt"`:

```css
.carousel-track::scroll-button(inline-start) {
	content: "‹" / "Previous slide";
}

.carousel-slide::scroll-marker {
	content: "" / attr(data-marker-name);
}
```


The `links` / `tabs` keyword on `scroll-marker-group` already picks these roles from CSS (see [CSS and the a11y tree](./README.md#css-and-the-a11y-tree)).

What is still missing:

- `::scroll-marker-group` is a landmark or tablist with no name.
	- The guide labels the fallback `<nav>` "Choose a slide". The native group it replaces cannot be labelled.
	- Two carousels on a page are two indistinguishable `navigation` landmarks.
- Marker names are "Slide N of M" by structure, but alt text only accepts strings (including `attr()`).
	- With explicit pagination, the author writes a `data-marker-name` on every slide (boilerplate).
	- With `::column`, there is no element for `attr()` to read.
- With `::column`, the slide itself is gone too, so `role="group"`, `aria-roledescription="slide"` and "N of M" have no element to sit on.


With ARIA in CSS, the names and roles live next to the rules that generate the UI, and the current marker is already covered by the ARIA WG's `:target-current` [resolution](https://github.com/w3c/aria/issues/2710#issuecomment-3819696897):

```css
.carousel-track {
	scroll-marker-group: after;

	&::scroll-marker-group {
		aria-label: "Slides";
	}

	&::scroll-button(inline-start) {
		content: "‹" / "Previous slide";
	}
}

.carousel-slide {
	role: group;
	aria-roledescription: "slide";
	aria-label: sibling-index() " of " sibling-count();

	&::scroll-marker {
		content: "";
		aria-label: "Slide " sibling-index();
	}
}
```

The `::column` shape needs the same rules on `::column` and `::column::scroll-marker`.

### Accordion panels and dropdown menus (`::details-content`)

**Current:** A `<details>`-based accordion renders its panel in `::details-content`, which has no element to carry `role="region"` or a label.
Authors who want the panel exposed as a labelled region must wrap the content in an extra `<div role="region" aria-labelledby="…">`, with a unique id on every summary.
Web Awesome's [`<wa-details>`](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/details/details.ts) and accordion item render exactly that wrapper inside their shadow root, plus `aria-controls` from the summary, around a native `<details>` whose own panel they then cannot use.

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

The same pseudo-element is the popup when `<details>` is used as a dropdown menu, as GitHub does throughout its UI.
`<summary>` already allows `aria-haspopup` in ARIA in HTML, and the UA already keeps its `aria-expanded` in sync with `[open]`.
What is blocked is the container: the `menu` role and the arrow-key model belong on the popup, and the element that would carry them is the wrapper `::details-content` exists to remove.
With `focusgroup` alongside the ARIA properties, the behavior token supplies the `menu` role and infers `menuitem` on the buttons, as `tablist` does for [tabs](#tabs-roletab-aria-selected):

```html
<details class="menu">
	<summary>Options</summary>
	<button>Rename</button>
	<button>Delete</button>
</details>
```

```css
.menu > summary {
	aria-haspopup: menu;
}

.menu::details-content {
	position: absolute;
	focusgroup: menu;
}
```

### Native tooltips (`::tooltip`)

**Current:** The `title` attribute is the web's built-in tooltip, and it is widely treated as unusable: hover only, after a delay, never on keyboard focus outside Edge, never on touch, and allowed on elements that cannot be focused at all.
The CSSWG has [resolved to add `::tooltip`](https://github.com/w3c/csswg-drafts/issues/8930) to style it, and the accessibility objection in that thread was that the attribute's problems should be fixed before it is decorated.
Its accessibility-tree side already works: `title` is exposed as the element's description, or its name when nothing else provides one.
What is missing is a way to reach the tooltip without a mouse.

With `interactivity` alongside `::tooltip`, the fix is two rules:

```css
/* reach tooltips on elements that are not already focusable */
[title]:not(:any-link, button, input, select, textarea) {
	interactivity: focusable;
}

[title]:is(:hover, :focus-visible)::tooltip {
	appearance: base;
}
```

The tooltip itself needs a role, and it has no element to carry one.
With ARIA properties, the UA stylesheet can declare it, the same way it declares every other UA default:

```css
::tooltip {
	role: tooltip;
}
```

That rule is one instance of a [larger pattern](#built-in-browser-magic-made-visible): UA-assigned semantics expressed as rules authors can read and override.

### Slotted children (`::slotted()`)

**Current:** A component can give *itself* a role and state through `ElementInternals` ([above](#custom-element-semantics-elementinternals)), and nothing else.
The children a consumer slots in are the consumer's markup, so a `<tab-group>` that wants each child exposed as a `tab` has two options, both bad.
It can write attributes into markup it does not own, from a `slotchange` handler, and keep them in sync forever.
Or it can require every child to be a component too, whose only job is to carry the role: Shoelace's `<sl-tab>` does exactly this, with `this.setAttribute('role', 'tab')` in `connectedCallback` and `this.setAttribute('aria-selected', …)` whenever `active` changes, on an element whose entire visible content is a `<slot>`.

```js
// sl-tab: a custom element whose job is to hold three attributes
connectedCallback() {
	super.connectedCallback();
	this.setAttribute('role', 'tab');
}
handleActiveChange() {
	this.setAttribute('aria-selected', this.active ? 'true' : 'false');
}
handleDisabledChange() {
	this.setAttribute('aria-disabled', this.disabled ? 'true' : 'false');
}
```

Its successor Web Awesome still does this in nine components (tree, tree item, radio, option, dropdown item, carousel, carousel item, divider, icon), each with `setAttribute('role', …)` in `connectedCallback` and a watcher per state.

With ARIA in CSS, the component's stylesheet already reaches slotted children to lay them out, so it can describe them in the same place.
The children can be plain `<button slot="tab">`s, and in the flat tree they sit directly under the host, so `tablist` → `tab` holds:

```css
:host {
	role: tablist;
}

::slotted([slot="tab"]) {
	role: tab;
	aria-selected: false;
}

::slotted([slot="tab"][selected]) {
	aria-selected: true;
}
```

Nothing is written into the consumer's DOM, and a child component exists only when it has behavior of its own.

### Repairing a component from outside (`::part()`)

**Current:** ARIA relationships are IDREFs, and IDREFs do not cross shadow boundaries, so a `<label>` wrapping a custom input does not label the `<input>` inside it.
Every fix requires the component's cooperation: making the element form-associated so the host itself is labelable, forwarding the name through element reflection, or adopting [Reference Target](https://github.com/WICG/webcomponents/blob/gh-pages/proposals/reference-target-explainer.md).
Where the component has done none of these, the consumer's options are what [Nolan Lawson catalogued](https://nolanlawson.com/2022/11/28/shadow-dom-and-accessibility-the-trouble-with-aria/): copy the label's text into an `aria-label` the component hopefully forwards, "set up a `MutationObserver` or similar technique to observe whenever the `<label>` changes", or stop using the component.
None of them work for `aria-describedby`, `aria-controls` or any relationship that is not a name.

With `::part()` and tree-scoped references, a selector reaches both ends of the relation, so the consumer repairs it from their own stylesheet:

```css
label:has(> fancy-input) {
	ref-name: --label;
	ref-scope: --label;
}

label > fancy-input::part(input) {
	aria-labelledby: --label;
}
```

The same path fixes a vendor component's internals when its exposed parts carry the wrong semantics (`fancy-nav::part(items) { role: list }`), which is [Markup you cannot edit](#markup-you-cannot-edit) across a shadow boundary.
It only reaches what the component exposes as a part, which is the right limit: parts are the component's declared styling surface, and this makes them its declared semantic surface too.

## Markup you cannot edit

CSS is often the only injection point into markup an author does not control: Markdown output, CMS templates, third-party embeds, components.
[Dave Rupert's case](https://daverupert.com/2022/10/use-case-for-cascading-attribute-sheets/) is a Markdown-powered blog where foreign-language snippets need `lang` and `dir`, without which the text "makes it inaccessible".
Adding them means breaking out of Markdown or threading conditionals through every template: "One is easy to write and the other is correct."
`lang` and `dir` are outside this proposal, but the same shape recurs with ARIA, and there CSS can also *repair* semantics.
A third-party navigation widget marks a plain list of links up as `role="menu"`/`role="menuitem"`, a misuse the [dropdown instructions](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-atoms/viewport-aware-dropdowns/guide.md) warn against because it promises a keyboard model the widget does not implement.
The embedding page cannot touch the markup, but it can restyle it.

```css
.vendor-nav [role="menu"] {
	role: list;
}

.vendor-nav [role="menuitem"] {
	role: listitem;
}
```

Across a shadow boundary, the same repair goes through [`::part()`](#repairing-a-component-from-outside-part).

## Built-in browser magic, made visible

Browsers compute a lot of accessibility semantics that no attribute expresses, and all of it is invisible.
[HTML-AAM](https://w3c.github.io/html-aam/) assigns implicit roles and states, engines add heuristics on top (WebKit dropping list semantics from unmarked lists), and browsers quietly correct known misuse (`aria-hidden` ignored on `html` and `body`, undone when focus lands inside the hidden subtree).
An author who wants to know why an element has a role reads a spec or a bug tracker; an author who disagrees with a heuristic has no override short of an attribute in markup.

Presentation has had the better deal for two decades.
HTML's [Rendering section](https://html.spec.whatwg.org/multipage/rendering.html) specifies presentational behavior as UA stylesheet rules (`[hidden] { display: none }`, `dialog:modal { … }`), inspectors show every one of them next to the author's own, and authors override them through the cascade.
With ARIA properties, the semantic half can be specified, shipped and inspected the same way:

```css
/* HTML-AAM implicit roles and states */
search {
	role: search;
}

input[type="checkbox"][switch] {
	role: switch;
}

details[open] > summary {
	aria-expanded: true;
}

dialog:modal {
	aria-modal: true;
}

/* CSS-generated UI with UA-assigned semantics */
::tooltip {
	role: tooltip;
}

/* the ARIA WG resolution for active scroll markers */
::scroll-marker:target-current {
	aria-current: true;
}

/* the CSSWG resolution for customizable select glyphs */
::checkmark,
::picker-icon {
	aria-hidden: true;
}

/* misuse corrections: !important marks what authors cannot undo */
html, body {
	aria-hidden: false !important;
}

/* engine heuristics, roughly */
:is(ul, ol):not(nav *) {
	role: if(style(list-style-type: none): none; else: auto);
}
```

Each of these replaces something that exists today in a worse form.
The `::tooltip` rule is the one [Native tooltips](#native-tooltips-tooltip) needs; the `aria-hidden` rule is the mitigation the ["`aria-hidden` of CSS" objection](./README.md#misuse) asks for, and [#13040](https://github.com/w3c/csswg-drafts/issues/13040) floated the same move for focusability (`[tabindex="0"] { interactivity: focusable !important }`).
The last rule is the [Safari list bug](https://bugs.webkit.org/show_bug.cgi?id=170179) from [Polyfills and workarounds](#polyfills-and-workarounds), written down: as a UA-origin rule it loses to the author's `ul { role: list }`, which is what the markup `role="list"` workaround achieves today, and the difference between engines becomes a diff between two stylesheets rather than a blog post.

Implicit relations are the least visible of all.
Per [HTML-AAM](https://www.w3.org/TR/html-aam-1.0/#att-command-popovers), a `commandfor` trigger gets an implicit `aria-details` to its popover unless the popover is its next sibling, and the [menu](https://github.com/GoogleChrome/guidance/blob/main/guides/ui-components/menu/guide.md) instructions bend the whole document structure around that: every popover must directly follow its trigger, and the final checklist verifies there is no `aria-details` on any trigger.
Nothing in the markup names the relation, so nothing in the markup can decline it.
As a UA rule it shows up in the inspector like any other, and an author who wants the placement freedom overrides it: `[commandfor] { aria-details: none }`.

The `::checkmark` rule is a decision the CSSWG is making right now, without this mechanism, and struggling with.
[#14316](https://github.com/w3c/csswg-drafts/issues/14316) resolved that `::checkmark` and `::picker-icon` are out of the accessibility tree, since the state they draw is already exposed on the control.
APA objected that this "actually *removes* information"; WebKit, that a hard-coded exclusion means "an author or user cannot override" it, and that `::checkmark` is already exposed on macOS.
The compromise on the table is hidden by default, exposed when the author provides explicit alt text, which needs a special `auto` behavior for `content` on these two pseudo-elements, and the issue remains open.
As a UA rule the compromise is the ordinary cascade: hidden by default, so overriding `content` leaks nothing into names or values, and an author or user stylesheet that wants the glyph announced says so:

```css
option::checkmark {
	content: "✓" / "selected";
	aria-hidden: false;
}
```

What this buys:
- **Inspectable.** DevTools already shows UA rules with provenance; a CSS-originated role is as traceable as a CSS-originated `display`, which is the [guardrail](./README.md#guardrails) the proposal asks implementations for.
- **Overridable where it should be, locked where it should not.** Author overrides go through the normal cascade (`::picker(select) { role: grid }`); corrections browsers must not let authors undo are `!important` in the UA origin, the same way `[hidden]` cannot be un-hidden by a lower-specificity rule.
- **Specifiable.** HTML-AAM and [CSS-AAM](https://github.com/w3c/css-aam) rows become rules, and every mitigation added to an attribute applies to the property for free, because both are inputs to the same computation.

The UA's own semantics are the one piece of markup nobody can edit.
