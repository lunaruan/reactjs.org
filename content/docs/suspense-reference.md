---
id: suspense-reference
title: Suspense API Reference
permalink: docs/suspense-reference.html
---

This page describes the Suspense APIs in React.

## Suspense Hooks {#suspense-hooks}

### `useTransition` {#use-transition}

```js
const [startTransition, isPending] = useTransition(CONFIG);
```

`useTransition` allows updates to be split up by immediate responses and responses that can suspend. It also optionally configures the amount of time React waits to show a loading state after a component suspends.

When an update happens, usually there is some response that needs to be shown immediately (ex. user feedback) and some response that might suspend (ex. data fetching).

`startTransition` is a function takes a callback. It should be used for any transition that might suspend to ensure that the transition does not stop more immediate actions from occuring.

```js
return (
    <button 
        onClick={() => {
            immediateResponse();
            startTransition(() => updateThatMightSuspend());
        }}
    }>
)
```

In this case, we can excecute `immediateResponse` right away and wait until later to execute `updateThatMightSuspend`.

Optionally, you can pass a config object to `useTransition` that tells React how long to wait before triggering the loading state.

```js
const CONIFG = {
    timeoutMs: 10000 // in milliseconds
};

function App() {
    const [startTransition, isPending] = useSuspenseTransition(CONFIG);
    const [page, setPage] = useState(0);
    return (
        <div>
            <Suspense fallback={<Glimmer />}>
                <PageDetails page={page}>
            </Suspense>
            <button 
                onClick={() => startTransition(() => setPage(page + 1))} 
                disabled={isPending}>
                Next Page
            </button>
        </div>
    );
}

```

In this case, React will wait up to 10 seconds before the `<Glimmer />`. 

During this time, the `isPending` flag becomes true. This can be used to indicate to the user that an action has been taken (in this case, that the button has been clicked and is disabled).

### `useDeferredValue` {#use-deferred-value}

```js
const deferredValue = useDeferredValue(value, CONFIG);
```

`useDeferredValue` returns a deferred value so old data and new data can be shown on separate parts of the page. 

A common case is a controlled text input.

```js
const CONFIG = {
  timeoutMs: 1000, // in milliseconds
};
const [text, setText] = useState(initialText);
const deferredText = useSuspenseDeferredValue(text, CONFIG);

return (
  <div>
    <input onChange={e => setText(e.target.value)} value={text} />
    <Suspense fallback={<Glimmer />}>
      <ResultList searchText={deferredText} />
    </Suspense>
  </div>
);
```

When you edit the text, `input` will immediate update. However, `ResultList` will still show the `deferredText` results.

Optionally, you can pass a config object to `useDeferredValue` that tells React how long to wait before showing the new text. In this case, React will wait at least 1 second before updating the `ResultList` with the new text.

**Note:** This pattern could potentially leave your code in an inconsistent state. This can be dangerous if the user thinks that the deferred old content is associated with the new content. A way to mitigate this risk is to have a shorter timeout.
