---
layout: page_v2
title: Alloy RTD module
display_name: Alloy
description: The Alloy RTD module passes the contextual and audience segments of Alloy, the first-party data platform of Ringier Advertising, to the bidders you name.
page_type: module
module_type: rtd
module_code: alloyRtdProvider
enable_download: true
sidebarType: 1
vendor_specific: true
---

# Alloy RTD Provider

## Description

This module passes data of [Alloy](https://www.ringier-advertising.ch/de/hauseigene-datenloesung-alloy/), the first-party data platform of Ringier Advertising, to the bidders you name:

- the contextual segments of the current page,
- the audience segments of the user.

It also tracks a page view in Alloy, once per URL. It does not set ad server targeting.

Combine it with the [Alloy ID submodule](/dev-docs/modules/userid-submodules/alloy.html) to also pass the Alloy ID.

You need an app ID for your site. Please contact Ringier Advertising at [tech-accounts@ringier.ch](mailto:tech-accounts@ringier.ch) to get one.

## External script

**This module loads external JavaScript: the Alloy SDK, about 137 KB (33 KB compressed), from `cdn.alloy.ch`.** By default it loads the current 2.x release; set `params.sdk.scriptUrl` to pin a version or to load it from elsewhere.

If the Alloy SDK is on the page already, the module uses that one and loads nothing. You can deny the script with the `loadExternalScript` [activity control](/dev-docs/activity-controls.html); the module then provides no data unless the SDK is on the page.

## Integration

1. Include the module into your `Prebid.js` build.

    ```sh
    gulp build --modules='rtdModule,alloyRtdProvider,...'
    ```

1. Configure the module via `pbjs.setConfig`.

    ```js
    pbjs.setConfig({
      ...
      realTimeData: {
        auctionDelay: 300,
        dataProviders: [
          {
            name: 'alloy',
            waitForIt: true,
            params: {
              sdk: {
                appId: 'examplech_de_web', // provided by Ringier Advertising
              },
            },
          },
        ],
      },
    });
    ```

`auctionDelay` and `waitForIt` give the Alloy SDK time to load and answer before the auction starts. When it needs longer, the auction starts without the Alloy data.

## Parameters

{: .table .table-bordered .table-striped }

| Name | Required | Description | Type | Default value | Example |
| --- | --- | --- | --- | --- | --- |
| sdk | Yes | How to configure and load the Alloy SDK, see [Alloy SDK parameters](#alloy-sdk-parameters). | object | - | `{ appId: 'examplech_de_web' }` |
| bidders | No | Bidders that get the Alloy data. No other bidder gets it. | string[] | `['ringier']` | `['ringier']` |
| timeout | No | The maximum time to wait for the Alloy SDK to load and answer, in milliseconds. | number | 1000 | 1500 |
| audiences | No | Fetch the audience segments of the user. | boolean | true | false |
| trackPageViews | No | Track a page view once per URL. Ignored if the SDK tracks page views itself (`sdk.pageViewAutoTracking`). Set it to `false` if something else on the page tracks Alloy page views. | boolean | true | false |
| logicalPath | No | Logical path of the page for the page view. Without it the SDK uses `window.alloyDataLayer.context.logicalPath`, then the app ID. | string | - | `'examplech_de_web/desktop/sport'` |
| contentId | No | ID of the article in the CMS. Defaults to `window.alloyDataLayer.context.contentId`. | string \| number | - | `'article-4711'` |

On a single-page app, update `logicalPath` and `contentId` (or `window.alloyDataLayer.context`) before the first auction of each page. The module tracks a new page view when the URL has changed.

### Alloy SDK parameters

`params.sdk` is the same for this module and the Alloy ID submodule. If you use both, give both the same object. The SDK is configured and loaded once, by whichever module runs first.

{: .table .table-bordered .table-striped }

| Name | Required | Description | Type | Default value |
| --- | --- | --- | --- | --- |
| appId | Yes | ID of the site in Alloy, provided by Ringier Advertising. | string | - |
| tenant | No | Alloy tenant. | string | `'riad'` |
| env | No | Alloy environment: `'prod'`, `'stg'` or `'dev'`. | string | `'prod'` |
| scriptUrl | No | URL of the Alloy SDK. | string | `'https://cdn.alloy.ch/client-sdk/2.x.x/alloy-sdk.umd.js'` |
| consentCookieName | No | Name of the cookie holding the TCF consent string. | string | `'eupubconsent-v2'` |
| autoStart | No | Start the SDK as soon as it has loaded. If `false`, the page calls `alloySDK.start()`. | boolean | true |
| tracking | No | Track events. | boolean | true |
| adTracking | No | Track ad events. | boolean | true |
| pageViewAutoTracking | No | Let the SDK track page views on its own. | boolean | false |
| dataLayerAutoTracking | No | Let the SDK track `window.dataLayer` events. | boolean | false |
| visitorReadOnly | No | Only read the visitor ID, never create one. | boolean | false |
| skipEventsTracking | No | Skip event tracking. | boolean | false |
| ssoId | No | ID of the logged-in user. | string | - |
| externalIds | No | Other IDs of the user, by name, for Alloy to match on. | object | - |
| load | No | Set to `false` if the page loads the Alloy SDK itself. The module then only uses that SDK. | boolean | true |

The module leaves an Alloy SDK alone that is on the page already or that something else has started: it does not configure or load it a second time.

## Output

The module adds the data to the first-party data of each bidder in `params.bidders`, and of no other bidder:

```js
{
  site: {
    content: {
      data: [
        { name: 'alloy_iab', segment: [{ id: '0oujx' }] },
        { name: 'alloy_topics', segment: [{ id: '3kd9a' }] },
        { name: 'alloy_ctx_ready', segment: [{ id: '1' }] }
      ]
    }
  },
  user: {
    data: [
      { name: 'alloy_audiences', segment: [{ id: '8fj2k' }] }
    ]
  }
}
```

Each entry is named by the key-value key it stands for: the `ringier` bidder turns the entries into the key-values `alloy_iab`, `alloy_topics`, `alloy_ctx_ready` and `alloy_audiences` on its side. The segment IDs are Alloy's own. A list without segments is left out. `alloy_ctx_ready` is `1` when Alloy knows the page itself, and `0` when it only knows its section or did not answer.

## Consent

The module is registered with the GVL ID of Ringier AG (1436) and needs the [TCF consent management module](/dev-docs/modules/consentManagementTcf.html). It only loads and asks the Alloy SDK when the CMP has answered and, if GDPR applies, the user has given consent for purpose 1 and for Ringier AG. Without TCF consent data, for example when the CMP timed out, it does nothing.
