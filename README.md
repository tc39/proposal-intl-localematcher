# `Intl.LocaleMatcher`

## Motivation

i18n-supported websites often get a list of preferred locales via `Accept-Language` header or `navigator.languages`. They then try to determine the best available locale based on the set of locales that they support (and have translations for).

This operation currently exists within ECMA-402 but is only available as an abstract operation. Surfacing this functionality as a top level API would improve locale negotiation correctness and developer productivity as sites will be able to reliably handle not only matching, but also aliases, fallbacks and such.

### Use cases

1. Given a set of locales an application has translations for and the set of locales a user requests, [find the best matching locales](https://github.com/vercel/next.js/issues/18676).
2. JS runtimes (& polyfills) are not required to guarantee supporting all locales. Given a set of locales it supports and what the user requests, [find the best matching locales](https://github.com/formatjs/formatjs/discussions/2669). 
3. An application can also provide [different "tones"](https://github.com/formatjs/formatjs/discussions/2642) of the same locales (e.g casual, formal), utilizing `-x-` private tag. Given a set of locales with extensions and what the user preference might be, find the best matching locales.

## Status

**Stage 1**

Ponyfill: https://formatjs.io/docs/polyfills/intl-localematcher

Working specification: [spec.emu](spec.emu).
[Design notes and open questions](DESIGN.md) explain the draft semantics and
conformance plan. These choices have not been approved by the committee.

## Champion

- [Long Ho (@longlho)](https://github.com/longlho)

## API

```ts
interface Options {
    algorithm?: 'lookup' | 'best fit'
}

Intl.LocaleMatcher.match(
    requestedLocales: string[],
    availableLocales: string[],
    defaultLocale: string,
    options?: Options
): string
```

### Options

1. `lookup` uses ECMA-402's `LookupMatchingLocaleByPrefix` operation.
1. `best fit` is the default and uses `LookupMatchingLocaleByBestFit`. Its
   linguistic policy and data remain implementation-defined. No CLDR or ICU
   algorithm, data version, distance, or tie-breaking rule is required.

### Draft semantics

- Both lists use `CanonicalizeLocaleList`, including single strings,
  `Intl.Locale` objects, and array-like inputs. Generic iterables are not read.
- Available locales describe application resources, independently of the
  engine's supported formatter locales. No implicit parent resources are added.
- Available tags containing Unicode `u` extensions throw `RangeError` in this
  draft. Requested `u` extensions are ignored and are not copied to the result.
- The required fallback is validated eagerly and canonicalized. It may be
  outside the available list. It is used only if no match is found.
- Results are canonical strings, not original resource-key spellings.
- Options must be an object or `undefined`, following `GetOptionsObject`.

Extension-aware resource selection, multiple results, and API placement remain
open questions. Private-use tags are accepted, but this draft does not define
best-fit semantics for application-specific tones.

## Examples

```ts
Intl.LocaleMatcher.match(
  ["fr-CA", "en"], ["fr", "en"], "en", {algorithm: "lookup"}
); // 'fr'
```

`Accept-Language` header parsing, quality weights, and `*` language ranges are
outside this API. Parse them before passing a list of locale identifiers.

## Build the draft

```sh
npm install
npm run build
```

The strict build checks ecmarkup syntax, references, and algorithm lint rules.
It does not validate a JavaScript implementation.

## Prior Arts

### [@hapi/accept](https://github.com/hapijs/accept)

This is the core of hapijs header parsing with quality preferences. This however does a naive hierarchy with exact matches only. For example:

```js
Accept.language("en;q=0.7, fr-XX;q=0.8", ["fr", "en"]); // language === "en"
```

which would not be accurate.

### [koa](https://koajs.com/#request)

Similarly, Koa's `request.acceptsLanguages` follow similar exact match algorithm.

### [UTS35 LanguageMatching](https://www.unicode.org/reports/tr35/tr35.html#LanguageMatching)

This details a more sophisticated locale negotiation algorithm that is more accurate than `hapi`/`koa`

### [RFC4647 Section 3.4](https://tools.ietf.org/html/rfc4647#section-3.4)

This is the `lookup` algorithm in ECMA-402.

### [cldrjs's lookup implementation](https://github.com/rxaviers/cldrjs/blob/master/doc/bundle_lookup_matcher.md#implementation-details)

Similar to UTS35 LanguageMatching.

## References

- https://github.com/tc39/ecma402/issues/513
- https://github.com/tc39/ecma402/issues/46
- https://github.com/vercel/next.js/issues/18676
