---
layout: bidder
title: Ringier
description: Prebid Ringier Bidder Adapter
biddercode: ringier
media_types: video, native
pbjs: true
pbs: false
prebid_member: false
gvl_id: 1436
tcfeu_supported: true
gpp_sids: tcfeu
usp_supported: false
coppa_supported: false
schain_supported: true
dchain_supported: false
userIds: all
safeframes_ok: true
deals_supported: true
floors_supported: true
fpd_supported: true
ortb_blocking_supported: true
dsa_supported: true
multiformat_supported: will-bid-on-one
sidebarType: 1
---

## Note

The Ringier adapter connects publishers to the Ringier Advertising Bidlayer, a server-side auction built on Prebid Server. Bidders, bidder parameters and rules are configured on the server for each `publisherId` and `placementId`. Please contact Ringier Advertising to get your IDs.

All bids are returned under the bidder code `ringier`. The demand source of a bid is available as an opaque identifier in `bid.meta.demandSource`.

## Bid Params

{: .table .table-bordered .table-striped }
| Name          | Scope    | Description                                   | Example         | Type     |
| ------------- | -------- | --------------------------------------------- | --------------- | -------- |
| `publisherId` | required | Publisher ID provided by Ringier Advertising. | `'pub-123'`     | `string` |
| `placementId` | required | Placement ID provided by Ringier Advertising. | `'example-top'` | `string` |

Sizes, floors, consent, user IDs and first party data are taken from the ad unit and the Prebid.js configuration.

## Configuration

Optional settings via `pbjs.setConfig({ ringier: { ... } })`:

{: .table .table-bordered .table-striped }
| Name            | Description                                                                                                     | Type                 |
| --------------- | --------------------------------------------------------------------------------------------------------------- | -------------------- |
| `clientBidders` | Bidders the publisher calls client-side on the page. The Bidlayer does not call them a second time server-side. | `string[]`           |
| `configVersion` | Version of the publisher setup the page was built with.                                                         | `string` or `number` |
| `pvId`          | Page view ID of the publisher. Defaults to the page view ID that Prebid.js assigns to this bidder.              | `string`             |
| `sendEvents`    | `false` switches off the event pixels (win, render, timeout, error). Default: `true`.                           | `boolean`            |
| `endpoint`      | Overrides the auction endpoint for testing. Only https URLs on `ringier-advertising.ch` are accepted.           | `string`             |

```javascript
pbjs.setConfig({
  ringier: {
    clientBidders: ['exampleBidder'],
    configVersion: '2026-11-05.1'
  }
});
```

## Multiformat

For ad units with several media types the Bidlayer bids with one of them: banner, then video, then native, as far as the placement allows. For outstream video the publisher provides the renderer.

## Deals

Deal IDs carry the demand source as a prefix (`<demand source>:<deal ID>`), so line items for deals have to target this value.

## First Party Data

The adapter sends the OpenRTB first party data of the request as it is: `ortb2.site.*`, `ortb2.user.*` and the ad unit's `ortb2Imp.ext.*`. User IDs are sent as `user.ext.eids`.

## ORTB Blocking

The following blocking parameters are passed on to the bidders of the Bidlayer: `bcat`, `badv`, `battr` and `bapp` of the request, and `battr` of `imp.banner` and `imp.video`.

## User Sync

With iframe syncs allowed, the adapter registers one iframe from the Bidlayer domain. It syncs the bidders configured for the publisher. No sync takes place without TCF purpose 1 consent.

```javascript
pbjs.setConfig({
  userSync: {
    filterSettings: {
      iframe: { bidders: ['ringier'], filter: 'include' }
    }
  }
});
```

## Disclosures

- The adapter records win, render, timeout and error events with a keepalive request to the Bidlayer domain. Publishers can switch them off with `setConfig({ ringier: { sendEvents: false } })`.
- Requests are gzip-compressed and sent with credentials.
