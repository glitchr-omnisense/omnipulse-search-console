# omnipulse/search-console

Formerly `omnireach/search-console` (renamed on 2026-11-10 with its family: `Omnireach\` is now `Omnipulse\`;
this package replaces the former name).

[omnisense/omnipulse](https://github.com/glitchr-omnisense/omnipulse)'s Google Search Console,
under any PHP host: a property's search queries, pages, countries and devices with their clicks,
impressions, CTR and average position, day by day or over a period - for a site taken over, the
history of its pages before and after.

## Install and a first call

```sh
composer require omnipulse/search-console
```

No account for this first call: Search Console's answer is recorded
(`Omnipulse\Testing\RecordedHttpClient` answers in place of the API), so nothing leaves the process.
A real call needs a Google Cloud service account added as a user of the Search Console property (its
JSON key as `credentials`), or an access token obtained elsewhere.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Omnipulse\Model\{Dimension, ReachQuery};
use Omnipulse\SearchConsole\SearchConsoleFactory;
use Omnipulse\Testing\RecordedHttpClient;

// Search Console's answer, recorded: nothing sent. HttpClient::create() (symfony/http-client) for a real call.
$http = RecordedHttpClient::routes([
    'POST /searchAnalytics/query' => '{"rows": [{"keys": ["atelier poterie strasbourg"], "clicks": 42, "impressions": 980, "ctr": 0.0429, "position": 3.62}]}',
]);
$console = (new SearchConsoleFactory($http))->create(['site' => 'sc-domain:atelier.example.org', 'access_token' => 'recorded']);

$report = $console->searchPerformance(new ReachQuery(new DateTimeImmutable('2026-09-24'), new DateTimeImmutable('2026-10-21'), [Dimension::Query]));
echo json_encode($report->rows[0]->dimensions + $report->rows[0]->metrics, JSON_UNESCAPED_SLASHES), "\n";   // {"query":"atelier poterie strasbourg","clicks":42,"impressions":980,"ctr":0.0429,"position":3.6}
```

With your credentials, through the family's registry:

```php
use Omnipulse\Registry;
use Omnipulse\SearchConsole\SearchConsoleFactory;
use Symfony\Component\HttpClient\HttpClient;

$registry = new Registry([new SearchConsoleFactory(HttpClient::create())], [
    'google' => ['factory' => 'search-console', 'options' => ['site' => 'sc-domain:example.org', 'credentials' => '/secrets/service-account.json']],
]);
```

Options: `site` (`sc-domain:example.org` or `https://example.org/`), `credentials` or
`access_token` (scope `webmasters.readonly`; the service account added as a user of the
property), `data_state` (`final`, or `all` for the provisional last days), `url`.

## The API

`POST sites/{site}/searchAnalytics/query` (<https://developers.google.com/webmaster-tools>):
`startDate`, `endDate`, `dimensions` (query, page, country, device, date - combined freely),
`dimensionFilterGroups` (`equals`), `rowLimit` (25,000 at most) and `startRow` for pages,
`dataState`. Countries come as ISO 3166-1 alpha-3 (`FRA`, upper-cased here), devices
lower-cased.

**Terms and limits** (<https://developers.google.com/webmaster-tools/terms>,
<https://developers.google.com/webmaster-tools/limits>): load quotas per site, per user and per
project - a query grouped by page or query costs more; 16 months of data; the top rows only
(anonymized queries are left out of a query breakdown).

**Google credentials**: a service account's JSON key (`credentials`: the JSON or the path of
its file) - the token is asked with a JWT signed by its key (RS256, the OAuth 2.0 JWT bearer
grant) and kept until a minute before it expires - or an `access_token` obtained elsewhere.
`GoogleToken` is the same class in omnipulse/search-console and omnipulse/ga4.

**Reuse**: Google's PHP libraries were set aside: `google/apiclient` with
`google/apiclient-services` (every Google API generated, a very large package, its own Guzzle
stack), `google/analytics-data` (gRPC and protobuf through google/gax), `google/auth` (Guzzle
again). The family injects the application's HTTP client and needs one JSON call per report and
one signed JWT, done here with ext-openssl.

## Usage policy

Search Console's usage policy is declared as data in [`policy.json`](policy.json) (omnisense/contracts'
schema, named in `composer.json`'s `extra.omnisense.policy`), its terms read on
2026-11-10 (https://developers.google.com/webmaster-tools/limits): Search Analytics: 1,200 queries a minute per site and per user, 40,000 a minute and 30,000,000 a day per project; rights unknown; OAuth credentials. omnipulse's registry enforces it - a call over a
limit is never made, a 429 and its `Retry-After` hold every next call
(`UsageLimitReachedException`) - and a site may tighten it, never loosen it
(omnisense/omnipulse's docs/policies.md).

## Verified, and not

- Verified (the core's harness, PHP 8.4): a query by query and country with two filters, the
  request's body and URL, the service account's token (asked once, kept; the JWT's claims, its
  signature checked with the key's public half - a key made by the test), credentials from JSON or
  a file, a refused property - against answers composed in the shape the reference documents, not
  recorded; every class loading with the package's requirements alone (`BareTest`).
- Not verified live: no Google account was used.

## Licence

MIT. An unofficial client, affiliated with Google in no way.
