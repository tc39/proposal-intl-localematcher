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

## Champion

- [Long Ho (@longlho)](https://github.com/longlho)

## API

```ts
interface Options {
    algorithm: 'lookup' | 'best fit'
}

Intl.LocaleMatcher.match(
    requestedLocales: string[],
    availableLocales: string[],
    defaultLocale: string,
    options?: Options
): string
```

### Options

1. `lookup` would continue to be the existing `LookupMatcher` implementation within ECMA-402.
1. `best fit` would be implementation-dependent.

## Examples

```ts
Intl.LocaleMatcher.match(["fr-XX", "en"], ["fr", "en"], "en"); // 'fr'
```

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
