# Define how reading-flow interacts with focusable display: contents elements

CSS Issue: https://github.com/w3c/csswg-drafts/issues/9230
HTML Issue: https://github.com/whatwg/html/issues/10533

## Current Status

Resolved:

1. [All focusable elements should remain focusable in reading-flow.](https://github.com/w3c/csswg-drafts/issues/9230#issuecomment-2165098385)
2. [display:contents focusable element occurs immediately before its first child in visual order](https://github.com/w3c/csswg-drafts/issues/9230#issuecomment-2369780005)

Still open:

1. Should display: contents that inherits from a reading-flow container layout parent be a focus scope owner?

Note: This document is a copy of the HTML issue linked about. It is saved in this repository for tracking.

## What is the issue with the HTML Standard?

The CSS Working Group has resolved to add the new reading-flow property (https://github.com/w3c/csswg-drafts/issues/8589, [spec](https://drafts.csswg.org/css-display-4/#reading-flow)) to enable focus navigation in visual order for layout items that might not be displayed in source order (such as grid, flex and masonry items). Chromium is implementing the new property and opened a [proposal](https://github.com/whatwg/html/issues/10407) on the needed HTML spec change. The explainer can be found [here](https://github.com/dizhang168/reading-order-items).

One remaining question is about `display:contents` elements in this context. `display: contents` elements are elements that do not have a rendered style or layout. They can still be focusable if the tabindex attribute is set, per [https://github.com/w3c/csswg-drafts/issues/2632](https://github.com/w3c/csswg-drafts/issues/2632) and https://github.com/w3c/csswg-drafts/issues/9230#issuecomment-2165098385. How `display: contents` elements should be navigated is not decided yet ([https://github.com/w3c/csswg-drafts/issues/9230](https://github.com/w3c/csswg-drafts/issues/9230)).

This issue discusses a few possibilities for handling `display:contents` elements with reading-flow.

## Option 1: Visit all items of a reading flow container together

All reading flow items are visited in the order they are defined by the layout algorithm for CSS reading-flow. Afterward, we visit all descendant display: contents elements, in case they are focusable. Only one focus scope is created per grid/flex container.

### Spec change

Update steps of [find the next item in reading order](https://github.com/dizhang168/reading-order-items?tab=readme-ov-file#reading-flow-sequential-navigation-search-algorithm).

1. Let _reading flow layout items_ be the list of reading flow items owned by a reading flow _scope_, sorted in reading flow.
2. Let _display contents elements_ be the list of display: contents **descendants** of the reading flow container, as they appear in source order.
3. Let _reading flow items_ be the concatenation of _reading flow layout items \_and_ display contents elements.\_
4. Visit step 2-4 of the algorithm to [find the next item in reading order](https://github.com/dizhang168/reading-order-items?tab=readme-ov-file#reading-flow-sequential-navigation-search-algorithm).

**Pro**: The elements are mostly visited in visual order.

**Con**: This would result in a mismatch with the Accessibility tree, which expects visiting siblings together and doesn’t want to combine descendants of different parents together.

## Option 2: A reading flow item with display: contents is a focus scope owner

Here, we update the definition of a reading flow container such that a container with display: contents will result in a new focus scope. Within it, we will visit all the direct children of this container starting with the reading flow items and then the display: contents elements. If within those elements, a display: contents element is found, a new focus scope will be created.

### Spec change

Add to [definition of a reading flow container](https://github.com/dizhang168/reading-order-items?tab=readme-ov-file#definitions):

- a flex container that has the CSS property reading-flow set to flex-visual or flex-flow.
- a **display:content element whose box tree parent** has CSS property display set to flex and CSS property reading-flow set to flex-visual or flex-flow.
- a grid container that has the CSS property reading-flow set to grid-rows, grid-columns or grid-order.
- a **display:content element whose box tree parent** has CSS property display set to grid and CSS property reading-flow set to grid-rows, grid-columns or grid-order.

Replace step 1 of [find the next item in reading order](https://github.com/dizhang168/reading-order-items?tab=readme-ov-file#reading-flow-sequential-navigation-search-algorithm).

1. Let _reading flow layout items_ be the list of reading flow items owned by _scope_, sorted in reading flow order.
   1. Filter out any item whose parent is not the reading flow container.
2. Let _display contents elements_ be the list of display: contents **direct children** of _scope_, as they appear in source order.
3. Let _reading flow items_ be the concatenation of _reading flow layout items \_and_ display contents elements.\_

**Pro**: This would match with the Accessibility tree.

**Con**: We are more likely to get a mismatch between the visual and the reading flow order (see Example 2).

## Examples

### Example 1: An item has display: contents and has no element child

```html
<div style="display:flex; reading-flow: flex-visual">
  <div id="A" style="display:contents; order: 2">A</div>
  <button id="B" style="order: 3">B</button>
  <button id="C" style="order: 1">C</button>
</div>
```

This will be displayed as:
<img width="69" alt="image" src="https://github.com/user-attachments/assets/4cdce8dd-1f2d-4286-9bdb-8654cc52f9b1">

However, neither div nor text node A is not recognized as a flex item and the order value is not used. Additionally, A is not focusable.

The visit order should be: C -> B.

### Example 2: An item has display: contents, is focusable and has no child

```html
<div style="display:flex; reading-flow: flex-visual">
  <div id="A" style="display:contents; order: 2" tabindex="0">A</div>
  <button id="B" style="order: 3">B</button>
  <button id="C" style="order: 1">C</button>
</div>
```

This will be displayed as:

<img width="69" alt="image" src="https://github.com/user-attachments/assets/2c9e2689-655f-4024-903d-d4db1634d7c0">

However, text node A is not recognized as a flex item and the order value is not used.
The visit order should be: C -> B -> A.

Now, the tricky part when the display contents element has children.

### Example 3: An item has display: contents, is focusable and has children

```html
<div style="display:flex; reading-flow: flex-visual">
  <div id="d1" style="display: contents" tabindex="0">
    <button style="order: 3" id="C">C</button>
    <button style="order: 1" id="A">A</button>
    <div id="d2" style="display: contents" tabindex="0">
      <button style="order: 4" id="D">D</button>
      <button style="order: 2" id="B">B</button>
    </div>
  </div>
</div>
```

The rendered output:
<img width="153" alt="image" src="https://github.com/user-attachments/assets/94b80b49-189e-4f96-a91a-4746efaeff35">

Given option 1, we would visit in the order A -> B -> C -> D -> d1 -> d2.
Given option 2, we would visit in the order div1 -> A -> C -> div2 -> B -> D.

### Example 4: A Slot is focusable and has children

```html
<div>
  <template shadowrootmode="open">
    <style>
      .wrapper {
        display: flex;
        reading-flow: flex-visual;
      }
    </style>
    <div class="wrapper">
      <button style="order: 4" id="D">D</button>
      <!-- Because slot has display: contents, the order value doesn't get used -->
      <slot style="order: 10" tabindex="0" id="slot"></slot>
      <button style="order: 2" id="B">B</button>
    </div>
  </template>
  <button style="order: 1" id="A">A</button>
  <button style="order: 3" id="C">C</button>
</div>
```

The rendered output:
<img width="153" alt="image" src="https://github.com/user-attachments/assets/94b80b49-189e-4f96-a91a-4746efaeff35">

Given option 1, we would visit in the order B -> D -> slot -> A -> C.
This is because it recognizes the slot as a scope owner and its children should not be visited until that scope is created.
Given option 2, we would visit in the order B -> D -> slot -> A -> C.

Note that we have already requested the feedback from Accessibility implementer @aleventhal. They prefer Option 2 because of how the accessibility tree is constructed. They also don’t think it is common to have display: contents mixed with non display: contents flex/grid items so this issue should not be common. However, just because a case is uncommon doesn’t mean we shouldn’t consider it. Especially because we want this feature to work well with web components.
We have opened a [blog post](https://developer.chrome.com/blog/reading-flow-display-contents?hl=en) to request developer feedback.
