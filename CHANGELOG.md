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
