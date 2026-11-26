---
layout: userid
title: Alloy ID
description: Alloy ID User ID sub-module
useridmodule: alloyIdSystem
bidRequestUserId: alloyId
eidsource: alloy.ch
example: '"1111"'
---

The Alloy ID is the visitor ID of [Alloy](https://www.ringier-advertising.ch/de/hauseigene-datenloesung-alloy/), the first-party data platform of Ringier Advertising. The submodule loads the Alloy SDK and passes the ID to bidders. Combine it with the [Alloy RTD module](/dev-docs/modules/alloyRtdProvider.html) to also pass Alloy's contextual and audience segments.

Add support for the Alloy ID to your Prebid.js package using:

```bash
gulp build --modules=userId,alloyIdSystem
```

## Alloy ID Registration

You need an app ID for your site. Please contact Ringier Advertising at [tech-accounts@ringier.ch](mailto:tech-accounts@ringier.ch) to get one.

## External script

**This submodule loads external JavaScript: the Alloy SDK, about 137 KB (33 KB compressed), from `cdn.alloy.ch`.** By default it loads the current 2.x release; set `params.sdk.scriptUrl` to pin a version or to load it from elsewhere.

If the Alloy SDK is on the page already, the submodule uses that one and loads nothing. You can deny the script with the `loadExternalScript` [activity control](/dev-docs/activity-controls.html); the submodule then provides no ID unless the SDK is on the page.

## Alloy ID Configuration

{: .table .table-bordered .table-striped }

| Param under userSync.userIds[] | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | Required | String | `"alloyId"` | `"alloyId"` |
| params | Required | Object | Details for the Alloy ID. | |
| params.sdk | Required | Object | How to configure and load the Alloy SDK, see [Alloy SDK parameters](#alloy-sdk-parameters). | |
| params.timeout | Optional | Number | Milliseconds the Alloy SDK may take to resolve the visitor ID. Defaults to `300`. | `500` |
| params.sdkTimeout | Optional | Number | Milliseconds to wait for the Alloy SDK to load and answer. After that the submodule provides no ID for this request; a later `refreshUserIds()` asks again. Defaults to `1000`. | `1500` |
| bidders | Optional | Array of strings | The bidders that get the ID. Without it, every bidder gets it. | `["ringier"]` |

Do not configure `storage`: the Alloy SDK keeps the ID itself. The submodule asks the SDK on every page view and on every `refreshUserIds()`, and Prebid.js stores nothing for it, even if `storage` is set.

The Alloy SDK is the source of truth. While the submodule waits for it, and when it provides no ID (no consent, an error, or no answer within `params.sdkTimeout`), an ID from an earlier request is no longer passed to bidders.

The submodule does not set a publisher provided ID (PPID) for your ad server.

### Alloy SDK parameters

`params.sdk` is the same for this submodule and the Alloy RTD module. If you use both, give both the same object. The SDK is configured and loaded once, by whichever module runs first.

{: .table .table-bordered .table-striped }

| Param under params.sdk | Scope | Type | Description | Default |
| --- | --- | --- | --- | --- |
| appId | Required | String | ID of the site in Alloy, provided by Ringier Advertising. | |
| tenant | Optional | String | Alloy tenant. | `"riad"` |
| env | Optional | String | Alloy environment: `"prod"`, `"stg"` or `"dev"`. | `"prod"` |
| scriptUrl | Optional | String | URL of the Alloy SDK. | `"https://cdn.alloy.ch/client-sdk/2.x.x/alloy-sdk.umd.js"` |
| consentCookieName | Optional | String | Name of the cookie holding the TCF consent string. | `"eupubconsent-v2"` |
| autoStart | Optional | Boolean | Start the SDK as soon as it has loaded. If `false`, the page calls `alloySDK.start()`. | `true` |
| tracking | Optional | Boolean | Track events. | `true` |
| adTracking | Optional | Boolean | Track ad events. | `true` |
| pageViewAutoTracking | Optional | Boolean | Let the SDK track page views on its own. | `false` |
| dataLayerAutoTracking | Optional | Boolean | Let the SDK track `window.dataLayer` events. | `false` |
| visitorReadOnly | Optional | Boolean | Only read the visitor ID, never create one. | `false` |
| skipEventsTracking | Optional | Boolean | Skip event tracking. | `false` |
| ssoId | Optional | String | ID of the logged-in user. | |
| externalIds | Optional | Object | Other IDs of the user, by name, for Alloy to match on. | |
| load | Optional | Boolean | Set to `false` if the page loads the Alloy SDK itself. The submodule then only uses that SDK. | `true` |

The submodule leaves an Alloy SDK alone that is on the page already or that something else has started: it does not configure or load it a second time.

## Consent

The submodule is registered with the GVL ID of Ringier AG (1436) and needs the [TCF consent management module](/dev-docs/modules/consentManagementTcf.html). It only loads and asks the Alloy SDK when the CMP has answered and, if GDPR applies, the user has given consent for purpose 1 and for Ringier AG. Without TCF consent data, for example when the CMP timed out, it does nothing.

## Alloy ID Example

```javascript
pbjs.setConfig({
    userSync: {
        auctionDelay: 300,
        userIds: [{
            name: 'alloyId',
            params: {
                sdk: {
                    appId: 'examplech_de_web' // provided by Ringier Advertising
                }
            },
            bidders: ['ringier']
        }]
    }
});
```

`auctionDelay` gives the Alloy SDK time to load and answer before the first auction starts. Without it, or when the SDK needs longer, the first auction starts without the ID.

This will add a `userId.alloyId` property to the bid requests of the named bidders, and this eid:

```javascript
{
    source: 'alloy.ch',
    uids: [{
        id: 'some-random-id-value',
        atype: 1
    }]
}
```
