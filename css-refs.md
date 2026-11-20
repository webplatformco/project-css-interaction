# Element references in CSS: `ref-name` and `ref-scope`

**TL;DR:** A general way to name an element from CSS (`ref-name`), bound the visibility of that name (`ref-scope`), and point at it from any feature that needs to reference another element (via a bare `<dashed-ident>` or a `ref(<dashed-ident>)`) . A generalization of `anchor-name` / `anchor-scope`.

## Problem

CSS features increasingly need to point at *another element*. Today each one picks its own mechanism:

| Mechanism | Used by | Problems |
|---|---|---|
| ID selector (`#foo`, `<id>`) | `element()`,  the defunct `nav-*` properties | Document-unique, so it breaks on the second instance of a component. Requires markup access. Cannot be set from CSS (and should not be). |
| Fragment URL (`url(#foo)`) | `clip-path`, `mask`, `filter`, `fill`, `stroke`, `marker-*`, `offset-path`, `target-counter()`, `target-text()` | All of the above, plus base URL confusion. Needed a special case to become tree-scoped ([#3320](https://github.com/w3c/csswg-drafts/issues/3320)). |
| Feature-specific name + scope pair | `anchor-name` + `anchor-scope`, `scroll-timeline-name` / `view-timeline-name` + `timeline-scope`, `timeline-trigger-name` / `event-trigger-name` + `trigger-scope` | Works well, but every new feature mints another pair, and the same element ends up named once per feature ([#7046](https://github.com/w3c/csswg-drafts/issues/7046)). |
| Feature-specific name, no scope | `container-name`, `view-transition-name`, GCPM `position: running()`, the defunct `flow-into` and `flow-from` | Same, without reuse across instances. |
| IDREF in a string | `aria-*` relationship properties as proposed in [#14534](https://github.com/w3c/csswg-drafts/issues/14534) | All the [ID problems](https://github.com/whatwg/html/issues/10143). |

The WG has already concluded that the name + scope pairs should behave identically ([#12581](https://github.com/w3c/csswg-drafts/issues/12581), [#13364](https://github.com/w3c/csswg-drafts/issues/13364)). The next step is to stop minting them: if the lookup is the same, the name can be too, and even when it isn't, the constraint can live at the resolution point.

Besides improving existing consumers, a generic reference primitive opens up new possibilities, such as:
- CSS-declared invoker relationships, e.g. making a pseudo-element an interest invoker for an HTML tooltip ([#14763](https://github.com/w3c/csswg-drafts/issues/14763)). Pseudo-elements cannot carry `interestfor`, so the target has to be named from CSS.
- Reading values from another element (e.g. an `inherit()` or style query that targets a ref). Speculative, and cycle-prone, but only expressible at all with a reference primitive.
- CSS-based [portals](https://react.dev/reference/react-dom/createPortal) where an element is moved to another place in the DOM tree

## Proposal

### `ref-name` and `ref-scope`

Two properties, both initial `none`, not inherited, not animatable, apply to all elements and pseudo-elements:
- `ref-name` exposes the element under one or more names.
- `ref-scope` confines the listed names (or `all`) to the element's subtree, in both directions.

Syntax:
```
ref-name:  none | <dashed-ident>#
ref-scope: none | all | <dashed-ident>#
```

Lookup:
1. Walk up the ancestor chain to the scoping element
2. Resolve name within that scope:
	- **Consumers that take a single element:** take the last match in tree order (same as #13364)
	- **Consumers that take a list** (no present consumers, CSS regions are defunct, and #14534 is a proposal) take *all* in-scope matches, in tree order.

Names are tree-scoped. The reference itself carries no layout or rendering requirement. Each consuming feature decides what a valid referent is (e.g. an acceptable anchor, a scroll container, a paint server) and what happens when it is not.

### The `ref()` function

Where there is no ambiguity, a bare `<dashed-ident>` will be interpreted as a reference (e.g. as the argument to `element()`). When disambiguation is required, the `ref()` function provides it:

```
<element> = ref( <dashed-ident> )
```

- Resolves to the element found by the `ref-name` lookup for that name, as above.
- Computed value is as specified.
- Resolution happens at used-value time, since the referent can change without the declaration changing.
- If nothing matches, the reference is invalid, and each consuming feature defines what that means (e.g. for `clip-path`, the same as a `url()` that matches nothing). Level 2 could add an optional fallback argument.
- Even in places where a `<dashed-ident>` is automatically a ref, an explicit `ref()` should still be allowed. This allows abstracting it away (e.g. into a variable) without needing to know where it will be used.
- Existing feature-specific names (e.g. `anchor-name`/`anchor-scope`) keep their own namespace for `<dashed-ident>` and require `ref()` to use `ref-name`/`ref-scope`. Alternatives, if we want to gradually get rid of the feature-specific names and migrate to ref-only, both with footguns:
	- Fall back to refs if a feature-specific reference is not found
	- Define `ref-name`/`ref-scope` as a shorthand of all existing feature-specific names so that setting `ref-name` or `ref-scope` also sets all the `*-name` and `*-scope`

### Examples

A drag ghost for a sortable list:

```css
.sortable {
	ref-scope: --dragged;

	> .dragging {
		ref-name: --dragged;
		opacity: 0.4;
	}

	> .ghost {
		position: fixed;
		inset: 0 auto auto 0;
		position-anchor: ref(--dragged);
		inline-size: anchor-size(inline);
		block-size: anchor-size(block);
		translate: var(--pointer-x) var(--pointer-y);
		background: element(--dragged);
		pointer-events: none;
	}

	&:not(:has(> .dragging)) > .ghost {
		display: none;
	}
}
```

Script only toggles `.dragging` and tracks the pointer. `element()` takes an ID today, so this would otherwise need a generated ID per dragged item, and a second name (`anchor-name`) for the sizing.

---

One name, two existing features that each need their own name today (`anchor-name` and `scroll-timeline-name`):

```css
@keyframes grow {
	from { scale: 0 1; }
	to { scale: 1 1; }
}

.gallery {
	position: relative;
	ref-scope: --track;

	> .track {
		ref-name: --track;
		overflow-x: auto;
	}

	> .progress {
		position: absolute;
		position-anchor: ref(--track);
		position-area: block-end;
		transform-origin: 0 50%;

		@media (prefers-reduced-motion: no-preference) {
			animation: grow linear both;
			animation-timeline: scroll(--track inline);
		}
	}
}
```

The track is both the anchor of the progress bar and the source of its scroll timeline, under a single name.

## Open questions

1. **Boxless referents.** `ref-name` has to work on `display: none` elements (hidden labels) and never-rendered ones (`<clipPath>`). Engines do not eagerly compute style inside `display: none` or `content-visibility: hidden` subtrees, so finding a ref declared there needs a defined answer.
2. **Shadow boundaries.** Tree-scoped by default, so `::part()` and `::slotted()` rules can reference names from the same stylesheet's tree. Whether a host can deliberately export a name is open, as is what Typed OM exposes for a referent in a closed tree.
3. **Composition** Currently, different sources setting `ref-name`/`ref-scope` trample on each other. The proper solution is #1594 but that's a big lift. A short-term solution could be a `@ref` rule that can be specified separately to connect a selector to a reference but that complicates introspection. For `ref-scope` specifically, one fix could be to reduce the need for it through better resolution algorithms, e.g. picking the match with the nearest common ancestor instead of the last one in the tree.
7. **Naming.** `ref-*` vs `element-*` vs `name`/`name-scope`.

## Alternatives considered

- **Generalize `anchor-name`.** Cheapest, since it has shipped. But the name is misleading for every other consumer, it does not apply to boxless elements, and its lookup includes anchor-specific acceptability. Still an option if `anchor-name` were redefined as a legacy alias of `ref-name` with the acceptability check moved to `position-anchor`.
- **`selector(<selector>)`** ([#5811](https://github.com/w3c/csswg-drafts/issues/5811)). No answer to "relative to what", so it cannot express "the nearest instance". Reverse matching cost and `:has()`-style invalidation on every consumer.
- **Keep IDs and make them tree-scoped** (the #3320 route). Fixes shadow trees, not repetition within one tree, and still needs markup access.

## Prior art

- `anchor-name` / `anchor-scope` ([#9045](https://github.com/w3c/csswg-drafts/issues/9045)), `timeline-scope` ([#7759](https://github.com/w3c/csswg-drafts/issues/7759)), `trigger-scope`
- Unified name lookup: [#12581](https://github.com/w3c/csswg-drafts/issues/12581), [#13364](https://github.com/w3c/csswg-drafts/issues/13364)
- "Do we need container-name references?" ([#7046](https://github.com/w3c/csswg-drafts/issues/7046)): reusing one name across features
- "Unify selector functions/values" ([#5811](https://github.com/w3c/csswg-drafts/issues/5811)): `<id>` vs `element()` vs `selector()`
- Tree-scoped fragment URLs ([#3320](https://github.com/w3c/csswg-drafts/issues/3320)) and tree-scoped names in general ([#1995](https://github.com/w3c/csswg-drafts/issues/1995), [#8135](https://github.com/w3c/csswg-drafts/issues/8135), [#12090](https://github.com/w3c/csswg-drafts/issues/12090))
- GCPM `position: running(name)` + `content: element(name)`: naming an element from CSS and referencing it by ident
