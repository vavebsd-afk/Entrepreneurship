# tweaks-panel

A reusable floating panel of design controls for HTML/React prototypes. Drop it in, wire up a few controls, and get an instant live-editing UI — sliders, segmented radios, color pickers, toggles, steppers, and more — without hand-rolling any of them.

Designed for use in standalone HTML files (React + Babel via CDN) and in Anthropic's [Claude Artifacts](https://claude.ai) environment.

---

## Contents

- [Quick start](#quick-start)
- [API](#api)
  - [`useTweaks`](#usetweaks)
  - [`TweaksPanel`](#tweakspanel)
  - [`TweakSection`](#tweaksection)
  - [`TweakSlider`](#tweakslider)
  - [`TweakToggle`](#tweaktoggle)
  - [`TweakRadio`](#tweakradio)
  - [`TweakSelect`](#tweakselect)
  - [`TweakText`](#tweaktext)
  - [`TweakNumber`](#tweaknumber)
  - [`TweakColor`](#tweakcolor)
  - [`TweakButton`](#tweakbutton)
- [Host protocol](#host-protocol)
- [Persistence](#persistence)
- [Deck stage integration](#deck-stage-integration)

---

## Quick start

Load React, Babel, and the panel script, then use it in your component:

```html
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script src="tweaks-panel.jsx" type="text/babel"></script>

<script type="text/babel">
  const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
    "primaryColor": "#D97757",
    "fontSize": 16,
    "density": "regular",
    "dark": false
  }/*EDITMODE-END*/;

  function App() {
    const [t, setTweak] = useTweaks(TWEAK_DEFAULTS);

    return (
      <div style={{ fontSize: t.fontSize, color: t.primaryColor }}>
        Hello world

        <TweaksPanel>
          <TweakSection label="Typography" />
          <TweakSlider label="Font size" value={t.fontSize} min={10} max={32} unit="px"
                       onChange={(v) => setTweak('fontSize', v)} />
          <TweakRadio  label="Density" value={t.density}
                       options={['compact', 'regular', 'comfy']}
                       onChange={(v) => setTweak('density', v)} />
          <TweakSection label="Theme" />
          <TweakColor  label="Primary" value={t.primaryColor}
                       options={['#D97757', '#2A6FDB', '#1F8A5B', '#7A5AE0']}
                       onChange={(v) => setTweak('primaryColor', v)} />
          <TweakToggle label="Dark mode" value={t.dark}
                       onChange={(v) => setTweak('dark', v)} />
        </TweaksPanel>
      </div>
    );
  }

  ReactDOM.createRoot(document.getElementById('root')).render(<App />);
</script>
```

The panel is hidden by default and opens when the host posts `__activate_edit_mode` (see [Host protocol](#host-protocol)). You can open it immediately for development by adding a button or triggering the message yourself.

---

## API

All components and hooks are exported onto `window` so they're available globally in HTML/Babel files.

---

### `useTweaks`

```js
const [t, setTweak] = useTweaks(defaults);
```

React hook that manages tweak values. `defaults` is a plain object; `t` is the current values object.

**`setTweak(key, value)`** — update a single key.

**`setTweak({ key: value, ... })`** — update multiple keys at once.

Both forms post the change to the parent window (`__edit_mode_set_keys`) for host-side persistence, and dispatch a `tweakchange` `CustomEvent` on `window` so in-page listeners can react.

---

### `TweaksPanel`

```jsx
<TweaksPanel title="Tweaks" noDeckControls={false}>
  {/* controls */}
</TweaksPanel>
```

The floating shell. Renders nothing until the host activates edit mode.

| Prop | Type | Default | Description |
|---|---|---|---|
| `title` | `string` | `"Tweaks"` | Header label |
| `noDeckControls` | `boolean` | `false` | Suppress the auto-injected deck rail toggle (see [Deck stage integration](#deck-stage-integration)) |

The panel is **draggable** (mouse-drag the header) and clamps itself to the viewport on resize.

---

### `TweakSection`

```jsx
<TweakSection label="Typography" />
```

A visual section divider — an uppercase label that groups controls below it. Pass `children` if you want the section to visually wrap them; otherwise just place it above the relevant controls.

---

### `TweakSlider`

```jsx
<TweakSlider
  label="Font size"
  value={t.fontSize}
  min={10}
  max={32}
  step={1}
  unit="px"
  onChange={(v) => setTweak('fontSize', v)}
/>
```

A labeled range slider. The current value is shown next to the label. `onChange` receives a `number`.

| Prop | Type | Default |
|---|---|---|
| `label` | `string` | — |
| `value` | `number` | — |
| `min` | `number` | `0` |
| `max` | `number` | `100` |
| `step` | `number` | `1` |
| `unit` | `string` | `""` |
| `onChange` | `(number) => void` | — |

---

### `TweakToggle`

```jsx
<TweakToggle label="Dark mode" value={t.dark} onChange={(v) => setTweak('dark', v)} />
```

An iOS-style toggle switch. `onChange` receives a `boolean`.

---

### `TweakRadio`

```jsx
<TweakRadio
  label="Density"
  value={t.density}
  options={['compact', 'regular', 'comfy']}
  onChange={(v) => setTweak('density', v)}
/>
```

A segmented control (pill-style radio). Supports drag-to-select across segments.

Options can be plain strings/numbers or `{ value, label }` objects:

```jsx
options={[
  { value: 'sm', label: 'Small' },
  { value: 'md', label: 'Medium' },
  { value: 'lg', label: 'Large' },
]}
```

**Automatic fallback:** if there are more than 3 options, or the option labels are too long to fit, `TweakRadio` renders a [`TweakSelect`](#tweakselect) dropdown instead. This is transparent — you use the same API either way.

---

### `TweakSelect`

```jsx
<TweakSelect
  label="Font family"
  value={t.font}
  options={['Inter', 'Georgia', 'JetBrains Mono']}
  onChange={(v) => setTweak('font', v)}
/>
```

An explicit dropdown. Options follow the same `string | { value, label }` shape as `TweakRadio`. Prefer `TweakRadio` for short option lists — it renders as a segmented control when it fits.

---

### `TweakText`

```jsx
<TweakText
  label="Tagline"
  value={t.tagline}
  placeholder="Enter text…"
  onChange={(v) => setTweak('tagline', v)}
/>
```

A single-line text field. `onChange` receives a `string`.

---

### `TweakNumber`

```jsx
<TweakNumber
  label="Columns"
  value={t.columns}
  min={1}
  max={12}
  step={1}
  unit="col"
  onChange={(v) => setTweak('columns', v)}
/>
```

A numeric input with an optional unit label. The label area also acts as a **scrub handle** — click-drag left/right to nudge the value. `onChange` receives a clamped `number`.

| Prop | Type | Default |
|---|---|---|
| `label` | `string` | — |
| `value` | `number` | — |
| `min` | `number` | — |
| `max` | `number` | — |
| `step` | `number` | `1` |
| `unit` | `string` | `""` |
| `onChange` | `(number) => void` | — |

---

### `TweakColor`

```jsx
// Single color picker with curated swatches
<TweakColor
  label="Accent"
  value={t.accent}
  options={['#D97757', '#2A6FDB', '#1F8A5B', '#7A5AE0']}
  onChange={(v) => setTweak('accent', v)}
/>

// Palette picker — each option is an array of hex strings
<TweakColor
  label="Palette"
  value={t.palette}
  options={[
    ['#D97757', '#29261b', '#f6f4ef'],
    ['#475569', '#0f172a', '#f1f5f9'],
    ['#7A5AE0', '#1a1030', '#f5f3ff'],
  ]}
  onChange={(v) => setTweak('palette', v)}
/>
```

A curated chip picker. Each chip renders a color swatch (or a palette card for array options). The active chip shows a checkmark; its color automatically contrasts against the background.

Without `options`, falls back to a native `<input type="color">`.

`onChange` emits the value in the **same shape it was passed** — a `string` option emits a string, an array option emits an array.

---

### `TweakButton`

```jsx
<TweakButton label="Reset" onClick={() => setTweak(TWEAK_DEFAULTS)} />
<TweakButton label="Copy JSON" onClick={handleCopy} secondary />
```

A small action button. Pass `secondary` for a lower-emphasis style.

---

## Host protocol

`TweaksPanel` communicates with a parent window over `postMessage`. This is what drives the toolbar toggle in environments like Claude Artifacts.

| Direction | Message type | Meaning |
|---|---|---|
| Panel → host | `__edit_mode_available` | Panel is mounted and ready to open |
| Host → panel | `__activate_edit_mode` | Show the panel |
| Host → panel | `__deactivate_edit_mode` | Hide the panel |
| Panel → host | `__edit_mode_dismissed` | User clicked ✕; host should flip its toggle off |
| Panel → host | `__edit_mode_set_keys` | Value changed; host should persist `edits` to the `EDITMODE` block |

The availability signal is posted **after** the listener is registered, so a `__activate_edit_mode` that arrives synchronously in response will always be caught.

---

## Persistence

Values are persisted by the host. The pattern uses a specially-delimited JSON block in the source file:

```js
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "fontSize": 16,
  "accent": "#D97757"
}/*EDITMODE-END*/;
```

When `setTweak` is called, the panel posts `{ type: '__edit_mode_set_keys', edits }` to the parent. A compatible host (e.g., Claude's artifact editor) rewrites the content between the `EDITMODE-BEGIN` / `EDITMODE-END` markers on disk, so tweaks survive a reload.

In development without a host, values live in React state for the session only.

---

## Deck stage integration

If a `<deck-stage>` custom element is present on the page and has announced itself via `__omelette_rail_enabled`, `TweaksPanel` automatically appends a **Thumbnail rail** toggle under a "Deck" section. The toggle posts `{ type: '__deck_rail_visible', on: boolean }` to `window` and mirrors the value from `localStorage` key `deck-stage.railVisible`.

Pass `noDeckControls` to suppress this if you want to place the toggle yourself.
