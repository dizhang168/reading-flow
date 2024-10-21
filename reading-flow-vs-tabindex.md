# Define how reading-flow interacts with focusable display: contents elements

HTML Issue: https://github.com/whatwg/html/issues/10642

## Current status

Not resolved, but has a proposal getting prototyped.

During the CSS/WHATWG joint session at TPAC, we concluded that it would be better to make tabindex more subordinate to 'reading-flow'.

## What is the issue with the HTML Standard?

Currently, the reading-flow property has the behavior that 'reading-flow' makes the element a scope owner. tabindex is an HTML property that can re-order the elements in a focus scope. Within that scope, tabindex has priority. This means tabindex ordering is consulted first, with visual-order giving tiebreaking between identical values.

## Why positive tabindex is bad

While supported by all browsers, web users are recommended to not use positive tabindex for many reasons.

**accessibility** tabindex is ignored in the accessibility tree. As such, it is recommended to avoid using tabindex values greater than 0. However, that advice is not always followed. This causes users to see the focus jump around in a page. There is also a pattern of web developers setting tabindex to fix a bug, but then this gets copied around and is not updated correctly.

**user experience** When web developers use tabindex to reshuffle the tabbing order, it creates a bad experience for the actual user because the focus order will be less intuitive. Elements will be skipped in favor of others and the focus navigation will jump around.

**maintenance** Not all web developers understand how tabindex, focus navigation and focus scopes work.

## Example use cases of tabindex in grid/flex

Currently, because the CSS reading-flow property is not available, users are falling back on using tabindex to make the focus order follow the visual order.

For example, if someone wants a grid columns order.

```HTML
<style>
body {
  margin: 40px;
}

.wrapper {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-gap: 10px;
  background-color: #fff;
  color: #444;
}

.box {
  background-color: #444;
  color: #fff;
  border-radius: 5px;
  padding: 20px;
  font-size: 150%;
}
</style>

<div class="wrapper">
  <div class="box a">A
    <div tabindex="1">1</div>
  </div>
  <div class="box b">B
    <div tabindex="2">2</div>
  </div>
  <div class="box c">C
    <div tabindex="3">3</div>
  </div>
  <div class="box d">D
    <div tabindex="1">1</div>
  </div>
  <div class="box e">E
    <div tabindex="2">2</div>
  </div>
  <div class="box f">F
    <div tabindex="3">3</div>
  </div>
</div>
```

This will be fixed with reading-flow, such as tabindex wouldn't be necessary:

```HTML
<div class="wrapper" style="reading-flow: grid-columns">
  <div class="box a">A
    <div tabindex="0">1</div>
  </div>
  <div class="box b">B
    <div tabindex="0">2</div>
  </div>
  <div class="box c">C
    <div tabindex="0">3</div>
  </div>
  <div class="box d">D
    <div tabindex="0">1</div>
  </div>
  <div class="box e">E
    <div tabindex="0">2</div>
  </div>
  <div class="box f">F
    <div tabindex="0">3</div>
  </div>
</div>
```

Example of people asking about focus navigation on Stackoverflow, specifically for flex item descendants:

