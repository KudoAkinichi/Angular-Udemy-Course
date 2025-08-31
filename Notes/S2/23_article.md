# What they are (and how they differ)

* **Serialization:** Attributes live in HTML and in the DOM; properties live only on the DOM objects. If you `outerHTML` an element, you’ll see attributes you set—but not arbitrary properties.([Jake Archibald][1])
* **Types:** Attribute values are always strings; properties can be any type (booleans, numbers, objects…).([Jake Archibald][1], [MDN Web Docs][2])
* **Casing:** Attribute names are case-insensitive (`HeLlO` becomes `hello`); property names are case-sensitive. Attribute **values** remain case-sensitive.([Jake Archibald][1])

```html
<div id="t" HeLlO="world"></div>
<script>
  const el = document.getElementById('t');
  console.log(el.getAttributeNames()); // ['id','hello']
  el.anyThing = { x: 1 }; // property (object), never shows in HTML
</script>
```

# “Reflection”: when a property mirrors an attribute

Many HTML attributes have a corresponding DOM property that **reflects** them, so reading/writing the property reads/writes the attribute (and vice versa). Example: `id` → `el.id`. Specs call these *reflected attributes*.([MDN Web Docs][3], [Jake Archibald][1])

Sometimes names differ for JS reasons or ergonomics, e.g.:

* `class` ↔ `className`
* `for` (on `<label>`) ↔ `htmlFor`
* `aria-label` ↔ `ariaLabel` (ARIA reflection became cross-browser in late 2023) ([Jake Archibald][1])

# Validation, coercion & defaults (properties do more work)

Properties often validate and coerce values and/or expose defaults; attributes don’t:

* `<input>.type` returns `'text'` if the attribute is missing/invalid, even if `getAttribute('type')` is `null` or `'foo'`.
* Boolean cases such as `<details open>`: `getAttribute('open')` is `''` (empty string), but the property `details.open` is `true`, and setting `details.open = false` removes the attribute.([Jake Archibald][1], [HTML Living Standard][4])

# The notorious `value` on form fields

Inputs have **both** `value` (property) and `value` (attribute), but the property does **not** reflect the attribute. Instead, the attribute reflects to `defaultValue`. The `value` property starts by deferring to `defaultValue`, then “switches” to a separate internal value after user input or JS assignment; form reset switches it back.([Jake Archibald][1], [MDN Web Docs][5])

```html
<input value="default">
<script>
  const el = document.querySelector('input');
  console.log(el.value);         // "default"  (initially from defaultValue)
  el.value = 'typed';            // now decoupled from attribute
  console.log(el.getAttribute('value')); // "default"
  el.form.reset();               // value reverts to defaultValue
</script>
```

# A useful mental model

* **Attributes configure** an element (good for initial/default settings, semantics, and serialization).
* **Properties hold state** (current, live value; may validate/coerce).
  Jake argues `<details>`/`<dialog>` updating their own `open` **attribute** on user interaction violates this mental model because it means the DOM can mutate your attributes behind your back. He would have preferred a `defaultopen` attribute + `el.open` state property.([Jake Archibald][1])

# Frameworks: why this gets confusing

* **Preact & Vue** generally set a property if it exists on the element; otherwise they set an attribute. Their server renderers “flip” that logic.
* **React** historically leaned attribute-first (which breaks some custom-element property APIs), but **React 19** switches to the Preact/Vue approach for custom elements.
* **Lit** keeps the distinction explicit: `.prop=${…}` sets a property, plain `attr=${…}` sets an attribute.([Jake Archibald][1], [lit.dev][6])

# How to use this in Angular (quick cheat-sheet)

Angular template binding prefers **properties** with `[]`. Use `attr.` when you truly need an **attribute** (often for ARIA or non-reflected attributes).

* **Property binding (live state):**

  ```html
  <img [src]="imgUrl">
  <button [disabled]="isBusy"></button>
  <input [value]="name"> <!-- writes the property (current value) -->
  ```

  Property binding maps to the DOM property and updates reactively.([Angular][7])

* **Attribute binding (configuration/semantics):**

  ```html
  <div [attr.aria-label]="label"></div>
  <td [attr.colspan]="cols"></td>
  ```

  Use this when there is **no** matching property, or you explicitly need the attribute (e.g., some ARIA, microdata, or custom, non-reflected attributes).([Angular][7])

* **Pitfall — `value`:**
  `[value]="x"` sets the **property** (current value), not the attribute. If you want to change the **default** shown after a form reset, bind `attr.value`. Most of the time in apps you want the property (`[value]`).([MDN Web Docs][5])

