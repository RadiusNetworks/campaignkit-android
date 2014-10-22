
#Campaign Kit Library for Android Client

Requirements for use: 

* Android API Level 18 or higher

* Google Play Services library set as a dependency if using Geofencing features.

* CampaignKit.properties file downloaded from campaignkit.radiusnetworks.com.




##Sample Reference Apps

We've created some simple apps that demonstrate usage of the Campaign Kit Library.

For Android Studio users:
https://github.com/RadiusNetworks/campaignkit-reference-android-studio

For Eclipse users:
https://github.com/RadiusNetworks/campaignkit-reference-android




##Getting Started


1) If you haven't downloaded the Campaign Kit Library and set it up as a dependent library to your project, please [do that first](https://github.com/RadiusNetworks/campaignkit-documentation/blob/master/docs/android/download.md).


2) Implement CampaignKitNotifier and CampaignKitManager

 * Open the Application class in your project. If you don't have one, create a class that extends Application.

 * Type 'extends Application implements CampaignKitNotifier" after the class name in its declaration.

 * Add all required methods for the CampaignKitNotifier. Quick-fix can do this for you (move your cursor onto CampaignKitNotifer and press ALT + ENTER on Android Studio or CMD + 1 on Eclipse for mac).

 * Create an instance of the CampaignKitManager (i.e. CampaignKitManager _ckManager;).

 * In the Application class's onCreate() method, add the following code:

```java 
    _ckManager = CampaignKitManager.getInstanceForApplication(this);
    _ckManager.start();
    _ckManager.setNotifier(this);
``` 

3) Handle didFindCampaign callback from CampaignKitNotifier

```java

  @Override
  public void didFindCampaign(Campaign campaign) {

    //sending notification or alert, depending on whether app is in background or foreground
    new CampaignNotificationBuilder(_mainActivity, campaign)
    .setSmallIcon(R.drawable.ic_launcher)
    .show();
    
    //displaying Campaign content
    showCampaign(campaign);
  }

  public void showCampaign(Campaign campaign){
    // TODO: write custom code or use code from the Campaign Kit reference app
  }
```



##Adding Geofence Support

Since geofence support is relatively new, it is disabled by default. Below are
the steps necessary to configure a sample app to use geofences through
Campaign Kit.

1) Install Google Play services. Due to the differences between Eclipse and
   Android Studio please refer to the [Google Setup docs](https://developer.android.com/google/play-services/setup.html)
   for the proper IDE instructions.


   - Android SDK Manager > Extras > Google Play services

2) If using Android Studio, Include the Google Play services as a dependency in `app/build.gradle`:

```groovy
  // PROJECT_ROOT/app/build.gradle
  dependencies {
      compile 'com.android.support:appcompat-v7:+'
      compile 'com.google.android.gms:play-services:+'
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile project(":campaignkit-android")
  }
```

3) Declare the Google Play service in the app's `AndroidManifest.xml` under the
   `<application>` section:

```xml
  <meta-data
      android:name="com.google.android.gms.version"
      android:value="@integer/google_play_services_version" />
```

4) Declare that the app needs to request `ACCESS_FINE_LOCATION`. To request
   this permission, add the following element as a child element of the
   `<manifest>` element:

```groovy
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

5) Check for Google Play support. For the most recent suggestions by Google,
   please refer to the [Android documentation on checking for Google Play
   services support](https://developer.android.com/google/play-services/setup.html#ensure).

  > Because each app uses Google Play services differently, it's up to you
  > decide the appropriate place in your app to check verify the Google Play
  > services version. For example, if Google Play services is required for your
  > app at all times, you might want to do it when your app first launches. On
  > the other hand, if Google Play services is an optional part of your app,
  > you can check the version only once the user navigates to that portion of
  > your app.
  >
  > To verify the Google Play services version, call
  > `isGooglePlayServicesAvailable()`. If the result code is `SUCCESS`, then
  > the Google Play services APK is up-to-date and you can continue to make a
  > connection. If, however, the result code is `SERVICE_MISSING`,
  > `SERVICE_VERSION_UPDATE_REQUIRED`, or `SERVICE_DISABLED`, then the user
  > needs to install an update. So, call
  > `GooglePlayServicesUtil.getErrorDialog()` and pass it the result error
  > code. This returns a `Dialog` you should show, which provides an
  > appropriate message about the error and provides an action that takes the
  > user to Google Play Store to install the update.


```java

  /**
   * Verify that Google Play services is available before making a request.
   *
   * @return true if Google Play services is available, otherwise false
   */
  private boolean servicesConnected() {

      // Check that Google Play services is available
      int resultCode =
              GooglePlayServicesUtil.isGooglePlayServicesAvailable(this);

      // If Google Play services is available
      if (ConnectionResult.SUCCESS == resultCode) {

          // In debug mode, log the status
          Log.d(TAG, "Google Play services available");

          // Continue
          return true;

      // Google Play services was not available for some reason
      } else {

          // Display an error dialog
          Dialog dialog = GooglePlayServicesUtil.getErrorDialog(resultCode, this, 0);
          if (dialog != null) {
              ErrorDialogFragment errorFragment = new ErrorDialogFragment();
              errorFragment.setDialog(dialog);
              errorFragment.show(getSupportFragmentManager(), TAG);
          }
          return false;
      }
  }

  /**
   * Define a DialogFragment to display the error dialog generated in
   * showErrorDialog.
   */
  public static class ErrorDialogFragment extends DialogFragment {

      // Global field to contain the error dialog
      private Dialog mDialog;

      /**
       * Default constructor. Sets the dialog field to null
       */
      public ErrorDialogFragment() {
          super();
          mDialog = null;
      }

      /**
       * Set the dialog to display
       *
       * @param dialog An error dialog
       */
      public void setDialog(Dialog dialog) {
          mDialog = dialog;
      }

      /*
       * This method must return a Dialog to the DialogFragment.
       */
      @Override
      public Dialog onCreateDialog(Bundle savedInstanceState) {
          return mDialog;
      }
  }

```

6) If Google Play is available, enable geofences for Campaign Kit.

```java

  ckManager = CampaignKitManager.getInstanceForApplication(this);
  if (servicesConnected()) {
      try {
          ckManager.enableGeofences();
      } catch (GooglePlayServicesException e) {
          Log.e(TAG, "Failed to enable geofences: " + e.getMessage());
      }
  }
  ckManager.setNotifier(this);
  ckManager.start()

```