- [How to Have Children Inherit Parent tabindex Value](https://stackoverflow.com/questions/76124236/how-to-have-children-inherit-parent-tabindex-value)
- [tabindex -1 not working for child elements](https://stackoverflow.com/questions/42761987/tabindex-1-not-working-for-child-elements)
- [how can i make the tabindex of the specific class every element to -1](https://stackoverflow.com/questions/72576875/how-can-i-make-the-tabindex-of-the-specific-class-every-element-to-1)
- [Flexbox Order and Tabbing Navigation](https://stackoverflow.com/questions/44308934/flexbox-order-and-tabbing-navigation)

## Proposal

(from github issue)

1. On the "reading flow items", tabindex is ignored for ordering purposes.
   1. If tabindex is higher than 0, set it to 0.
2. On the descendants of the "reading flow items", tabindex's ordering behavior is scoped to that reading flow item subtree.
   1. The reading flow items are scope owners.

The participants in the TPAC 2024 CSSWG WHATNOT Joint meeting discussion believed this has a more predictable and desirable behavior for authors.

Ignoring tabindex's ordering effects on the reading order items themselves seems useful; it allows authors to use tabindex to define one particular ordering, in particular a non-visual one (or one for legacy UAs), and then let 'reading-flow' override that in other cases. Similarly, having each item be a scope for their descendant tabindexes seemed to better match the mental model we were working with, where 'reading-flow' effectively rearranges the reading order items.

## Spec changes

Add to **reading flow scope owner** definition:

- a reading flow item

Change the definition of **reading flow item**:
A **reading flow item** is an element whose box tree parent is a reading flow container.

### Changes to `tabindex-ordered focus navigation scope`

[https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope)

Change

The order within a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) is determined by each element's [tabindex value](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-value) and, for reading-flow focus navigation scopes, by the special rules provided by the **sequential navigation search algorithm**. Note: tabindex takes precedence over **reading flow**.

to

The order within a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) is determined by each element's [tabindex value](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-value) and the order within a [reading-flow focus navigation scope](TBD), by the special rules provided by the **reading flow sequential navigation search algorithm**.

### Changes to section 6.6.3 The tabindex attribute

Under **If the value is greater than zero**, add at the end:

If element is a reading flow item, set its tabindex default value to 0.


### Changes to new section 6.6.4 The reading flow

Change definition:

A **reading-flow focus navigation scope** is a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) where the scope owner is a reading flow scope owner.

to

A **reading-flow focus navigation scope** is a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) where the scope owner is a reading flow container or a reading flow display: contents container.



_ETC... TO BE EXPANDED directly in the HTML spec PR._

## Alternate proposals

Here are some other options, which we have eliminated (for now).

Proposal 1. Do not make reading flow items focus scope owner, positive tabindex still in effect.

- Con: it is possible for the reading flow item to be visited after its descendant. This isn't great because it doesn't fix the problem with positive tabindex.
- Con: reading-flow should only affect the items, not its descendants.

Proposal 2. Do not make reading flow items focus scope owner, positive tabindex not in effect.

- Con: Developers are not able to set positive tabindex, but it might still be useful, to be more aligned with existing behavior and if their goal is to not follow the visual-order.
- Con: reading-flow should only affect the items, not its descendants.

## Open questions

### `reading-flow: normal`

For the value `reading-flow: normal`, should it behaves as:

Proposal 1. Not a reading flow container (current implementation).

Proposal 2. Still uses the dom order, but force each reading-flow item to be a scope owner and does not have a positive tabindex.

I think option 1 still makes the most sense since we would like to have a default behavior that is the same as the current no reading-flow behavior. But maybe it would be a good idea to add another value to enable the tabindex behavior in this explainer.

## Example with the proposed change

### Example 1

```HTML
<style>
.box {
  display: grid;
  reading-flow: grid-order;
}

</style>

<div class="box" id="w" tabindex="0">
  <div id="A" tabindex="0" style="order: 3">
    <button tabindex="1">Item A1</button>
    <button tabindex="2">Item A2</button>
  </div>
  <div id="B" tabindex="0" style="order: 1">
    <button tabindex="1">Item B1</button>
    <button tabindex="2">Item B2</button>
  </div>
  <div id="C" tabindex="0" style="order: 2">
    <button tabindex="1">Item C1</button>
    <button tabindex="2">Item C2</button>
  </div>
</div>
```

Note the example above is uncommon, but allows us to test different cases.

With the current implementation, the order would be:

- wrapper
  - tabindex=1
    - order: 1 so B1
    - order: 2 so C1
    - order: 3 so A1
  - tabindex=2
    - order: 1 so B2
    - order: 2 so C2
    - order: 3 so A2
  - tabindex=0
    - order: 1 so B
    - order: 2 so C
    - order: 3 so A

By changing so tabindex doesn't affect reading flow item, but will making reading flow items' tabindex scope owner, the order changes:

- wrapper
  - order: 1 so B
    - tabindex=1 so B1
    - tabindex=2 so B2
  - order: 2 so C
    - tabindex=1 so C1
    - tabindex=2 so C2
  - order: 3 so A
    - tabindex=1 so A1
    - tabindex=2 so A2

In this case, having the scope on the reading-flow item does look and feel more natural.
If the reading-flow item was not a scope owner, then tabindex takes priority and it would be easy to have cases where descendants would be visited first.
