# CSS reading-flow Explainer

Authors: Di Zhang, Mason Freed

Last updated: June 18, 2024

Issue: https://github.com/whatwg/html/issues/10407

## Introduction to the problem

Focus navigation is the mechanism that allows users to navigate and access the contents of a website using their keyboard. Currently, this navigation follows the source order, aka the order the elements are defined in the DOM tree. This causes a disconnect when the elements are displayed in a different order, using a flexbox or grid layout, where the visual reading flow can be different from the underlying source order using features like the `order` property.

The CSS Working Group proposed to solve this problem using the [new CSS property reading-flow](https://drafts.csswg.org/css-display-4/#reading-flow). This property allows developers to specify how items within a flex or grid container should be read. In this explainer, we are proposing changes to the WHATWG specifications to support this new property for sequential focus navigation. Namely, we propose adding a new focus scope owner and more steps to the sequential navigation search algorithm.

Note this feature will become even more valuable in the upcoming CSS Masonry, which uses an automatic layout method in which items are displayed in a harder-to-predict order.

## New specifications in WHATWG

### Definitions

A **reading flow container** is either

- a flex container that has the CSS property `reading-flow` set to `flex-visual` or `flex-flow`.
- a grid container that has the CSS property `reading-flow` set to `grid-rows`, `grid-columns` or `grid-order`.

A **reading flow item** is a flex item or grid item whose layout parent is a reading flow container.

### New Focus Navigation Scope Owner

The definition of [focus navigation scope owner](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) should be modified:

_A node is a focus navigation scope owner if it is a Document, a shadow host, a slot, an element in the popover showing state which also has a popover invoker set, or a **reading flow container**._

Add this to the [associated focus navigation owner](https://html.spec.whatwg.org/multipage/interaction.html#associated-focus-navigation-owner) algorithm, after existing step 2 and before the existing step 3:

_2.5. If element’s layout parent is a reading flow container, then return the reading flow container element._

### Changes to `sequential navigation search algorithm`

[https://html.spec.whatwg.org/multipage/interaction.html#sequential-navigation-search-algorithm](https://html.spec.whatwg.org/multipage/interaction.html#sequential-navigation-search-algorithm)

Add new steps after existing step 1 and before the existing step 2:

1.5. If _candidate_ is a **reading flow item** or null, _direction_ is "forward", and _starting point_ is in a **reading-flow focus navigation scope** _scope_, then let _new candidate_ be the result of the **reading flow sequential navigation search algorithm** with _candidate_, _direction_ and _starting point_’s focus navigation _scope_.

If _starting point_ is a **reading flow item**, _direction_ is "backward", and _starting point_ is in a **reading-flow focus navigation scope** _scope_, then let the _new candidate_ be the result of the **reading flow sequential navigation search algorithm** with _starting point_, _direction_ and _starting point_’s focus navigation _scope_.

If _new candidate_ is null, then let _starting point_ be _candidate_, and return to step 1 of this algorithm. Otherwise, let _candidate_ be _new candidate_.

#### reading flow sequential navigation search algorithm

To **find the next item in reading flow**, given a reading flow item _current_, a direction _direction_ and a reading-flow focus navigation scope _scope_, perform the following steps. They return an Element.

1. Let _reading flow items_ be the list of reading flow items owned by _scope_, sorted in **reading flow**.
2. If _reading flow items_ is empty, return null.
3. If _direction_ is “forward”, then:
   1. If _current_ is the reading flow item from _reading flow items_ that comes first in DOM tree order, return first item in _reading flow items_.
   2. If _current_ is null, let _previous_ be the reading flow item from _reading flow items_ that comes last in DOM tree order.
   3. Otherwise, let _previous_ be the reading flow item that comes before _current_ in DOM tree order.
   4. If _previous_ is the last item in _reading flow items_, return null.
   5. Otherwise, return the item that comes after _previous_ in _reading flow items_.
4. Otherwise:
   1. Let _previous_ be the item that comes before _current_ in _reading flow items_.
   2. If _previous_ is null, return null.
   3. Otherwise, if _previous_ does not have any DOM tree descendants, return _previous_.
   4. Otherwise, return the last DOM tree descendant of _previous_.

### Changes to `tabindex-ordered focus navigation scope`

[https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope)

Change

The order within a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) is determined by each element's [tabindex value](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-value), as described in the section below.

to

The order within a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) is determined by each element's [tabindex value](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-value) and, for reading-flow focus navigation scopes, by the special rules provided by the **sequential navigation search algorithm**. Note: tabindex takes precedence over **reading flow**.

### Add new section 6.6.4 The reading flow

Add this new section after existing section [6.6.3 The tabindex attribute](https://html.spec.whatwg.org/multipage/interaction.html#the-tabindex-attribute):

A **reading-flow focus navigation scope** is a [tabindex-ordered focus navigation scope](https://html.spec.whatwg.org/multipage/interaction.html#tabindex-ordered-focus-navigation-scope) where the scope owner is a reading flow container.

The **reading flow** for a **reading-flow focus navigation scope** is determined by the container’s [reading-flow](https://drafts.csswg.org/css-display-4/#propdef-reading-flow) value:

- For `flex-visual`: the reading flow should be defined by the flex items, sorted in the visual reading flow and taking the writing mode into account.
- For `flex-flow`: the reading flow should be defined by the flex items, sorted by the CSS ‘flex-flow’ direction.
- For `grid-rows`: the reading flow should be defined by the grid items, sorted first by their displayed row order, and then by their column order, taking the writing mode into account.
- For `grid-columns`: the reading flow should be defined by the grid items, sorted first by their displayed column order, and then by their row order, taking the writing mode into account.
- For `grid-order`: the reading flow should follow the [order-modified document order](https://drafts.csswg.org/css-display-4/#order-modified-document-order).

## Examples

In the following examples, we use the new **reading flow sequential navigation search algorithm** to find the next reading flow item to navigate to.

### Example - `grid-order`

```HTML
<!DOCTYPE html>
<style>
.wrapper {
  display: grid;
  reading-flow: grid-order;
}
</style>
<div class="wrapper">
 <button id="a" style="order: 2">A</button>
 <button id="b" style="order: 4">B</button>
 <button id="c" style="order: 3">C</button>
 <button id="d" style="order: 1">D</button>
</div>
```

<img src="images/example-1-graph.png" width="400" />

**Forward navigation**

Start at the first element in the reading-flow focus navigation scope:

1. Find first element in reading flow items, D.
2. Return D

Given _starting point_ D, _candidate_ null and _direction_ forward:

1. _current_ is null.
2. _previous_ is last item in DOM tree order, D.
3. Move forward from D in reading flow.
4. Return A.

...

Given _starting point_ B, _candidate_ C and _direction_ forward:

1. _current_ is C.
2. _previous_ is B.
3. Move forward from B in reading flow.
4. Return null.

**Backward navigation**

End at the last element in the reading-flow focus navigation scope:

1. Find last element in reading flow items, B.
2. Return B

Given _starting point_ B and _direction_ backward:

1. _current_ is B.
2. _previous_ is C, it has no descendants.
3. Return C.

...

Given _starting point_ D and _direction_ backward:

1. _current_ is D.
2. _previous_ is null.
3. Return null.

### Example - `grid-order` with nested children

```HTML
<!DOCTYPE html>
<style>
.wrapper {
 display: grid;
 reading-flow: grid-order;
}
</style>
<div class="wrapper">
 <div id="A" style="order: 2">A
   <button id="a" style="order: 3">a</button>
 </div>
 <div id="B" style="order: 3">B
   <button id="b">b</button>
 </div>
 <div id="C" style="order: 1">C
   <button id="c">c</button>
 </div>
</div>
```

<img src="images/example-2-graph.png" width="300" />

**Forward navigation**

Start at the first element in the reading-flow focus navigation scope:

1. Find first element in reading flow items, C.
2. Return C.

Given _starting point_ C, _candidate_ c and _direction_ forward:

1. Not in reading flow sequential navigation since c is not a reading flow item.
2. Return c.

Given _starting point_ c, _candidate_ null and _direction_ forward:

1. _current_ is null.
2. _previous_ is last item in DOM tree order, C.
3. Move forward from C in reading flow.
4. Return A.

...

Given _starting point_ b, _candidate_ C and _direction_ forward:

1. _current_ is C.
2. _previous_ is B.
3. Move forward from B in reading flow.
4. Return null.

**Backward navigation**

End at the last element in the reading-flow focus navigation scope:

1. Find last element in reading flow, B.
2. Find last descendant within it, b
3. Return b.

Given _starting point_ b and _direction_ backward:

1. Not in reading flow sequential navigation since b is not a reading flow item.
2. Return B.

Given _starting point_ B and _direction_ backward:

1. _current_ is B.
2. _previous_ is A, it has descendant a.
3. Return a.

...

Given _starting point_ C and _direction_ backward:

1. _current_ is C.
2. _previous_ is null.
3. Return null.

## Open Questions

#### What should be the reading flow if reading flow items are defined through display: contents and cross different scopes?

```HTML
<!DOCTYPE html>
<meta charset="utf-8">

<div>
<template shadowrootmode="open" shadowrootdelegatesfocus>
<style>
.wrapper {
  display: flex;
  reading-flow: flex-visual;
}
</style>
<div class=wrapper>
<button id="A" style="order: 2">Item A</button>
<slot></slot>
<button id="C" style="order: 4">Item C</button>
</div>
</template>

<button id="B1" style="order: 1">Slotted B1</button>
<button id="B2" style="order: 3">Slotted B2</button>
</div>
```

Render:

<img src="images/open-question-1-render.png" width="300" />

Source order: A,B1,B2,C

reading flow: B1,A,B2,C

Given the [flattened tabindex-ordered focus navigation ](https://html.spec.whatwg.org/multipage/interaction.html#flattened-tabindex-ordered-focus-navigation-scope)scope, step 2.2, we should visit all elements within a scope together (so B1, then B2). However, that is visually the wrong order.

#### What should be the reading flow if a reading flow item is a display: contents scope owner?

```HTML
<!DOCTYPE html>
<meta charset="utf-8">

<style>
 .wrapper {
   display: flex;
   reading-flow: flex-visual;
 }
 </style>
<div class=wrapper id="root">
 <div style="display: contents">
   <template shadowrootmode=open>
     <slot></slot>
   </template>
   <button id="A2" style="order: 2">A</button>
   <button id="B2" style="order: 1">B</button>
 </div>
 <button id="C" style="order: 3">C</button>
</div>
```

Render:

<img src="images/open-question-2-render.png" width="100" />

Source order: A,B,C

reading flow: B,A,C

In this case, we have a DIV that is:

- A Shadow host (so a focus navigation scope owner)
- Its layout parent is a reading flow container
- Has display: contents

Should the DIV qualify as a reading flow item? If so, it can be included in the defined **reading-flow focus navigation scope**, but there isn’t a straightforward way to include it in the reading flow, since it isn’t part of the reading flow container, and isn’t displayed on its own. So it’s unclear where it belongs with respect to the other reading flow items. Perhaps the best option is to say that `display:contents` items inside **reading flow container**s are not focusable.

## List of relevant issues

[csswg-drafts issue 9230](https://github.com/w3c/csswg-drafts/issues/9230) Define how reading-flow interact with focusable display: contents elements.

[csswg-drafts issue 7387](https://github.com/w3c/csswg-drafts/issues/7387) Providing authors with a method of opting into following the visual order, rather than logical order

[csswg-drafts issue 9921](https://github.com/w3c/csswg-drafts/issues/9921) Is reading-order-items the best name for this property?

[csswg-drafts issue 9922](https://github.com/w3c/csswg-drafts/issues/9922) Should the reading-order-items property apply to tables in addition to flex and grid layouts?

[csswg-drafts issue 9923](https://github.com/w3c/csswg-drafts/issues/9923) Proposed alternative syntax for reading order

[csswg-drafts issue 8589](https://github.com/w3c/csswg-drafts/issues/8589) Do we need reading-order: &lt;integer> or should reading-order: auto be allowable in all grid or flex layouts?

[csswg-drafts issue 8257](https://github.com/w3c/csswg-drafts/pull/8257) Define 'reading-order: auto'