* **Booleans:**
  `[disabled]="bool"` is correct (property). If you must toggle the attribute explicitly, use `[attr.disabled]="bool ? '' : null"`, but prefer the property.([HTML Living Standard][4])

# TL;DR

* Attributes: stringy, serialized, case-insensitive names, great for defaults/config.
* Properties: typed, validated/coerced, live state, case-sensitive names.
* Sometimes they reflect each other; sometimes they don’t (notably `value`). Know which you’re touching, and pick the right kind of binding.([Jake Archibald][1], [MDN Web Docs][3])

If you want, I can turn this into a small playground of examples (plain JS or Angular) to see the differences live.

[1]: https://jakearchibald.com/2024/attributes-vs-properties/ "HTML attributes vs DOM properties - JakeArchibald.com"
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Element/attributes?utm_source=chatgpt.com "Element: attributes property - MDN - Mozilla"
[3]: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Reflected_attributes?utm_source=chatgpt.com "Attribute reflection - MDN - Mozilla"
[4]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html?utm_source=chatgpt.com "Boolean attributes - HTML Standard - whatwg"
[5]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultValue?utm_source=chatgpt.com "HTMLInputElement: defaultValue property - MDN - Mozilla"
[6]: https://lit.dev/docs/templates/expressions/ "Expressions – Lit"
[7]: https://angular.dev/guide/templates/binding?utm_source=chatgpt.com "Binding dynamic text, properties and attributes - Angular"

Perfect 👍 Let’s ground Jake Archibald’s **Attributes vs Properties** discussion with **Angular examples**.
We’ll go case by case with common pitfalls.

---

# 🔹 1. **Property Binding (the default)**

```html
<!-- app.component.html -->
<img [src]="imgUrl">
<button [disabled]="isBusy">Save</button>
<input [value]="username">
```

```ts
// app.component.ts
export class AppComponent {
  imgUrl = 'https://picsum.photos/200';
  isBusy = true;
  username = 'Aryan';
}
```

* `[src]` → sets the **`src` property** of the `<img>` element (updates the live image).
* `[disabled]` → sets the **`disabled` property** (button becomes disabled).
* `[value]` → sets the **property value** of the input (the *current text inside the box*).

✅ Use `[property]` bindings when you want Angular to bind to **live element state**.

---

# 🔹 2. **Attribute Binding**

Sometimes, the HTML attribute has **no reflected property** (or is needed for semantics/ARIA).

```html
<div [attr.aria-label]="label"></div>
<td [attr.colspan]="colCount">Cell</td>
```

```ts
export class AppComponent {
  label = 'User profile';
  colCount = 3;
}
```

* `[attr.aria-label]` → sets the **ARIA attribute** for screen readers.
* `[attr.colspan]` → sets the actual **attribute**, which browsers parse when laying out a table.

✅ Use `[attr.*]` when no property exists.

---

# 🔹 3. **Boolean Attributes vs Properties**

```html
<input type="checkbox" [checked]="isChecked">
```

```ts
export class AppComponent {
  isChecked = true;
}
```

* `[checked]` sets the **property** → the checkbox is *checked on screen*.
* If you instead wrote `[attr.checked]="isChecked"`, Angular would set the attribute to `"true"`, which **does not actually check the box** (since only empty string `""` or no value is valid).

⚠️ Always use the property for booleans like `disabled`, `checked`, `selected`.

---

# 🔹 4. **The Special Case: `value`**

Jake emphasized that `<input>`’s `value` property and `value` attribute are **not the same**.

```html
<!-- Property binding -->
<input [value]="username">
<!-- Attribute binding -->
<input [attr.value]="username">
```

```ts
export class AppComponent {
  username = 'Aryan';
}
```

* `[value]` → sets the **current property value**. If the user types something else, Angular does not reset it.
* `[attr.value]` → sets the **default value attribute**. This is only used:

  * when the input is initially rendered,
  * or after calling `.form.reset()`.

So:

* Use `[value]` for **live forms**.
* Use `[attr.value]` if you want to control the **default** value.

---

# 🔹 5. **Custom Elements / Web Components**

When Angular works with a **custom element**, sometimes you must pick attribute vs property:

```html
<!-- Web component -->
<my-slider [value]="50"></my-slider>
<my-slider [attr.value]="50"></my-slider>
```

* If `my-slider` defines a **`value` property** → use `[value]`.
* If it only reads the attribute → use `[attr.value]`.

---

✅ **Rule of Thumb in Angular**

* Prefer **property binding** (`[prop]`) for live state (99% cases).
* Use **attribute binding** (`[attr.*]`) when:

  * no property exists,
  * you’re dealing with ARIA or other semantic attributes,
  * you need the *default* not the *live state* (rare).

---
