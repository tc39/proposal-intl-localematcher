# `Intl.LocaleMatcher`

Prepare available locales once, then select translations, spellcheckers, or picker candidates.

**Stage 1.** Champion: [Long Ho](https://github.com/longlho).

[Specification](https://tc39.es/proposal-intl-localematcher/) · [Source](spec.emu) ·
[Research and open issues](DESIGN.md) ·
[FormatJS ponyfill](https://formatjs.github.io/docs/polyfills/intl-localematcher/)

## Proposed API

```js
const available = ['fr', 'en'];
const matcher = new Intl.LocaleMatcher(available, {algorithm: 'lookup'});

matcher.match(['fr-CA', 'en']); // {locale: 'fr', requestedLocale: 'fr-CA'}
matcher.match(['ja']); // undefined
const locale = matcher.match(navigator.languages)?.locale ?? 'en';
```

| Method | Result | Use case |
| --- | --- | --- |
| `match(requested)` | One result or `undefined` | App translation |
| `matchAll(requested)` | One winner per request, deduplicated | Multilingual spellchecking |
| `filter(requested)` | All acceptable available locales | Locale picker |

```js
const regional = new Intl.LocaleMatcher(['fr', 'en', 'fr-CA'], {algorithm: 'lookup'});
regional.matchAll(['fr-CA']);
// [{locale: 'fr-CA', requestedLocale: 'fr-CA'}]
regional.filter(['fr-CA']);
// [{locale: 'fr', requestedLocale: 'fr-CA'},
//  {locale: 'fr-CA', requestedLocale: 'fr-CA'}]
```

`matchAll` keeps the first result for each resource, in request order. It does not
search for alternatives when winners repeat. `filter` groups resources by their
earliest accepting request, preserving available order within each group. Neither
method defines a message-fallback chain; no matches returns `[]`.

Resource selection does not establish formatter support. Downstream operations
may need the original requests, not only the selected resource. Both multi-result
methods currently deduplicate resources, which can discard distinct accepted
requests such as `en-US` and `en-GB` matching `en`. Request-preserving output remains
an [open design question](DESIGN.md#chained-negotiation).

Lookup filtering tests prefix lookup against each candidate alone. Best fit uses
shared pairwise acceptance, independent of other candidates and request-priority
penalties. Filter results are not sorted by linguistic distance.

- Constructor snapshots and canonicalizes available locales; later mutations
  don't affect it. Options are read once and require an object or `undefined`.
- `lookup` uses RFC 4647 prefix lookup. `best fit` (default) reuses ECMA-402's
  implementation-defined matching, without requiring CLDR/ICU data or scores.
- `match` returns `{locale, requestedLocale}` or `undefined`. `locale` is a canonical
  available tag; `requestedLocale` is the canonical winning request, including its
  extensions. Requested `u` extensions do not affect matching or get appended to
  `locale`. No implicit default or formatting locale is supplied.
- Inputs follow `CanonicalizeLocaleList`: strings, Intl.Locale objects, and
  array-like lists. Generic iterables are not consumed. Available `u` extensions
  remain rejected provisionally; private-use tone matching remains unspecified.

Draft API changes: instance method replaces static `Intl.LocaleMatcher.match`;
an object separates resource selection from the winning user preference and leaves
room for future metadata.
The FormatJS ponyfill currently implements the older stateless API.

HTTP parsing, message fallback, weights, scores, and resource indices stay
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
