# yee-there

> ExtendableEvent for DOM elements. *"Are we there yet."*

Service workers have [`ExtendableEvent`](https://developer.mozilla.org/en-US/docs/Web/API/ExtendableEvent) — an event whose lifetime is extended by `waitUntil(promise)` until associated async work settles. yee-there brings the same shape to general DOM elements: an element registers its own completion as its own event, resolving its own hold.

## Why

yee's appliers (and most custom elements) do async work: fetch handlers, network setup, decoder init, subscription establishment. Today that async work is invisible to the page — there's no general way to ask "is this element done arriving?" or "wait for everything on this page to be ready."

yee-there fills that gap. An element emits a `yee-there` event when it has a hold to register; the event's `waitUntil(promise)` extends the element's "in flight" status until the promise settles. Aggregators (yee-mix, the document, anything) can subscribe to wait for holds to clear.

## The shape

```ts
class YeeThereEvent extends ExtendableEvent {
  static readonly type = 'yee-there'
  constructor(init?: EventInit) { super('yee-there', init) }
}

interface YeeThereable extends EventTarget {
  // Element emits yee-there when it has an in-flight hold.
  // waitUntil() extends the hold until promises resolve.
}

// An aggregator:
class YeeThereGate extends EventTarget {
  // Resolves when all observed holds on the matched subtree have cleared.
  // Re-opens when a new yee-there event fires.
  ready(): Promise<void>
}
```

An element starts a hold:

```ts
this.dispatchEvent(new YeeThereEvent())
const evt = ... // grab a reference before dispatch
evt.waitUntil(asyncWork())
```

A page or mix host waits:

```html
<yee-there-gate observe="body">
  <!-- stamps a row only when the subtree is 'there' -->
</yee-there-gate>
```

## Where it lands in the yee family

This is the **observability layer for async element lifecycle**. It's what the moqy, cast-yee, and command-auto-yee speculations kept reaching for:

- moqy `<moq-subscribe>` is "there" when its first object arrives.
- cast-yee is "there" when its Presentation connection reaches `connected`.
- command-auto-yee is "there" when its wiring has bound commander to commandable.
- A `<yee-listen>` with an async handler is "there" when its handler resolves.

yee-there makes these observable uniformly. yee-mix's processing can be gated on yee-there events from its appliers, giving the transactional apply semantics yee v2 needs.

## Mental model

Lifted from [`mpsc-completion`](https://github.com/rektide/mpsc-completion) — a multi-producer single-completion channel. Each element with in-flight work is a producer; the gate is the single consumer that completes when all producers settle. yee-there is that pattern as a DOM event.

## Status

Baseline. README + design contract only — implementation and tickets under beads prefix `ythere`.

## Related

- [`yee`](https://github.com/rektide/yee) — primary consumer; will gate mix processing on yee-there events.
- [`yee-goeth`](https://github.com/rektide/yee-goeth) — the symmetric contract for *teardown* (graceful departure).
- [`mpsc-completion`](https://github.com/rektide/mpsc-completion) — the underlying mental model.

## License

MIT
