<script>
  import { tick } from 'svelte';
  import Svelecte from "$lib/Svelecte.svelte";
  import { dataset } from '../data.js';

  let creatable = false;
  let fetch = null;
  const opts = {
    start() {
      creatable = false;
      fetch = null;
      return [
        {value: 'create', text: 'I want to enter next item myself'},
        {value: 'fetch', text: 'I want to search for some colors'},
        {value: 'country', text: 'Show country list'}
      ]
    },
    create() {
      creatable = true;
      return [];
    },
    fetch() {
      fetch = '/api/colors';
      return [];
    },
    country: dataset.countries
  }

  let resolverValue = [];

  const optionResolver = (optGroups, /** @type {Set}*/ selection) => {
    let step = Array.from(selection.keys()).shift() || 'start';
    if (!Object.keys(optGroups).includes(step)) {
      step = 'start';
      tick().then(() => {
        resolverValue = [];
      });
    }
    return optGroups[step]();
  }
</script>

# Options & value

Simple arrays, object array and option groups are all supported and can be passed as `options` property.

## Defining options

### Simple array

```javascript
const simpleOptions = ['First Item', 'Second Item', 'Third Item'];
[
  { value: 'First Item', text: 'First Item' },
  { value: 'Second Item', text: 'Second Item' },
  { value: 'Third Item', text: 'Third Item' }
]
```
<Svelecte options={['First Item', 'Second Item', 'Third Item']} placeholder="Simple array as options"/>

Array is automatically converted to object array. Svelecte uses `valueField` and `labelField` property all basic
functionality and when these properties are not specified, it tries some default values (like `id` or `value` for `valueField`)

### Option groups

Option group design is inspired by its HTML counterpart. Basically group is an `object` with 2 properties representing
"label" and "options" part.

- `groupLabelField` - name of property representing group label, default value is `'label'`
- `groupItemsField` - name of property containing array of options, defaults to `'options'`

```javascript
const optionGroups = [
  {
    label: 'Option Header',
    options: [
      // group items (must be object array)
    ]
  }
]
```

<Svelecte options={dataset.countryGroups()} placeholder="Select with option groups"/>

## Working with value

Svelecte is designed as `<select>` and `<select multi>` replacement. Under the hood real `<select>` element is created
and can be used in standard forms. By default `value` property return `string` or `number` (depending on your `valueField` property)
or `string[] | number[]` for `multiple`. That's what you really need when working with forms.

⚠️ Property `name` must be defined for `<select />` element to be created.

If you need whole objects, you have 3 options:

- you can listen to `change` event
- bind `readSelection` property. It always return object or object array.
- set `valueAsObject` property. You can now bind objects to `value` property

```svelte
<script>
  let value = [14];
  let obj = {id: 14, name: 'Red color'};
</script>

<Svelecte bind:readSelection onchange={getSelectedObjects} bind:value {options}/>

<Svelecte bind:value={obj} valueAsObject {options}/>
```

## Option resolver (advanced)

`v4.0` introduces new optional property `optionResolver` which allows you to change available options
on-the-fly. This way we can overcome svelte internal reactivity limitations

You can pass it a simple function with following signature, that will return currently available option list.

```js
function(options: any, selectedKeys: Set): array
```

 behind this is that you can provide multiple `options` instances for multiple steps. Check the example.

### 🧩 Example

<br>
Selection: {resolverValue}
<Svelecte {fetch} {creatable} options={opts} {optionResolver} multiple max={2} bind:value={resolverValue} />

Source:

```svelte
<script>
let fetch = null;
let creatable = false;
const opts = {
  start() {
    creatable = false;  // change to other svelecte prop
    fetch = null;       // change to other svelecte prop
    return [
      {value: 'create', text: 'I want to enter next item myself'},
      {value: 'fetch', text: 'I want to search for some colors'},
      {value: 'country', text: 'Show country list'}
    ]
  },
  create() {
    creatable = true;
    return [];
  },
  fetch() {
    fetch = '/api/colors';
    return [];
  },
  country() { return [/** data ... */] }
}

let value = [];

const optionResolver = (optGroups, /** @type {Set}*/ selection) => {
  let step = Array.from(selection.keys()).shift() || 'start';
  if (!Object.keys(optGroups).includes(step)) {
    step = 'start';
    tick().then(() => {
      value = [];
    });
  }
  return optGroups[step]();
}
</script>

<Svelecte options={opts} {optionResolver} multiple max={2}
  {fetch} {creatable}
  bind:value={resolverValue}
/>
```
