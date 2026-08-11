# hx-live-hyperscript

Hyperscript-flavored shorthand for the htmx [`hx-live`](https://four.htmx.org/extensions/hx-live) extension.

```text
expression
    │
    ▼
hx-live-hyperscript
    │  rewrites syntax
    ▼
core q() expression
    │
    ▼
hx-live
```

It adds syntax. It does not add state or a runtime API.

## Status

```text
source       ✓ migrated
tests        ✓ migrated
packaging    ◌ pending
standalone   ◌ pending
```

Load order:

```text
htmx
└── hx-live
    └── hx-live-hyperscript
```

## State

Two sigils choose where state lives:

```text
@  selected element
^  nearest owner, starting at the selected element
```

```text
<section data-count="1">
    <button>+</button>
       │       │
       │       └── ^data-count  → section
       └────────── @data-count  → button
```

| Shorthand | Core expression |
| --- | --- |
| `@data-count` | `q(this).data.count` |
| `^data-count` | `q(this).closest.data.count` |
| `@aria-expanded` | `q(this).aria.expanded` |
| `^aria-expanded` | `q(this).closest.aria.expanded` |
| `@.active` | `q(this).class.active` |
| `^.active` | `q(this).closest.class.active` |
| `@readonly` | `q(this).attr.readonly` |
| `^readonly` | `q(this).closest.attr.readonly` |

Use a sigil after `q()` to target another selection:

```js
q('#cart').@data-count++
q('.tab').@aria-selected = true
q('.item').^data-open = true
```

Whole namespaces stay available:

```js
{ ...@data-* }
{ ...^data-* }
@class.assign({ active: true, loading: false })
```

## Selectors

Selector literals remove the `q('...')` wrapper:

```text
#cart              → q('#cart')
<.row/>            → q('.row')
<previous input/>  → q('previous input')
```

```js
#cart.@data-count++
<.row/>.@hidden = true
<previous input/>.@value
<.error/>.^data-invalid = true
```

Directional selectors use the same grammar as `q()`:

```text
<first .item/>
<last .item/>
<next .item/>
<previous .item/>
<closest .field/>
<.item in #panel/>
```

## Possessives

`'s` continues from a selector literal:

```js
#cart's @data-count++
<.field/>'s @value
```

```text
#cart's @data-count
  │          │
  │          └── state on #cart
  └───────────── selected element
```

## Helpers

Inside `toggle()` and `take()`, sigils replace string attribute names:

```text
toggle(@aria-expanded)               toggle('aria-expanded')
toggle(@data-view, 'grid', 'list')   toggle('data-view', 'grid', 'list')
take(@aria-selected)                 take('aria-selected')
take(@.active)                       take('.active')
```

Both columns have the same behavior.

## Boundaries

The rewriter skips:

```text
strings
comments
regular expressions
raw template text
```

Normal JavaScript keeps its meaning when a value comes first:

```js
flags ^ mask
count < max
```

## Files

```text
src/ext/hx-live-hyperscript.js
test/tests/ext/hx-live-hyperscript.js
```

## License

[BSD Zero Clause](LICENSE)
