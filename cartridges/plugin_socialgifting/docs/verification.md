# Verification — 8 October 2026

## Automated

- 42 unit tests (`npm test`), JavaScript lint, production JS/CSS build.
- Metadata and job XML validate against the Salesforce schemas.

## Sandbox

Sandbox `zyeu-002`, site `RefArch_Practice`, SFRA 8. The cartridge was uploaded, the metadata in
`metadata/social-gifting` imported, and `SocialGiftingEnabled`, `WeddingRegistryEnabled`,
`GroupGiftingEnabled`, `AnonymousGiftingEnabled`, `RegistryCommentsEnabled`, `RegistryPollsEnabled`,
`RegistryReservationEnabled`, `GuestVotingEnabled` and `RegistryPurchaseEnabled` switched on.
Flows were driven in headless Chrome; screenshots are in the [feature guide](../../../README.md#features).

| Flow | Result |
| --- | --- |
| Guest opens `Registry-Dashboard` | Redirected to login |
| Create a wedding registry (public, active, address from address book) | Created; redirected to the registry page |
| Enable privacy and participation options | Saved |
| Add the selected variant from the product page | Added; a second add of the same variant is refused |
| Add a product by ID with quantity, notes and group gifting | Added |
| Comment on an item | Shown under the item, escaped |
| Create a poll over two items, guest voting allowed | Shown to members and guests; guest vote accepted |
| Invite a co-owner by email | "Invitation sent." (acceptance not tested) |
| Create a group gift | "Contribute · 0 / 150.00 USD" on the item; group gift page opens |
| Guest reserves an item | Reserved: 1, Remaining: 0 |
| Second visitor reserves or buys the same unit | Refused: "This gift has already been purchased or reserved…" |
| Guest buys a registry item anonymously | Checkout starts at payment with the recipient hidden; order 00000203 placed with the basic-credit test processor; item shows Purchased: 1 |
| Normal Add to Cart and checkout | Unaffected once the metadata is imported |

## Fixed during verification

- **Every registry page returned HTTP 500** (`Header name Cache-Control is not allowed to be set or added`).
  SFCC forbids setting `Cache-Control`; the middleware now relies on `cachePeriod = 0` and a past
  `Expires`, which keep responses out of shared caches.
- **Registry gifts were assigned the store-pickup method.** The first applicable shipping method was
  `005 Store Pickup`. Registry deliveries now skip methods with `storePickupEnabled` and prefer the
  site default.
- **Add to Cart failed site-wide** while the cartridge was on the path without its metadata
  (`Unknown dynamic property 'sgContributionKey'`). This is the documented install order: import the
  metadata first.

## Observed, not changed

- Counts on My Registries and remaining quantities are printed with a decimal (`3.0 items`, `Remaining: 1.0`).
- The poll title field is labelled "Registry name".
- Poll results are not visible to the voter unless "Show results before closing" is set.

## Not verified

- Contribution payments: no funding product (`GroupGiftContributionProductID`) was configured, so no
  contribution was paid, captured or refunded.
- Group-gift fulfilment (`app.socialGifting.fulfillment.createOrder`), invitation acceptance, secret
  gift mode, archive, notification emails and scheduled jobs.
- The opt-in concurrency tests in `test/integration` (they need a disposable sandbox fixture).
- The SFRA basic-credit processor is a test processor; it does not prove real card capture.
