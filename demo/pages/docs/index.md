---
title: Documentation
---
Add [Algolia DocSearch](https://docsearch.algolia.com) to a Cecil website.

## Installation

```bash
composer require cecil/theme-docsearch
```

## Configuration

```yaml
theme:
  - docsearch
docsearch:
  appId: <YOUR_APP_ID>
  apiKey: <YOUR_SEARCH_API_KEY>
  indexName: <YOUR_INDEX_NAME>
```

## Templates

```twig
{# in <head> #}
{{ include('partials/docsearch/head.html.twig') }}
{# where the search button should be displayed #}
{{ include('partials/docsearch.html.twig') }}
```
