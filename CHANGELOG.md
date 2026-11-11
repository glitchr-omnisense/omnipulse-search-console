# Changelog

## Unreleased (1.x)

### Changed
- Token requests counted: the service account's token request (oauth2.googleapis.com) goes through the provider's guard, checked before it is sent (a
  site's limit naming `token`, the provider's waits) and counted after, in the registry's limiter.
  `SearchConsoleFactory` gives `GoogleToken` the provider's guard (`Config::guard()`, its `guard:` argument): each token request is counted under `token`; Search Console's declared limits name `searchPerformance` and do not count it. Needs omnisense/contracts 16c8da1 or later (`Config::guard()`; 1568f2c, what the sites'
  vendors hold, has it). Tests/SearchConsoleProviderTest: a site's limit of 0 tokens a day, the token request never sent.

### Added
- Search Console's usage policy declared as data: `policy.json` (omnisense/contracts' schema), its terms
  read on 2026-11-10, named in `composer.json` (`extra.omnisense.policy`, and
  `omnisense/contracts` required). The provider implements `DeclaresUsagePolicyInterface` and gives
  the policy to its Capabilities; the factory hands the provider's Backoff to its HTTP calls, so
  that a 429 and its `Retry-After` hold every next call. `Tests/PolicyTest.php`.

### Changed
- `omnireach/search-console` is now `omnipulse/search-console` (2026-11-10: the omnireach family is renamed omnipulse,
  this repository's history kept): the namespace `Omnipulse\SearchConsole`, requiring `omnisense/omnipulse`;
  `composer.json` replaces `omnireach/search-console`, so that a project never installs both.

