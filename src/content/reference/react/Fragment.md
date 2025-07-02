---
title: <Fragment> (<>...</>)
---

<Intro>

`<Fragment>`, often used via `<>...</>` syntax, lets you group elements without a wrapper node. 

<ExperimentalBadge /> Fragments can also accept refs, which enable interacting with underlying DOM nodes without adding wrapper elements.

```js
<>
  <OneChild />
  <AnotherChild />
</>
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `<Fragment>` {/*fragment*/}

Wrap elements in `<Fragment>` to group them together in situations where you need a single element. Grouping elements in `Fragment` has no effect on the resulting DOM; it is the same as if the elements were not grouped. The empty JSX tag `<></>` is shorthand for `<Fragment></Fragment>` in most cases.

<ExperimentalBadge /> Fragments can accept refs to provide access to the underlying DOM nodes they wrap, enabling interactions like event handling, intersection observation, and focus management without requiring additional wrapper elements.

#### Props {/*props*/}

- **optional** `key`: Fragments declared with the explicit `<Fragment>` syntax may have [keys.](/learn/rendering-lists#keeping-list-items-in-order-with-key)
- <ExperimentalBadge />  **optional** `ref`: A ref object or callback function. React will forward the ref to a `FragmentInstance` object that provides methods for interacting with the DOM nodes wrapped by the Fragment.

<Experimental>
#### FragmentInstance {/*fragmentinstance*/}

When you pass a ref to a fragment, React provides a `FragmentInstance` object with methods for interacting with the DOM nodes wrapped by the fragment:

**Event handling methods:**
- `addEventListener(type, listener, options?)`: Adds an event listener to all first-level DOM children of the fragment
- `removeEventListener(type, listener, options?)`: Removes an event listener from all first-level DOM children of the fragment  
- `dispatchEvent(event)`: Dispatches an event to a virtual child of the Fragment to call any added listeners and bubble to the DOM parent.

**Layout methods:**
- `compareDocumentPosition(otherNode)`: Compares the document position of the fragment with another node
  - An element preceding a fragment is `Node.DOCUMENT_POSITION_PRECEDING`
  - An element following a fragment is `Node.DOCUMENT_POSITION_FOLLOWING`
  - An element containing the fragment is `Node.DOCUMENT_POSITION_PRECEDING` and `Node.DOCUMENT_POSITION_CONTAINING`
  - An element within the fragment is `NODE.DOCUMENT_POSITION_CONTAINED_BY`
  - Empty Fragments will attempt to compare positioning within the React tree and include `NODE.DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC`
  - Elements that have a different relationship in the React tree and DOM tree due to portaling or other insertions are `NODE.DOCUMENT_POSITION_IMPLEMENTATION_SPECIFIC`
- `getClientRects()`: Returns a flat array of `ClientRect` objects representing the bounding rectangles of all DOM nodes
- `getRootNode()`: Returns the root node containing the fragment's DOM nodes

**Focus management methods:**
- `focus(options?)`: Focuses the first focusable DOM node in the Fragment
- `focusLast(options?)`: Focuses the last focusable DOM node in the Fragment
- `blur()`: Removes focus from all DOM nodes in the Fragment

**Observer methods:**
- `observeUsing(observer)`: Starts observing the fragment's DOM children with an IntersectionObserver or ResizeObserver
- `unobserveUsing(observer)`: Stops observing the fragment's DOM children with the specified observer
</Experimental>

#### Caveats {/*caveats*/}

- If you want to pass `key` to a Fragment, you can't use the `<>...</>` syntax. You have to explicitly import `Fragment` from `'react'` and render `<Fragment key={yourKey}>...</Fragment>`.

- React does not [reset state](/learn/preserving-and-resetting-state) when you go from rendering `<><Child /></>` to `[<Child />]` or back, or when you go from rendering `<><Child /></>` to `<Child />` and back. This only works a single level deep: for example, going from `<><><Child /></></>` to `<Child />` resets the state. See the precise semantics [here.](https://gist.github.com/clemmy/b3ef00f9507909429d8aa0d3ee4f986b)

- <ExperimentalBadge /> If you want to pass `ref` to a Fragment, you can't use the `<>...</>` syntax. You have to explicitly import `Fragment` from `'react'` and render `<Fragment ref={yourRef}>...</Fragment>`.

---

## Usage {/*usage*/}

### Returning multiple elements {/*returning-multiple-elements*/}

Use `Fragment`, or the equivalent `<>...</>` syntax, to group multiple elements together. You can use it to put multiple elements in any place where a single element can go. For example, a component can only return one element, but by using a Fragment you can group multiple elements together and then return them as a group:

```js {3,6}
function Post() {
  return (
    <>
      <PostTitle />
      <PostBody />
    </>
  );
}
```

Fragments are useful because grouping elements with a Fragment has no effect on layout or styles, unlike if you wrapped the elements in another container like a DOM element. If you inspect this example with the browser tools, you'll see that all `<h1>` and `<article>` DOM nodes appear as siblings without wrappers around them:

<Sandpack>

```js
export default function Blog() {
  return (
    <>
      <Post title="An update" body="It's been a while since I posted..." />
      <Post title="My new blog" body="I am starting a new blog!" />
    </>
  )
}

