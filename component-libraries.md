Which components of popular libraries would be a CSS class on a built-in element if CSS could set ARIA (and focusability)?
Each component links to its source; rows link to the matching [use case](./use-cases.md) where one exists.

# Web Awesome

[Web Awesome](https://github.com/shoelace-style/webawesome) (branch `next`, October 2026) has 73 components.
It also ships [native styles](https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/styles/native.css) for `button`, `details`, `dialog`, `hr`, checkboxes, radios, text fields, `select`, `input[type=range]`, `input[type=color]` and `progress`, so for many components the styling half of the class already exists in the same package; the component exists for the other half.

| | Count |
|-|-|
| A class on a native element today, no ARIA needed | 10 |
| A class once CSS can set ARIA | 32 |
| A class once CSS can set ARIA and `focusgroup` | 7 |
| Stays a component | 24 |

Columns: the native element the class would go on, what CSS already covers, what ARIA in CSS adds (the blocker), and what JS would remain, if any.

## A class today

Nothing stands in the way; the component wraps presentation.

| Component | Element | Notes |
|-|-|-|
| [badge][] | `span` | |
| [callout][] | `div` | |
| [card][] | `article`/`div` | header, media and footer are slots → children |
| [tag][] | `span` + `button` | the remove button is markup; the `wa-remove` event is the author's listener either way |
| [button][] | `button` / `a` | native styles already cover both; `aria-busy` from `[loading]` is the one ARIA write ([Loading buttons](./use-cases.md#loading-buttons-aria-busy)) |
| [button-group][] | `div` | the JS toggles hover/focus classes on neighbours for border collapsing, which is `:has()` today |
| [popup][] | any | anchor positioning covers placement, flip, shift, arrows, `anchor-size()`; virtual elements stay JS |
| [animated-image][] | `img` | `image-animation: controlled` is the CSS version, with UA-assigned semantics |
| [scroller][] | `div` | `overflow` + scroll-driven animations for the shadows; focused scroll containers already handle Home/End. The `role="region"` is the [scrollable regions](./use-cases.md#scrollable-regions) case |
| [textarea][] | `textarea` | native styles exist; `resize` is CSS; autosize is `field-sizing: content`; the character count is a small `output` + JS |

## A class once CSS can set ARIA

| Component | Element | CSS already covers | ARIA in CSS adds | Residue |
|-|-|-|-|-|
| [skeleton][] | `div` | everything | `aria-busy: true` ([Skeleton](./use-cases.md#skeleton)) | |
| [spinner][] | `div` | everything | `role: progressbar; aria-label: "Loading"` ([Spinner](./use-cases.md#spinner-roleprogressbar)) | |
| [avatar][] | `span`/`img` | shape, initials, icon | `role: img; aria-label: attr(label)` ([Avatar](./use-cases.md#avatar-roleimg)) | |
| [divider][] | `hr` | native styles, orientation, label via `::before` | `aria-orientation` from `[orientation]` ([orientation](./use-cases.md#aligning-aria-orientation-with-styling)); a labelled divider needs `aria-label` | |
| [icon][] | `span` | `mask-image` | `aria-hidden` unless `[label]`, then `role: img; aria-label: attr(label)` ([decorative elements](./use-cases.md#decorative-elements-aria-hiddentrue--rolenone)) | fetching SVG by name stays JS; a mask-based icon set does not |
| [checkbox][] | `input[type=checkbox]` | native styles | `aria-describedby: --hint` via `ref-scope` ([relationships](./use-cases.md#relationships-that-force-templating-aria-describedby-aria-errormessage)); `:user-invalid { aria-invalid }` ([form validation](./use-cases.md#form-validation-aria-invalid)) | `setCustomValidity()` is native |
| [switch][] | `input[type=checkbox switch]` | native styles | `role: switch` where the UA does not ([polyfills](./use-cases.md#polyfills-and-workarounds)); `aria-describedby` | |
| [radio][] | `input[type=radio]` | native styles | `aria-describedby` | |
| [radio-group][] | `fieldset` | native styles | `aria-orientation` from `[orientation]`; `aria-describedby` | |
| [checkbox-group][] | `fieldset` | native styles | same | |
| [input][] | `input` | native styles, decorations via `::before`/`::after` or siblings | `aria-describedby: --hint`; `:user-invalid`; the [password toggle](./use-cases.md#password-visibility-toggle)'s `aria-pressed` | clearing and type flipping are one line of JS each |
| [number-input][] | `input[type=number]` | native styles | `aria-describedby`; `:user-invalid` | custom stepper icons |
| [select][] (single) | `select` | customizable select, `::picker()` | `aria-describedby`; `:user-invalid`; `::picker()` roles where the picker is a [grid](./use-cases.md#customizable-select-with-grid-picker) | multiple-with-tags and lazy loading stay JS |
| [option][] | `option` | customizable select | nothing, unless relabelled ([select listboxes](./use-cases.md#select-listboxes-as-other-widgets-role-aria-checked)) | |
| [details][] | `details` | native styles, `::details-content` transitions, icon placement | `::details-content { role: region; aria-labelledby: --summary }` ([accordion panels](./use-cases.md#accordion-panels-and-dropdown-menus-details-content)) | |
| [accordion][], [accordion-item][] | `details[name]` | exclusive accordion, heading wrapper is markup | same as details | expand/collapse all is the author's loop either way |
| [dialog][] | `dialog` | native styles, `closedby` for light dismiss, `commandfor` to open | `aria-labelledby: --title` via `ref-scope` ([dialog labels](./use-cases.md#dialog-labels-aria-labelledby)); `dialog:modal { aria-modal: true }` is UA | `autofocus` covers initial focus |
| [drawer][] | `dialog` | dialog + placement, transitions | same as dialog | |
| [popover][] | `[popover]` | `popover` + anchor positioning + `commandfor`; the UA sets `aria-expanded` on the invoker | nothing on the popover itself; roles when it holds a widget | |
| [tooltip][] | `::tooltip` / `[popover]` | interest invokers + anchor positioning | `role: tooltip` (UA for [`::tooltip`](./use-cases.md#native-tooltips-tooltip)); the [shared description list](./use-cases.md#shared-relationship-lists-aria-labelledby) | manual show/hide is `showPopover()` |
| [progress-bar][] | `progress` or `div` | native styles; `width: var(--value)` | `aria-valuenow: var(--value)` ([values that draw the UI](./use-cases.md#values-that-draw-the-ui-aria-valuenow)) | |
| [progress-ring][] | `progress` or `div` | `conic-gradient` from `attr(value)` / `var(--value)` | same | |
| [breadcrumb][], [breadcrumb-item][] | `nav > ol > li > a` | separators via `::before`, dropdown items are markup | `li:last-child > a { aria-current: page }` ([breadcrumbs](./use-cases.md#breadcrumbs-aria-current)) | |
| [step][], [stepper][] | `nav > ol > li` | variants, orientation, icons | `.active { aria-current: step }` on host or button; `.loading { aria-busy: true }` ([stepper](./use-cases.md#stepper-aria-current-aria-busy)) | step-change announcements (one-shot, `ariaNotify()`) |
| [pagination][] | `nav > ol > li > a` | compact, summary via `sibling-index()`/`sibling-count()` | `aria-current: page`; `aria-disabled` on boundary controls ([pagination](./use-cases.md#pagination-aria-current-aria-disabled)) | generating the list from `total`/`page-size` is templating or the server |
| [rating][] | `fieldset > input[type=radio]` | the classic CSS-only widget: `:checked`, `:hover` and `:has(~ :hover)` for preview, extra radios for precision | `aria-readonly` for read-only; radios are already a correctly announced value ([rating](./use-cases.md#rating-aria-readonly)) | `wa-hover` event |
| [slider][] (single) | `input[type=range]` | native styles, `datalist` markers | `aria-valuetext: control-value() "%"` ([slider](./use-cases.md#slider-aria-valuetext)) | the value tooltip needs `control-value()` in `content`; range (two thumbs) stays JS |
| [carousel][], [carousel-item][] | `div` with scroll snap | `::scroll-marker`, `::scroll-button()`, `scroll-snap`, slides per page via `scroll-snap-type` and sizing ([CSS carousels](./use-cases.md#css-carousels-scroll-button-scroll-marker-column)) | `aria-label: sibling-index() " of " sibling-count()` ([carousel slides](./use-cases.md#carousel-slides-aria-label-from-position)); `:snapped { aria-current }`; `aria-hidden`/`interactivity: inert` on off-screen slides ([scroll-snap](./use-cases.md#scroll-snap-current-item-aria-current)) | loop, autoplay and mouse dragging stay JS |

## A class once CSS can set ARIA and `focusgroup`

These need roving focus or arrow-key traversal as well, which is the other half of the program.

| Component | Element | CSS already covers | ARIA + `focusgroup` adds | Residue |
|-|-|-|-|-|
| [tab][], [tab-group][], [tab-panel][] | `details[name]` or radio group | layout, overflow scrolling (`::scroll-button()`) | `focusgroup: tablist`; `aria-selected` from `[open]`/`:checked`; `role: tabpanel; interactivity: focusable` ([tabs](./use-cases.md#tabs-roletab-aria-selected)) | closable tabs are a button plus the author's handler; manual activation is what `details` does natively |
| [dropdown][], [dropdown-item][] | `details` + `::details-content` | anchor positioning, nested `details` for submenus, `:checked` for checkable items | `focusgroup: menu` on the panel ([dropdown menus](./use-cases.md#accordion-panels-and-dropdown-menus-details-content)); `menuitemcheckbox` and `aria-checked` on checkbox inputs, `aria-haspopup` on submenu summaries ([menu items](./use-cases.md#menu-items-rolemenuitemcheckbox-aria-checked-aria-haspopup)) | typeahead, submenu hover timing |
| [tree][], [tree-item][] | nested `details` | indent guides, icons, `[open]` | `role: tree`/`treeitem`; `aria-level` from nesting; `aria-expanded` from `[open]`, `none` on leaves; `aria-selected`; `focusgroup` ([tree view](./use-cases.md#tree-view-aria-level-aria-expanded)) | multi-selection, indeterminate parents, lazy loading |

## Stays a component

Behavior that is neither presentation nor semantics.

| Component | Why |
|-|-|
| [tag-input][], [otp-input][], [time-input][], [known-date][] | composite editing: tokenisation, segment focus, parsing, locale rules |
| [color-picker][] | the panel: 2D picking, format conversion, eyedropper. A plain swatch is `input[type=color]`, which native styles already cover |
| [split-panel][] | CSS `resize` has no keyboard path and resizes the element rather than a divider |
| [comparison][] | the range value drives a *sibling's* `clip-path`; one line of JS, no CSS path |
| [copy-button][] | clipboard |
| [qr-code][] | canvas |
| [format-bytes][], [format-date][], [format-number][], [relative-time][] | `Intl`, timers |
| [include][], [markdown][], [random-content][] | fetching and parsing |
| [animation][], [intersection-observer][], [mutation-observer][], [resize-observer][] | declarative wrappers over JS APIs |
| [toast][], [toast-item][] | one-shot announcements and timers |
| [zoomable-frame][] | `zoom` on an `iframe` is CSS; the controls and theme sync are JS |
| [page][] | a layout; not ARIA-blocked, and mostly CSS already |

## What this says

- **The blocker is semantics, not styling.** For 39 components the styling is already CSS, and for the form controls, `details`, `dialog`, `progress` and `hr` among them, Web Awesome's own native stylesheet already styles the element the class would go on. What the component adds is `role`, `aria-*` and (for seven) focus management.
- **Form controls are the biggest group.** Ten wrappers around native inputs exist mainly to attach a label and a hint, to sync `aria-invalid`, and to style the result; `ref-scope` and `:user-invalid` cover the first two and native styles the third.
- **Native behaviour is usually already there.** `details`, `dialog`, `popover`, scroll snap, customizable select and radio groups supply the interaction; the components re-implement it in JS so they can own the attributes. Rating is the clearest case: the radio version is *more* accessible than the slider the component exposes.
- **The residue is small and one-shot.** Where JS remains in the "class" tiers it is clipboard, timers, announcements, or a single line that flips an attribute, none of it the state-sync code that ARIA in CSS removes.

[accordion]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/accordion/accordion.ts
[accordion-item]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/accordion-item/accordion-item.ts
[animated-image]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/animated-image/animated-image.ts
[animation]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/animation/animation.ts
[avatar]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/avatar/avatar.ts
[badge]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/badge/badge.ts
[breadcrumb]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/breadcrumb/breadcrumb.ts
[breadcrumb-item]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/breadcrumb-item/breadcrumb-item.ts
[button]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/button/button.ts
[button-group]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/button-group/button-group.ts
[callout]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/callout/callout.ts
[card]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/card/card.ts
[carousel]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/carousel/carousel.ts
[carousel-item]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/carousel-item/carousel-item.ts
[checkbox]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/checkbox/checkbox.ts
[checkbox-group]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/checkbox-group/checkbox-group.ts
[color-picker]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/color-picker/color-picker.ts
[comparison]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/comparison/comparison.ts
[copy-button]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/copy-button/copy-button.ts
[details]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/details/details.ts
[dialog]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dialog/dialog.ts
[divider]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/divider/divider.ts
[drawer]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/drawer/drawer.ts
[dropdown]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dropdown/dropdown.ts
[dropdown-item]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/dropdown-item/dropdown-item.ts
[format-bytes]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/format-bytes/format-bytes.ts
[format-date]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/format-date/format-date.ts
[format-number]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/format-number/format-number.ts
[icon]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/icon/icon.ts
[include]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/include/include.ts
[input]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/input/input.ts
[intersection-observer]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/intersection-observer/intersection-observer.ts
[known-date]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/known-date/known-date.ts
[markdown]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/markdown/markdown.ts
[mutation-observer]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/mutation-observer/mutation-observer.ts
[number-input]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/number-input/number-input.ts
[option]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/option/option.ts
[otp-input]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/otp-input/otp-input.ts
[page]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/page/page.ts
[pagination]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/pagination/pagination.ts
[popover]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/popover/popover.ts
[popup]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/popup/popup.ts
[progress-bar]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/progress-bar/progress-bar.ts
[progress-ring]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/progress-ring/progress-ring.ts
[qr-code]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/qr-code/qr-code.ts
[radio]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/radio/radio.ts
[radio-group]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/radio-group/radio-group.ts
[random-content]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/random-content/random-content.ts
[rating]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/rating/rating.ts
[relative-time]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/relative-time/relative-time.ts
[resize-observer]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/resize-observer/resize-observer.ts
[scroller]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/scroller/scroller.ts
[select]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/select/select.ts
[skeleton]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/skeleton/skeleton.ts
[slider]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/slider/slider.ts
[spinner]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/spinner/spinner.ts
[split-panel]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/split-panel/split-panel.ts
[step]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/step/step.ts
[stepper]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/stepper/stepper.ts
[switch]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/switch/switch.ts
[tab]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tab/tab.ts
[tab-group]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tab-group/tab-group.ts
[tab-panel]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tab-panel/tab-panel.ts
[tag]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tag/tag.ts
[tag-input]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tag-input/tag-input.ts
[textarea]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/textarea/textarea.ts
[time-input]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/time-input/time-input.ts
[toast]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/toast/toast.ts
[toast-item]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/toast-item/toast-item.ts
[tooltip]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tooltip/tooltip.ts
[tree]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tree/tree.ts
[tree-item]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/tree-item/tree-item.ts
[zoomable-frame]: https://github.com/shoelace-style/webawesome/blob/next/packages/webawesome/src/components/zoomable-frame/zoomable-frame.ts
