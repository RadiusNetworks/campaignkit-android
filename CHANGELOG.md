## Version 0.15.0 - August 9, 2018

Enhancements:

- Support Android 8.1 (API 27)
- Support Google Play 15.x

Deprecations:

- `Campaign#getIdAsInt` has been deprecated and will be removed in the next
  major release.

  While the ids happen to be integers today that may change in the future. Plan
  now to assume ids may be any form of string value.

Breaking Changes:

- `Campaign#getAttributes` now returns an unmodifiable map

  While this is technically a breaking change, as now modifications will throw
  an `java.lang.UnsupportedOperationException`, it better reflects the intended
  owner of the attributes. After a sync, the campaign is updated to match the
  server. This includes resetting the attributes to only those from the server.
- `Campaign#getEndAt` and `Campaign#getStartAt` now return copies of the dates

  While this is technically a breaking change, as you can no longer mutate the
  start and end times, these getters should always have returned defensive
  copies.
- `Campaign#equals` changes the behavior for campaigns with missing ids

  When a `Campaign` instance lacks an id it will no longer be equal to other
  instances that are also missing an id. Instead they will now only be equal to
  themselves (i.e. the same object instance). `Campaign` equality is based on
  the `id` and without this value comparison behavior is undefined.


## Version 0.14.0 - January 11, 2018

Enhancements:

- Support Android 8.0 (API 26)

  Android 8.0 imposes new restrictions on BLE scanning while in the background.
  This release changes the behavior on Android 8 to use the `JobScheduler` to
  perform background scans. Due to limitations of the OS these background scan
  jobs may not be scheduled faster than once every 15 minutes. However, when a
  scan job completes, if no beacons are detected a low power scan will begin.
  This scan will continue to run until any beacon is detected at which point
  the `JobScheduler` will start again.

  Foreground behavior remains the same as prior releases even on Android 8.
  Additionally, these background changes only affects devices running Android
  8.0 or newer. Devices running older Android versions will continue to use the
  existing legacy behavior.

  For more details see [Background Execution Limits](https://developer.android.com/about/versions/oreo/background.html)
  and [Background Location Limits](https://developer.android.com/about/versions/oreo/background-location-limits.html).

  Apps updating to this release need to bump their `com.android.support`
  packages to 26.0.0 or newer due to changes in Android 8.0 requiring
  notifications to be sent to channels. Apps using
  `CampaignNotificationBuilder` should use the new
  `CampaignNotificationBuilder#setChannelId(String channelId)` or
  `CampaignNotificationBuilder#show(String channelId)` API for support with
  channels.
- Support Google Play 11.x
- Expose `CampaignNotificationBuilder` intent `Bundle` keys storing information
  about the triggering campaign and source
- Use `android.support.v7.app.AlertDialog` for in app alerts built with
  `CampaignNotificationBuilder`; falls back to `android.app.AlertDialog` for
  apps which do not include the `appcompat-v7` lib or whose activities do not
  use compatible themes
- Add support to change notification theme with `CampaignNotificationBuilder#setTheme`
- Add support for setting the notification text title and color through
  `CampaignNotificationBuilder#setDefaultColor` and
  `CampaignNotificationBuilder#setColor`
- Hooks into location setting broadcasts for improved geofence based campaign
  support on devices running KitKat (API 19) or higher
- When location services is disabled, or otherwise stops providing location,
  `CampaignKitNotifier#didDetectPlace` will now be called with event
  `CKEventType.CKEventDidDetectStateUnknown`

Bug Fixes:

- Make `CampaignNotificationBuilder` alert dialog cancel button language aware
  through `android.R.string.cancel`
- Fixes an issue where after geofence based campaigns have been registered and
  location services disabled, then re-enabled, not all campaigns may trigger
  after the next sync
- Fix race condition where toggling geofences on/off may have resulted in
  inconsistent geofence registrations resulting in related campaigns not
  triggering when expected
- Fix triggering campaigns for active geofence (entered/inside) after sync
- Ensure in-memory active geofences (those the device is currently within) have
  attribute and campaign information updated after a sync.
- Clean geofence state when geofences are disabled.
- Improve triggering geofence based campaigns related to a variety of edge
  cases around improper internal geofence tracking and location services state
  changes
- Fix triggering of recurring campaigns and future campaigns while inside
  geofences

Deprecations:

- `Campaign#checkIsConnectedToRedemptionPlace` has been deprecated as it was
  always an internal implementation detail that was accidentally exposed in the
  published API
- `Campaign#updateCampaign` has been deprecated as it was always an internal
  implementation detail that was accidentally exposed in the published API


## Version 0.13.1 - November 30, 2017

Bug Fixes:

- Fix bug causing the configuration setting `ALLOW_UNSUPPORTED_GOOGLE_PLAY` to
  always be `false` after a sync.
- Fix manifest element nesting which may cause a lint error on manifest merge


## Version 0.13.0 - May 19, 2017

Enhancements:

- Support Android 7.x (API 24 and 25)
- Support Google Play 10.x
- Improve sync efficiency by using E-Tags for conditional sync
- Support toggling between running separate `CampaignKitManager` instances;
  only a single instance is allowed to run at any one time

Bug Fixes:

- Fix analytic upload timestamp formatting
- Fix edge case crash caused by parsing a malformed BLE packet
- Fix UI slowness, and possible hang, in Android 6+ when Location Services
  permissions are revoked while scanning

  It is still recommended that apps always check permissions and BLE/Location
  Services state when returning from the background. If the appropriate
  permissions or services are not in the desired state either engage the user
  or explicitly stop the `CampaignKitManager`.
- Fix edge case `Context` leak with manager creation
- Protect against `SecurityException` crashes caused by Samsung Knox
- Fix crash caused by `IllegalStateException` when bluetooth adapter is off for
  HTC devices on Android 6.0+
- Prevent UI lag, and possible ANR (Application Not Responding) message, by not
  blocking the main thread when starting or stopping a bluetooth scan
- Don't start scanning when BLE feature is unavailable
- Fix issue where campaign analytic state was lost when the app was force
  closed or the device rebooted

Breaking Changes:

- Remove `android.permission.WAKE_LOCK` from manifest; the SDK does not use
  this position and its inclusion forces an unnecessary permission onto apps


## Version 0.12.3 - November 7, 2016

Bug Fixes:

- Fix bug causing rapid kit syncs when setting a custom sync interval


## Version 0.12.2 - October 27, 2016

Bug Fixes:

- Fix crash caused by a race condition resulting in a
  `ConcurrentModificationException` which may occur during a sync while near
  beacons
- Fix crash caused by a race condition resulting in a `NullPointerException`
  which may occur during a sync while near beacons


## Version 0.12.1 - September 16, 2016

Bug Fixes:

- Fix crash on wifi or power state change which occurs before a Campaign Kit
  manager has been created
- Fix race condition queuing multiple syncs on wifi or power state change
- Respect configured sync interval when syncing on wifi or power state changes
- Use sync interval provided in kit config
- Prevent duplicate geofence enter events on sync


## Version 0.12.0 - September 7, 2016

Enhancements:

- Make app id unique to the install not specific to a device
- Regenerate app id if value becomes corrupted
- Add support for version 9.x of the Google Play services library
- Add nullability annotations to provide better insight into the public API and
  help developers guard against `NullPointerException` only when necessary.
  See https://developer.android.com/studio/write/annotations.html for further
  details.
- Support canceling alerts built using `CampaignNotificationBuilder`
- Adds config setting `Configuration.ALLOW_UNSUPPORTED_GOOGLE_PLAY` to by-pass
  the Google Play version check. This may result in a runtime crash as it will
  allow running against untested versions of Google Play.
- Retry analytic data upload when no network is available
- Attempt analytic upload with out delay; previously there was a 30 second
  delay from recording a campaign event and the upload
- Attempt analytic upload on successful kit sync
- Add ability to transmit beacon advertisements on supported hardware

Bug Fixes:

- Improve thread safety around starting and stopping Campaign Kit Manager
- Improve thread safety around background time sync; ensuring only a single
  sync task is ever queued at a time
- Improve handling of notifications created using `CampaignNotificationBuilder`
  when generated using a non-activity `Context`
- Fix geofence region monitoring race condition which may exclude a region
- Monitor existing geofences on `start` from local cache
- Fix race condition which may drop some analytic data
- Fix queuing duplicate analytic upload job on upload error
- Move analytic upload off `AsyncTask` queue on Android Honeycomb and newer
- Prune analytic data after upload

Breaking Changes:

- Not exactly a _breaking_ change but a slight behavior modification. The
  `MissingKeyException` and `MissingValueException` classes now do _not_
  automatically append the key name to the provided detail message. This change
  is to fall more in line with how existing exceptions work as well as provide
  better support for internationalized messages in the future.

Deprecations:

- The `CampaignKitManager` has exposed several public methods due to its
  implementing several Proximity Kit callback interfaces. This was a legacy
  internal implementation detail and these methods were only public due to Java
  inheritance semantics. These methods were never meant to be part of the
  public API and should not be called. They will be removed in the next major
  release. The full list of methods follows:

  - `CampaignKitManager#didEnterGeofence(ProximityKitGeofenceRegion)`
  - `CampaignKitManager#didExitGeofence(ProximityKitGeofenceRegion)`
  - `CampaignKitManager#didDetermineStateForGeofence(int, ProximityKitGeofenceRegion)`
  - `CampaignKitManager#didEnterRegion(ProximityKitBeaconRegion)`
  - `CampaignKitManager#didExitRegion(ProximityKitBeaconRegion)`
  - `CampaignKitManager#didDetermineStateForRegion(int, ProximityKitBeaconRegion)`
  - `CampaignKitManager#didRangeBeaconsInRegion(Collection<ProximityKitBeacon>, ProximityKitBeaconRegion)`
  - `CampaignKitManager#didSync()`
  - `CampaignKitManager#didFailSync(Exception)`
  - `CampaignKitManager#()`
- Deprecate support for 4.x - 7.x of the Google Play service library.

  This only adds a warning message to the logs when an app is built against a
  version of Google Play services in this range. Apps will continue to function
  as expected but developers are encourage to move to a newer version of Google
  Play.

  For historical reference version 4.4.x of Google Play services was released
  in May 2014. The most recent deprecated version, 7.8.x, was released in
  August 2015.


Version 0.11.0 - July 21, 2016
------------------------------

Version number jumped so that it follows the our common Android release cycle.

Enhancements:

- Update to [Proximity Kit 0.11.0](https://github.com/RadiusNetworks/proximitykit-android/releases/tag/0.11.0)
- Report OS platform (as Android) with analytics and syncs
- Include local device bluetooth state on server sync
- Guard against remote exception when processing ranging events

Bug Fixes:

- Fix `NullPointerException` edge case caused by a network issue on kit sync

Breaking Changes:

- Change the lib's manifest package name to `com.radiusnetworks.campaignkit`
  from the previously incorrect `com.example.campaignkit_android`


Version 0.6.3 - March 14, 2016
------------------------------

Bug Fixes:

- Fix issue where campaigns with trigger distances may not be found after
  entering the region at a greater distance
- Fix issue where campaigns with trigger distances may not be found after a
  sync when the associated place was previously entered
- Reflect place field updates after sync when calling `didDetectPlace` for
  previously entered places
- Fix `NullPointerException` caused when the `CampaignKitNotifier` is not set
  or is set to `null`
- Fix `NoClassDefFoundError` thrown when ranging beacons or sending
  notifications on Android Honeycomb (API 13) and below
- Fix a `NullPointerException` caused by stopping the manager while a failed
  sync is being processed
- Fix `NullPointerException` which occurs when enabling geofences for a kit
  that has none setup
- Fix a `NullPointerException` caused by disabling geofences while a sync is
  being processed
- Fix issue where removing the map from a kit does not clear monitored
  geofences after a sync
- Perform beacon and geofence monitoring/ranging changes prior to calling
  `CampaignKitNotifier#didSync`


Version 0.6.2 - February 23, 2016
--------------------------------

Bug Fixes:

- Update to [AltBeacon/android-beacon-library](https://github.com/AltBeacon/android-beacon-library)
  version [2.7.1](https://github.com/AltBeacon/android-beacon-library/releases/tag/2.7.1)
  to fix a race condition which could cause scanning to stop


Version 0.6.1 - November 25, 2015
---------------------------------

Bug Fixes:

- Properly read network input streams allowing for accurate traffic stats
- Ensure network input streams are closed
- Ensure network connections are closed
- Starts and monitor known geofences when enabling geofences after already
  starting the manager
- Improve thread safety around `start()`, `stop()`, and enabling / disabling
  geofences
- Record minors for fully qualified regions in analytics
- Properly record beacon analytics when ranging in regions
- Covert times to UTC in analytics JSON


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
- Support Google Play services 8.x

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
