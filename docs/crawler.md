# Crawler configuration

DocSearch queries an Algolia index filled by the [Algolia Crawler](https://www.algolia.com/doc/tools/crawler/getting-started/overview/). When the website is accepted in the [DocSearch program](https://docsearch.algolia.com/apply/), Algolia creates a default crawler configuration: this page explains how to adapt it to a Cecil website, if needed.

The configuration is edited in the [Crawler dashboard](https://crawler.algolia.com/), in the _Editor_ tab of the crawler.

## Example

```js
new Crawler({
  appId: "<YOUR_APP_ID>",
  apiKey: "<YOUR_CRAWLER_API_KEY>",
  indexPrefix: "",
  rateLimit: 8,
  maxDepth: 10,
  maxUrls: 5000,
  renderJavaScript: false,
  startUrls: ["https://example.com/"],
  sitemaps: ["https://example.com/sitemap.xml"],
  discoveryPatterns: ["https://example.com/**"],
  exclusionPatterns: [],
  ignoreCanonicalTo: false,
  schedule: "every 1 day",
  safetyChecks: { beforeIndexPublishing: { maxLostRecordsPercentage: 30 } },
  actions: [
    {
      indexName: "<YOUR_INDEX_NAME>",
      pathsToMatch: ["https://example.com/**"],
      recordExtractor: ({ $, helpers }) => {
        return helpers.docsearch({
          recordProps: {
            lvl0: {
              selectors: "",
              defaultValue: "Documentation",
            },
            lvl1: ["main h1", "head > title"],
            lvl2: "main h2",
            lvl3: "main h3",
            lvl4: "main h4",
            lvl5: "main h5",
            lvl6: "main h6",
            content: "main p, main li",
          },
          aggregateContent: true,
          recordVersion: "v3",
        });
      },
    },
  ],
  initialIndexSettings: {
    "<YOUR_INDEX_NAME>": {
      attributesForFaceting: ["type", "lang"],
      attributesToRetrieve: ["hierarchy", "content", "anchor", "url", "url_without_anchor", "type"],
      attributesToHighlight: ["hierarchy", "content"],
      attributesToSnippet: ["content:10"],
      camelCaseAttributes: ["hierarchy", "content"],
      searchableAttributes: [
        "unordered(hierarchy.lvl0)",
        "unordered(hierarchy.lvl1)",
        "unordered(hierarchy.lvl2)",
        "unordered(hierarchy.lvl3)",
        "unordered(hierarchy.lvl4)",
        "unordered(hierarchy.lvl5)",
        "unordered(hierarchy.lvl6)",
        "content",
      ],
      distinct: true,
      attributeForDistinct: "url",
      customRanking: ["desc(weight.pageRank)", "desc(weight.level)", "asc(weight.position)"],
      ranking: ["words", "filters", "typo", "attribute", "proximity", "exact", "custom"],
      minWordSizefor1Typo: 3,
      minWordSizefor2Typos: 7,
      allowTyposOnNumericTokens: false,
      minProximity: 1,
      ignorePlurals: true,
      advancedSyntax: true,
      attributeCriteriaComputedByMinProximity: true,
      removeWordsIfNoResults: "allOptional",
    },
  },
});
```

Replace `https://example.com/` by the `baseurl` of the website, and `<YOUR_APP_ID>`, `<YOUR_CRAWLER_API_KEY>` and `<YOUR_INDEX_NAME>` by your own values.

> [!WARNING]
> The crawler uses an API key with **write** access: never use it in the `docsearch` section of the `config.yml`, which requires the **search-only** API key.

## Main options

### Start URLs and sitemap

Cecil generates a `sitemap.xml` file by default: add it in `sitemaps` so every page is discovered, even pages not linked from the menus. `startUrls` and `discoveryPatterns` should match the `baseurl` of the website.

Use `exclusionPatterns` to skip pages that shouldn't be indexed, for example tags pages or paginated lists:

```js
exclusionPatterns: ["https://example.com/tags/**", "https://example.com/**/page/**"],
```

Cecil generates static HTML: `renderJavaScript` can stay `false`, which makes the crawl faster.

### Selectors

The `recordProps` of `helpers.docsearch` define how records are extracted from the HTML of the pages. Adapt the CSS selectors to the templates of your theme:

- `lvl0`: the top level of the results (often the current section in the sidebar, for example `"#sidebar li > a.active"`), with a `defaultValue` when it is not found
- `lvl1` to `lvl6`: the headings of the page (`h1` to `h6`), scoped to the main content to ignore the header, the menus and the footer
- `content`: the text of the page (paragraphs, list items, etc.)

For example, with a theme where the content is rendered in an `<article>` element inside a `#content` container:

```js
recordProps: {
  lvl0: {
    selectors: "#sidebar li > a.active",
    defaultValue: "Documentation",
  },
  lvl1: "article h1",
  lvl2: "article h2",
  lvl3: "article h3",
  lvl4: "article h4",
  lvl5: "article h5",
  lvl6: "article h6",
  content: "#content p, #content li",
},
```

> [!TIP]
> Headings with an `id` attribute are used as anchors in the results URLs. Cecil adds an `id` to Markdown headings by default.

### Language

The theme filters the results on the current language with the `lang` facet (`facetFilters: ['lang:<language>']`). The crawler fills the `lang` attribute of the records with the `lang` attribute of the `<html>` element, which Cecil's layouts set to the language of the page (`<html lang="{{ site.language }}">`): keep it in your own layouts.

The `partials/docsearch/head.html.twig` template also adds a `docsearch:language` meta tag to every page, stored by the crawler as a `language` attribute.

For the filter to work, `lang` must be declared in `attributesForFaceting`:

```js
initialIndexSettings: {
  "<YOUR_INDEX_NAME>": {
    attributesForFaceting: ["type", "lang"],
    // ...
  },
},
```

If the index has no `lang` facet, disable the filter in the `config.yml`:

```yaml
docsearch:
  language: false
```

### Custom facets

Every `docsearch:<name>` meta tag of a page is stored by the crawler as a `<name>` attribute of its records. It can be used to filter the results, for example on the version of the documentation:

```twig
<meta name="docsearch:version" content="2">
```

```js
attributesForFaceting: ["type", "lang", "version"],
```

```yaml
docsearch:
  search_parameters:
    facetFilters: ['version:2']
```

> [!NOTE]
> `search_parameters.facetFilters` overrides the language filter: add `'lang:<language>'` to the list to keep it.

## Index settings

`initialIndexSettings` are only applied when the index is **created**. To change the settings of an existing index, edit them in the Algolia dashboard (_Search_ › _Index_ › _Configuration_), or delete the index and run the crawler again.

## Run the crawler

After saving the configuration, start a crawl with the _Restart crawling_ button of the dashboard. The `schedule` option runs the crawler periodically: a crawl can also be triggered after each deployment with the [Crawler API](https://www.algolia.com/doc/rest-api/crawler/#tag/actions/operation/startReindex).

Use the _URL Tester_ tab of the dashboard to check the records extracted from a page before running a full crawl.

## Resources

- [DocSearch: create a crawler](https://docsearch.algolia.com/docs/create-crawler)
- [DocSearch: record extractor](https://docsearch.algolia.com/docs/record-extractor)
- [DocSearch: meta tags](https://docsearch.algolia.com/docs/required-configuration/#introduce-global-information-as-meta-tags)
- [Algolia Crawler: configuration API](https://www.algolia.com/doc/tools/crawler/apis/configuration/)