function Post({ title, body }) {
  return (
    <>
      <PostTitle title={title} />
      <PostBody body={body} />
    </>
  );
}

function PostTitle({ title }) {
  return <h1>{title}</h1>
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```

</Sandpack>

<DeepDive>

#### How to write a Fragment without the special syntax? {/*how-to-write-a-fragment-without-the-special-syntax*/}

The example above is equivalent to importing `Fragment` from React:

```js {1,5,8}
import { Fragment } from 'react';

function Post() {
  return (
    <Fragment>
      <PostTitle />
      <PostBody />
    </Fragment>
  );
}
```

Usually you won't need this unless you need to [pass a `key` to your `Fragment`.](#rendering-a-list-of-fragments)

</DeepDive>

---

### Assigning multiple elements to a variable {/*assigning-multiple-elements-to-a-variable*/}

Like any other element, you can assign Fragment elements to variables, pass them as props, and so on:

```js
function CloseDialog() {
  const buttons = (
    <>
      <OKButton />
      <CancelButton />
    </>
  );
  return (
    <AlertDialog buttons={buttons}>
      Are you sure you want to leave this page?
    </AlertDialog>
  );
}
```

---

### Grouping elements with text {/*grouping-elements-with-text*/}

You can use `Fragment` to group text together with components:

```js
function DateRangePicker({ start, end }) {
  return (
    <>
      From
      <DatePicker date={start} />
      to
      <DatePicker date={end} />
    </>
  );
}
```

---

### Rendering a list of Fragments {/*rendering-a-list-of-fragments*/}

Here's a situation where you need to write `Fragment` explicitly instead of using the `<></>` syntax. When you [render multiple elements in a loop](/learn/rendering-lists), you need to assign a `key` to each element. If the elements within the loop are Fragments, you need to use the normal JSX element syntax in order to provide the `key` attribute:

```js {3,6}
function Blog() {
  return posts.map(post =>
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  );
}
```

You can inspect the DOM to verify that there are no wrapper elements around the Fragment children:

<Sandpack>

```js
import { Fragment } from 'react';

const posts = [
  { id: 1, title: 'An update', body: "It's been a while since I posted..." },
  { id: 2, title: 'My new blog', body: 'I am starting a new blog!' }
];

export default function Blog() {
  return posts.map(post =>
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  );
}

function PostTitle({ title }) {
  return <h1>{title}</h1>
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```

</Sandpack>

<Experimental>
---

### Using Fragment refs for DOM interaction {/*using-fragment-refs-for-dom-interaction*/}

Fragment refs allow you to interact with the DOM nodes wrapped by a Fragment without adding extra wrapper elements. This is useful for event handling, visibility tracking, focus management, and replacing deprecated patterns like `ReactDOM.findDOMNode()`.

```js {11-12,16-18}
import { Fragment, useRef, useEffect } from 'react';

function ClickableWrapper({ children, onClick }) {
  const fragmentRef = useRef(null);

  useEffect(() => {
    const handleClick = (event) => {
      onClick(event);
    };
    
    fragmentRef.current.addEventListener('click', handleClick);
    return () => fragmentRef.current.removeEventListener('click', handleClick);
  }, [onClick]);

  return (
    <Fragment ref={fragmentRef}>
      {children}
    </Fragment>
  );
}
```
---

### Tracking visibility with Fragment refs {/*tracking-visibility-with-fragment-refs*/}

Fragment refs are particularly useful for visibility tracking and intersection observation. This enables you to monitor when content becomes visible without requiring the child components to expose refs:

```js {19,21,31-34}
import { Fragment, useRef, useEffect } from 'react';

function VisibilityObserver({ threshold = 0.5, onVisibilityChange, children }) {
  const fragmentRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        onVisibilityChange(entries[0].isIntersecting);
      },
      { threshold }
    );
    
    fragmentRef.current.observeUsing(observer);
    return () => fragmentRef.current.unobserveUsing(observer);
  }, [threshold, onVisibilityChange]);

  return (
    <Fragment ref={fragmentRef}>
      {children}
    </Fragment>
  );
}

function MyComponent() {
  const handleVisibilityChange = (isVisible) => {
    console.log('Component is', isVisible ? 'visible' : 'hidden');
  };

  return (
    <VisibilityObserver onVisibilityChange={handleVisibilityChange}>
      <SomeThirdPartyComponent />
      <AnotherComponent />
    </VisibilityObserver>
  );
}
```

This pattern replaces effect-based visibility logging, which can be unreliable due to component lifecycle issues and Activity unmounting/remounting.

---

### Focus management with Fragment refs {/*focus-management-with-fragment-refs*/}

Fragment refs provide focus management methods that work across all DOM nodes within the Fragment:

```js
import { Fragment, useRef } from 'react';

function FocusableGroup({ children }) {
  const fragmentRef = useRef(null);

  return (
    <>
      <button onClick={() => {fragmentRef.current.focus()}}>Focus</button>
      <Fragment ref={fragmentRef}>
        {children}
      </Fragment>
    </>
  );
}
```

The `focus()` method focuses the first focusable element within the Fragment, while `focusLast()` focuses the last focusable element.

</Experimental>
