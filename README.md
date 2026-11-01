# `Intl.LocaleMatcher`

Select an application's translation locale from user preferences. Exposes locale
matching already used internally by ECMA-402 formatters.

**Stage 1.** Champion: [Long Ho](https://github.com/longlho).

[Working spec](spec.emu) · [Design and open issues](DESIGN.md) ·
[FormatJS ponyfill](https://formatjs.io/docs/polyfills/intl-localematcher)

## API

```js
Intl.LocaleMatcher.match(
  ['fr-CA', 'en'],           // User preferences
  ['fr', 'en'],              // Available translations
  'en',                     // Fallback
  {algorithm: 'lookup'}
); // 'fr'
```

- `lookup`: ECMA-402's `LookupMatchingLocaleByPrefix` (RFC 4647).
- `best fit` (default): `LookupMatchingLocaleByBestFit`, with implementation-defined
  linguistic policy and data. No required CLDR or ICU algorithm.

## Draft contract

- Both lists use `CanonicalizeLocaleList`: strings, `Intl.Locale` objects, and
  array-like inputs. Generic iterables are not consumed.
- Available tags describe application resources, independently of engine support.
  No parent resources are added.
- Return a canonical available tag, or the eagerly validated canonical fallback.
  Fallback may be outside the available list; it does not influence matching.
- Ignore requested Unicode `u` extensions; reject them on available tags.
  Private-use tags are accepted without standardized tone-matching semantics.
- Options must be an object or `undefined`. HTTP headers, quality weights, and
  wildcard ranges must be handled by the caller.

These are draft choices, not committee consensus. API placement, extension
handling, fallback policy, and multiple results remain open. The
[candidate selection algorithm](DESIGN.md#candidate-best-fit-selection-algorithm)
is a discussion option; normative best fit remains implementation-defined.

## Build

```sh
npm ci
npm run build
```

Strict ecmarkup build checks syntax, references, and algorithm lint rules.
JavaScript conformance tests are planned, not implemented here.

## References

- [ECMA-402 negotiation](https://tc39.es/ecma402/#sec-locale-and-parameter-negotiation)
- [UTS 35 language matching](https://www.unicode.org/reports/tr35/#LanguageMatching)
- [RFC 4647 lookup](https://www.rfc-editor.org/rfc/rfc4647#section-3.4)
- [Original discussion](https://github.com/tc39/ecma402/issues/513)
- [Next.js use case](https://github.com/vercel/next.js/issues/18676)
