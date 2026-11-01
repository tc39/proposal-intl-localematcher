# `Intl.LocaleMatcher`

Prepare available translation locales once, then match user preferences.

**Stage 1.** Champion: [Long Ho](https://github.com/longlho).

[Specification](https://tc39.es/proposal-intl-localematcher/) · [Source](spec.emu) ·
[Research and open issues](DESIGN.md) ·
[FormatJS ponyfill](https://formatjs.github.io/docs/polyfills/intl-localematcher/)

## Proposed API

```js
const available = ['fr', 'en'];
const matcher = new Intl.LocaleMatcher(available, {algorithm: 'lookup'});

matcher.match(['fr-CA', 'en']); // 'fr'
matcher.match(['ja']);         // undefined
const locale = matcher.match(navigator.languages) ?? 'en';
```

- Constructor snapshots and canonicalizes available locales; later mutations
  don't affect it. Options are read once and require an object or `undefined`.
- `lookup` uses RFC 4647 prefix lookup. `best fit` (default) reuses ECMA-402's
  implementation-defined matching, without requiring CLDR/ICU data or scores.
- `match` returns one canonical available tag or `undefined`. No implicit default
  or requested Unicode `u` extensions are added to the result.
- Inputs follow `CanonicalizeLocaleList`: strings, Intl.Locale objects, and
  array-like lists. Generic iterables are not consumed. Available `u` extensions
  remain rejected; private-use tone matching remains unspecified.

Draft API change: instance method replaces static `Intl.LocaleMatcher.match`.
The FormatJS ponyfill currently implements the older stateless API.

HTTP parsing, message fallback, multiple results, and resource indices stay
outside this draft. Open decisions and foundational precedents are in
[design notes](DESIGN.md).

## Build

```sh
npm ci
npm run build
```

Strict ecmarkup build checks syntax, references, and algorithm lint rules.
Runtime conformance tests are planned, not implemented here.

## References

- [ECMA-402 negotiation](https://tc39.es/ecma402/#locale-and-parameter-negotiation)
- [RFC 4647 lookup](https://www.rfc-editor.org/rfc/rfc4647#section-3.4)
- [Original discussion](https://github.com/tc39/ecma402/issues/513)
