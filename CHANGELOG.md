Version 0.6.0 - November 11, 2015
---------------------------------

Enhancements:

- Add `Place#getAttributes` for reading with custom metadata
- Support custom `Configuration` on initialization of `CampaignKitManager` via
  the factory `CampaignKitManager#getInstance(Context, Configuration)`
- Support basic Campaign segmentation on sync using tags set in a
  `Configuration` (segmentation is based on an `OR` of the tags)
- Support disabling cellular data for network communications through a flag in
  a `Configuration`

Bug Fixes:

- Fix race condition when accessing, the now deprecated (see below), factory
  `CampaignKitManager#getInstanceForApplication(Context)` from multiple threads
  prior to the initial creation of a `CampaignKitManager`
- Improved network and battery savings by defaulting to syncing Status Kit
  updates every 2 minutes instead of 30 seconds

Deprecations:

- Deprecate `CampaignKitManager#getInstanceForApplication(Context)` in favor of
  `CampaignKitManager#getInstance(Context, Configuration)`
- `CampaignKitManager` is not designed to be inherited. Several non-public
  helper methods were previously exposed as part of the public API via the
  `protected` access level modifier. This includes the following methods:

  - `CampaignKitManager(Context)`
  - `CampaignKitManager#getInstance()`
  - `CampaignKitManager#callDidFindCampaign(Campaign, Place)`
  - `CampaignKitManager#getAnalyticsURL()`
  - `CampaignKitManager#getAuthToken()`
  - `CampaignKitManager#getCampaignMap()`
  - `CampaignKitManager#setCampaignMap(HashMap<String, Campaign>)`
  - `CampaignKitManager#getPkManager()`
  - `CampaignKitManager#getTriggeredCampaigns()`
  - `CampaignKitManager#setTriggeredCampaigns(ArrayList<Campaign>)`
  - `CampaignKitManager#pruneAndReturnExpiredCampaigns()`
  - `CampaignKitManager#setCampaign(Campaign)`

  You should treat these methods as private implementation details and not use
  them. If you are using them we recommend moving away them them as soon as
  possible.
- Deprecate `CampaignKitManager#setPartnerIdentifier(String)` in favor of
  `CampaignKitManager#getInstance(Context, Configuration)`
- Deprecate `CampaignKitManager#clearPartnerIdentifier()`, no replacement


Version 0.5.4 - October 9, 2015
-------------------------------

Bug Fixes:

- Fix `NullPointerException` when attempting to create a `Place` from data
  which does not set a `type` value; this now raises a more helpful
  `IllegalArgumentException` stating that the "type" field was missing
- Fix `NullPointerException` when accessing `Place#getRssi` for regions or
  `Place#name` when the values are `null`
- Update local campaign cache persistence so it is more resilient to exceptions


Version 0.5.3 - September 9, 2015
---------------------------------

Bug Fixes:

- Remove deleted campaigns from local cache
- Fix issue where `Campaign#hashCode` did not align with the general Java
  contract for `Campaign#equals`
- Parse JSON timestamps as UTC instead of using the device's time zone
- Improve local campaign cache persistence format
- Update to [Proximity Kit 0.8.0](https://github.com/RadiusNetworks/proximitykit-android/releases/tag/0.8.0)


Version 0.5.2 - August 7, 2015
------------------------------

Bug Fixes:

- Set found at `Place` prior to calling `CampaignKitNotifier#didFindCampaign`
- Update local campaign cache with trigger distance on sync
- Improve recalculating local campaign cache recurrence times on sync
- Synchronize persistence of local cache
- Improve local campaign cache persistence frequency including saving on
  `CampaignKitManager#stop`
- Improve handling of local cache persistence on failed syncs


Version 0.5.1 - July 30, 2015
-----------------------------

Bug Fixes:

- Update to [Proximity Kit 0.7.0](https://github.com/RadiusNetworks/proximitykit-android/releases/tag/0.7.0)
  fixing several issues with background jobs consuming all resources from
  `AsyncTask`'s thread pools


Version 0.5.0 - July 29, 2015
-----------------------------

Enhancements:

- Add `CampaignKitManager#setPartnerIdentifier(String)` and
  `CampaignKitManager#clearPartnerIdentifier()` to the public API. Allowing
  the configuration of an additional custom identifier which is attached to
  analytic events.


Version 0.4.1 - July 15, 2015
-----------------------------

Bug Fixes:

- Fix issue where HTTPS requests were not made.
- Fix issue where an update to the "CampaignKit.properties" file is not
  recognized until the app is re-installed or the app data is cleared
- Fix issue where kits with more than 20 places would crash due to a
  `NullPointerException` when parsing the place ID.


Version 0.4.0 - May 28, 2015
----------------------------

Enhancements:

- Add `CampaignKitManager#stop()` to the public API

  This stops any further syncing with the cloud. It additionally, unregisters
  any beacons and geofences which are being monitored.

Bug Fixes:

- Fix Android SDK level check properly supporting geofences on Android SDK 9+
- Fix issue with ranging where the ranging timestamp is incorrectly store as a
  future time due to background task execution delays
- Fix issue where a group of beacons are ranged at the same time but may report
  slightly different timestamps


Version 0.3.0 - May 5, 2015
---------------------------

Enhancements:

- Support for [Director](https://director.radiusnetworks.com) analytics
- Upgrade to [Proximity Kit 0.6.1](https://github.com/RadiusNetworks/proximitykit-android/releases/tag/0.6.1)

Bug Fixes:

- Fix for recurring campaigns
- Remove static compiled 'support-v4' library
- Remove misleading parse error log entries


Version 0.2.2 - March 23, 2015
------------------------------

Bug Fixes:

- Upgrade to [ProximityKit 0.5.0](https://github.com/RadiusNetworks/proximitykit-android/releases/tag/0.5.0)


Version 0.2.0 - March 18, 2015
------------------------------

Enhancements:

- Added ability to limit campaign visibility by trigger distance. Minimum
  trigger distance is set from https://campaignkit.radiusnetworks.com
- Exposed distance and rssi values within some Place class instances
- Support to Android SDK level 22
- Upgrade to [Proximity Kit 0.4.0](https://github.com/RadiusNetworks/proximitykit-android/releases/tag/0.4.0)


Version 0.1.3 - February 17, 2015
---------------------------------

Bug Fixes:

- Update to [android-beacon-library 2.1.3](https://github.com/AltBeacon/android-beacon-library/releases/tag/2.1.3)
- Allow public access to all active and future campaigns through
  `CampaignManager.getActiveCampaigns()`


Version 0.1.2 - February 6, 2015
--------------------------------

Breaking Change:

- Require Android minimum SDK version 7+

Bug Fixes:

- Update to [android-beacon-library 2.1.2](https://github.com/AltBeacon/android-beacon-library/releases/tag/2.1.2)


Version 0.1.0 - October 28, 2014
--------------------------------

Initial release:

- Require JDK 1.7+
- Require Android minimum SDK version 1+
- Geofence support requires Android minimum SDK version 9+
- Beacon support requires Android minimum SDK version 18+
- Locked into Google Play services version 5.x
- Added foreground only capabilities
- Added Campaign Kit Analytics now works through Proximity Kit to minimize
  server interactions and save battery life
- Added Redemption Places to make the 'fulfilled' analytic possible
