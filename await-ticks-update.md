# Async/await Ticks Update

## Review

- Awaiting a promise requires 3 jobs (ticks), whereas we intuitively expect it (and would like for performance reasons) to require only 1 job.
- Change involves using PromiseResolve instead of creating a new Promise to wrap the `await` operand.
- Significant performance gains in V8 from implementing this change.
- ChakraCore (mostly) already implements these semantics.
- Babel transformation uses ChakraCore semantics.
- Semantic tradeoff: for built-in, non-subclass promises with a monkey-patched "then", the "then" method will not be called.

## Where did we leave off last time?

- Concern was raised by Mark Miller regarding a monkey-patching use case: Override "then" so that it returns frozen promise objects. Does this still work?

## Update

- Analysis of monkey-patching use cases:
  1. **Modifying the returned promise**: (e.g. frozen promises): The non-monkey-patched promise resulting from `PerformPromiseThen` is never seen by user code.
  1. **Wrap the input callbacks**: (e.g. async "zone" tracking): For `await`, "then" only receives "resolve" and "reject", which cannot invoke user code.
- "Frozen promise" use case still works, because the promise resulting from `PerformPromiseThen` is never visible to user code.
- We believe this change makes the right tradeoffs and does not significantly impact known monkey-patching use cases.

## Conclusion

- We would like to achieve consensus on making this change.
- We would like to make this change soon, while Babel transformations are still widely used.
