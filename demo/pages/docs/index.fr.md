---
title: Documentation
---
Ajouter [Algolia DocSearch](https://docsearch.algolia.com) à un site Cecil.

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
{# dans <head> #}
{{ include('partials/docsearch/head.html.twig') }}
{# là où le bouton de recherche doit être affiché #}
{{ include('partials/docsearch.html.twig') }}
```
