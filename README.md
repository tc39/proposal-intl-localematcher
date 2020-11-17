# `Intl.bestAvailableLocale`
## Overview

### Motivation

i18n-supported websites often get a list of preferred locales via `Accept-Language` header or `navigator.languages`. They then try to determine the best available locale based on the set of locales that they support (and have translations for).

This operation currently exists within ECMA-402 but is only available as an abstract operation. Surfacing this functionality as a top level API would improve locale negotiation correctness and developer productivity as sites will be able to reliably handle not only matching, but also aliases, fallbacks and such.

### API

```ts
interface Options {
    localeMatcher: 'lookup' | 'best fit'
}

Intl.bestAvailableLocale(
    requestedLocales: string[], 
    availableLocales: string[], 
    defaultLocale: string,
    options: Options
): string
```

#### Options

1. `lookup` would continue to be the existing `LookupMatcher` implementation within ECMA-402.
1. `best fit` would be implementation-dependent.

### Examples

```ts
Intl.bestAvailableLocale(['fr-XX', 'en'], ['fr', 'en'], 'en') // 'fr'
```

## Prior Arts

### [@hapi/accept](https://github.com/hapijs/accept)

This is the core of hapijs header parsing with quality preferences. This however does a naive hierarchy with exact matches only. For example:

```js
Accept.language("en;q=0.7, fr-XX;q=0.8", ['fr', 'en']); // language === "fr"
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