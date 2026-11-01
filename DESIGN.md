# LocaleMatcher design

Working draft, not committee consensus. Proposed API:

```js
const matcher = new Intl.LocaleMatcher(availableLocales, {algorithm: 'best fit'});
const locale = matcher.match(requestedLocales) ?? defaultLocale;
```

## Decision

Prepare the supported set once; return one canonical tag or `undefined` per call.
Constructor options configure matching, not defaulting. This replaces the static
API and required fallback argument. It is not the current FormatJS ponyfill API.

```mermaid
flowchart TD
  available[Available locales and options] --> constructor[Validate and snapshot once]
  constructor --> matcher[Matcher instance]
  requests[Requested locales] --> match[Validate requests and match]
  matcher --> match
  match --> result[Canonical available tag or undefined]
  result --> caller[Caller applies fallback]
```

## Foundational precedents

| System | Relevant contract |
| --- | --- |
| [ICU](https://unicode-org.github.io/icu-docs/apidoc/released/icu4c/classicu_1_1LocaleMatcher_1_1Builder.html) | Prepared matcher; defaulting can be disabled. [Result](https://unicode-org.github.io/icu-docs/apidoc/released/icu4c/classicu_1_1LocaleMatcher_1_1Result.html) separates supported identity/indices from resolved formatting locale. |
| [Go x/text](https://pkg.go.dev/golang.org/x/text/language#Matcher) | Prepared matcher returns tag, supported index, confidence; returned tag can carry user extensions. |
| [Java Locale](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Locale.html) | Separate RFC lookup and filtering APIs; no match is null or an empty list. String lookup preserves supplied spelling. |
| [Fluent langneg](https://docs.rs/crate/fluent-langneg/0.14.2/source/src/negotiate/mod.rs) | Lookup, Matching, and Filtering have distinct cardinalities. Optional default; supplied available entries retain identity. |
| [Apple Foundation](https://developer.apple.com/documentation/foundation/bundle/1409418-preferredlocalizations) | Selected localization array may include a region and parent; explicitly not a complete preference ranking. |
| [Fuchsia](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/lib/intl/lookup/rust/src/lib.rs) | One ICU selection per request, then catalog search per message. Construction fails if nothing matches. |
| [GNU gettext](https://www.gnu.org/software/gettext/manual/html_node/The-LANGUAGE-variable.html) / [.NET ResourceManager](https://learn.microsoft.com/en-us/dotnet/core/extensions/package-and-deploy-resources#the-resource-fallback-process) | Ordered language/dialect or culture-parent fallback per message. Separate from choosing one locale for a session. |

Takeaways: preparation has established purpose; no-match need not require a fake
fallback; language selection, formatting preferences, and missing-message fallback
have different contracts. Multiple results do not imply a universal ordering.

These systems do not establish one common scoring formula. Keep best fit
implementation-defined; the earlier weighted-distance sketch is not adopted.
No new CLDR data source, version, thresholds, or numeric scores are required.

## Observable semantics

- Constructor requires `new`. Resolve `newTarget.prototype`, canonicalize complete
  available input, reject `u` extensions, then read options and `algorithm`.
- `CanonicalizeLocaleList` copies, canonicalizes, and deduplicates available tags.
  Later array, getter, or option changes do not alter this private snapshot.
- Each `match` checks receiver branding before reading the complete requested
  list. Empty lists or no match return `undefined`; no host default is read.
- Reuse existing prefix/best-fit operations on the whole request list. No separate
  exact-match prepass, synthetic parent resources, or requested-extension copying.
- Other extensions/private-use tags retain existing operation semantics; no tone
  policy is invented. Rejection of available `u` extensions remains provisional.

Preparation/indexing is an implementation choice, without timing guarantees.
Request getters and coercion errors remain observable on every call.

## Resource identity and open issues

Canonical output can differ from resource-key spelling (`iw` versus `he`). This
draft keeps string output; callers must map canonical tags to resources. ICU/Go
indices and Fluent references motivate a possible richer result, not an added API.

| Issue | Proposed position / remaining work |
| --- | --- |
| [#2: Placement](https://github.com/tc39/proposal-intl-localematcher/issues/2) | Constructor now owns a reusable snapshot; no namespace-only justification. API naming still reviewable. |
| [#3: Use cases](https://github.com/tc39/proposal-intl-localematcher/issues/3) | Translation-resource selection. Cannot infer unlisted languages or translation completeness; HTTP parsing remains separate. |
| [#4: Return type](https://github.com/tc39/proposal-intl-localematcher/issues/4) | Canonical String or undefined. Accept Intl.Locale inputs. Resource indices/objects remain open. |
| [#6: Extensions](https://github.com/tc39/proposal-intl-localematcher/issues/6) | Keep requested `u` preferences separate from resource identity; defer private-use tone negotiation. |
| [#8: Multiple results](https://github.com/tc39/proposal-intl-localematcher/issues/8) | Explicit undefined removes sentinel-default tricks. One winner per request, all acceptable resources, and message-fallback chains still need separate designs. |

No issues are closed. `algorithm` versus `localeMatcher`, stronger exact-match
invariants, and available-extension policy remain open.

## Validation plan

Strict spec builds validate markup and references, not runtime behavior. Planned
portable tests cover constructor/newTarget order, subclassing, descriptors,
prototype and cross-realm branding, detached/proxied receivers, snapshot mutation
and getters, option reads, per-call request validation, lookup truncation,
canonical output, extensions, empty inputs, and explicit no-match.

Best-fit tests must allow implementation-defined answers. ICU oracle fixtures
belong in implementation suites. No engine implementation is added here.
