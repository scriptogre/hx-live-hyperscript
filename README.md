# hx-live-hyperscript

Hyperscript-flavored shorthand for the htmx [`hx-live`](https://four.htmx.org/extensions/hx-live) extension.

It rewrites shorthand expressions to the core `q()` API before `hx-live`
evaluates them. It adds syntax, not state or runtime behavior.

> [!NOTE]
> Packaging and standalone test infrastructure are still pending.

## Load the Extension

Load scripts in this order:

```text
htmx
└── hx-live
    └── hx-live-hyperscript
```

The extension source is at
[`src/ext/hx-live-hyperscript.js`](src/ext/hx-live-hyperscript.js).

## Use Local State

Prefix an attribute with `@` to read or write it on the current element.

```html
<button aria-pressed="false"
        hx-on:click="@aria-pressed = !@aria-pressed">
    Mute
</button>
```

The core form is:

```js
q(this).aria.pressed = !q(this).aria.pressed
```

The same syntax covers each state namespace:

```js
@data-count++
@aria-expanded = true
@.active = true
@readonly = true
```

## Use Shared State

Prefix an attribute with `^` to use its nearest owner, starting at the current
element.

```html
<section data-count="0">
    <button hx-on:click="^data-count++">Add</button>
    <output :text="^data-count"></output>
</section>
```

```text
section[data-count]
       ▲
       │ ^data-count
     button
```

The core form is:

```js
q(this).closest.data.count++
```

Use the same owner lookup after a selection:

```js
q('.item').^data-open = true
q('.option').^aria-activedescendant = this.id
```

## Select Elements

Use `#id` for an ID or `<.../>` for any selector accepted by `q()`.

```js
#cart.@data-count++
<.row/>.@hidden = true
<previous input/>.@value
<.error/>.^data-invalid = true
```

```text
#cart              → q('#cart')
<.row/>            → q('.row')
<previous input/>  → q('previous input')
```

Directional selectors use the core `q()` grammar:

```js
<first .item/>
<last .item/>
<next .item/>
<previous .item/>
<closest .field/>
<.item in #panel/>
```

## Continue from a Selection

Use `'s` to access state or a property from a selector literal.

```js
#cart's @data-count++
<.field/>'s @value
```

It is equivalent to continuing through the selected `q()` proxy.

## Name Attributes in Helpers

Inside `toggle()` and `take()`, a sigil replaces the string attribute name.

```js
toggle(@aria-expanded)
toggle(@data-view, 'grid', 'list')
take(@aria-selected)
take(@.active)
```

These have the same behavior as:

```js
toggle('aria-expanded')
toggle('data-view', 'grid', 'list')
take('aria-selected')
take('.active')
```

## Reference

### Attribute Sigils

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

### Namespaces

Use a wildcard or `@class` to access a whole namespace:

```js
{ ...@data-* }
{ ...^data-* }
@class.assign({ active: true, loading: false })
```

### Selector Literals

| Shorthand | Core expression |
| --- | --- |
| `#cart` | `q('#cart')` |
| `<.row/>` | `q('.row')` |
| `<previous input/>` | `q('previous input')` |
| `<.item in #panel/>` | `q('.item in #panel')` |

## How It Works

The extension hooks into expression compilation and rewrites supported syntax:

```text
HTML expression
      │
      ▼
hx-live-hyperscript
      │  syntax rewrite
      ▼
core q() expression
      │
      ▼
hx-live evaluation
```

For example:

```text
^data-count++
      ↓
q(this).closest.data.count++
```

The rewriter skips strings, comments, regular expressions, and raw template
text. Normal JavaScript keeps its meaning when a value comes first:

```js
flags ^ mask
count < max
```

## Development

```text
src/ext/hx-live-hyperscript.js        extension source
test/tests/ext/hx-live-hyperscript.js browser tests
```

## License

[BSD Zero Clause](LICENSE)
