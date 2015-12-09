# Campaign Kit for Android

Requirements for use:

* Minimum Android API Level 7.
* Android API Level 18 or higher to use AltBeacon features.
* Android API Level 9 or higher to use geofencing features.
* Google Play Services library set as a dependency. The library version must be
  within the range of 4.2 - 8.x
* `CampaignKit.properties` file downloaded from
  https://campaignkit.radiusnetworks.com.


## Sample Reference App

We've created a simple app demonstrating the basic setup and usage of Campaign
Kit for Android:

- Android Studio (recommended):
  https://github.com/RadiusNetworks/campaignkit-reference-android-studio
- Eclipse: https://github.com/RadiusNetworks/campaignkit-reference-android


## Getting Started

1. If you haven't downloaded the Campaign Kit library and set it up as a
   dependent library for your project, [do that now](download).
2. Create a singleton instance of a `CampaignKitManager`

  A `CampaignKitManager` instance is the primary interface for working with
  Campaign Kit. It provides access to `Campaign` objects and serves as the
  central registration point for receiving beacon/geofence related event
  notifications.

  Currently the Campaign Kit for Android library manages a singleton instance
  of a `CampaignKitManager` for you. However, this will not always be the case.
  We _strongly_ suggest you spend the time to setup your own singleton instance
  now. This will greatly help ease future upgrade.

  For simplicity in this guide, we will manage our singleton in a [custom `Application` subclass](https://github.com/RadiusNetworks/campaignkit-reference-android-studio/blob/master/app/src/main/java/com/radiusnetworks/campaignkitreference/MyApplication.java).
  If your project does not already have one create a new class in your source
  directory which `extends Application`.

  To initialize the Campaign Kit manager you will need the configuration
  settings for your kit. These can be downloaded from the Campaign Kit server
  as a `.properties` file. These settings can be loaded directly from the file,
  statically compiled into your source, or load via another mechanism of your
  choice..

  For newer Android applications, we suggest simply adding the file as an
  _asset_ (e.g. placed in [`PROJECT_ROOT/app/src/assets`](https://github.com/RadiusNetworks/campaignkit-reference-android-studio/tree/master/app/src/main/assets)).
  We can use a helper to load the properties for us:

  ```java
  private Configuration loadConfig() {
      Properties properties = new Properties();
      try {
          properties.load(getAssets().open("CampaignKit.properties"));
      } catch (IOException e) {
          throw new IllegalStateException("Unable to load properties file!", e);
      }
      return new Configuration(properties);
  }
  ```

  Older apps which upgrade are likely treating the `.properties` file as a
  resouce (e.g. `PROJECT_ROOT/app/src/resources`). If you do not wish to
  migrate to `/assets` or simply prefer using Java resources, replace the above
  method with the following:

  ```java
  private Configuration loadConfig() {
      Properties properties = new Properties();
      InputStream in = getClassLoader().getResourceAsStream("CampaignKit.properties");
      if (in == null) {
          throw new IllegalStateException("Unable to find CampaignKit.properties files");
      }
      try {
          properties.load(in);
      } catch (IOException e) {
          throw new IllegalStateException("Unable to load properties file!", e);
      }
      return new Configuration(properties);
  }
  ```

  We can now create our singleton instance in the `onCreate` method:

  ```java
  public class MyApplication extends Application {
      /**
       * Storage for an instance of the manager
       */
      private static CampaignKitManager ckManager = null;

      /**
       * Object to use as a thread-safe lock
       */
      private static final Object ckManagerLock = new Object();

      @Override
      public void onCreate() {
          super.onCreate();

          synchronized (ckManagerLock) {
              if (ckManager == null) {
                  ckManager = CampaignKitManager.getInstance(this, loadConfig());
              }
          }

          ckManager.start();
      }

      private Configuration loadConfig() {
          Properties properties = new Properties();
          try {
              properties.load(getAssets().open("CampaignKit.properties"));
          } catch (IOException e) {
              throw new IllegalStateException("Unable to load properties file!", e);
          }
          return new Configuration(properties);
      }
  }
  ```
3. Receive interesting event notifications by implementing `CampaignKitNotifier`

  Adjust the application class to implement `CampaignKitNotifier`. If this
  interface is not recognized manually import it or use the IDE's quick fix
  (Android Studio: ALT + ENTER, Eclipse: CMD + 1). Be sure to implement all of
  the required methods for the interface. Both Android Studio and Eclipse can
  do this for you via quick-fix.

  In order to receive the notifications we need to tell the
  `CampaignKitManager` to send them to our class. We do this by sending
  `CampaignKitManager#setNotifier(this)` in the `onCreate` method. We want to
  do this before we send `CampaignKitManager#start()` so that we don't miss any
  potential notifications.

  Your class should now look similar to:

  ```java
  public class MyApplication extends Application implements CampaignKitNotifier {
      /**
       * Storage for an instance of the manager
       */
      private static CampaignKitManager ckManager = null;

      /**
       * Object to use as a thread-safe lock
       */
      private static final Object ckManagerLock = new Object();

      @Override
      public void onCreate() {
          super.onCreate();

          synchronized (ckManagerLock) {
              if (ckManager == null) {
                  ckManager = CampaignKitManager.getInstance(this, loadConfig());
              }
          }

          ckManager.setNotifier(this);

          ckManager.start();
      }

      @Override
      public void didFindCampaign(Campaign campaign) {

      }

      @Override
      public void didDetectPlace(Place place, CKEventType ckEventType) {

      }

      @Override
      public void didSync() {

      }

      @Override
      public void didFailSync(Exception e) {

      }

      private Configuration loadConfig() {
          Properties properties = new Properties();
          try {
              properties.load(getAssets().open("CampaignKit.properties"));
          } catch (IOException e) {
              throw new IllegalStateException("Unable to load properties file!", e);
          }
          return new Configuration(properties);
      }
  }
  ```

  Implement these methods as you see fit for your app. For some ideas, check
  out the [reference app](https://github.com/RadiusNetworks/campaignkit-reference-android-studio/blob/master/app/src/main/java/com/radiusnetworks/campaignkitreference/MyApplication.java).

4. Targeting Android 6 (API level/SDK version 23) and higher

  If you decide to target Android 6+ you will need to [request runtime
  permissions](http://developer.radiusnetworks.com/2015/09/29/is-your-beacon-app-ready-for-android-6.html).
  This will be necessary if you want to detect beacons in the background and/or
  use geofences.

  When and how to ask for permissions is a decision best left up to you - the
  app developer. Google provides some handy [patterns](https://www.google.com/design/spec/patterns/permissions.html)
  for when and how to request permissions within your app.

  Our [reference app](https://github.com/RadiusNetworks/campaignkit-reference-android-studio)
  has implemented a very [basic approach](https://github.com/RadiusNetworks/campaignkit-reference-android-studio/blob/master/app/src/main/java/com/radiusnetworks/campaignkitreference/MainActivity.java).
  Geofences and background beacon detection are core to the app's
  functionality. So we've decided to request permissions up front. Due to
  permissions potentially being revoked at any time we recheck them every time
  the app is opened through our main activity's `onStart()` method.

  ```java
  public class MainActivity extends FragmentActivity {
      /**
       * Request code to use when launching the location permission resolution activity
       */
      private static final int REQUEST_LOCATION_ACCESS = 1;

      /**
       * Verify critical permissions and settings every time the app is brought to the foreground.
       */
      @Override
      protected void onStart() {
          super.onStart();
          togglePermissionFeatures();
      }

      /**
       * Called after the user has either denied or granted our permission request.
       *
       * @see #togglePermissionFeatures()
       */
      @Override
      public void onRequestPermissionsResult(int requestCode,
                                             @NonNull String[] permissions,
                                             @NonNull int[] grantResults) {
          switch (requestCode) {
              case REQUEST_LOCATION_ACCESS:
                  // Received permission result for location access

                  // Check if the only required permission has been granted
                  if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                      // Location access has been granted, geofences can be enabled
                      enableGeofences();
                  } else {
                      // Location access was denied, so we cannot enable geofences
                      Toast.makeText(
                              this,
                              "Both background beacon detection and geofence events are prevented.",
                              Toast.LENGTH_SHORT
                      ).show();
                      disableGeofences();
                  }
                  break;
              default:
                  super.onRequestPermissionsResult(requestCode, permissions, grantResults);
          }
      }

      private void disableGeofences() {

      }

      private void enableGeofences() {

      }

      @TargetApi(Build.VERSION_CODES.M)
      private void togglePermissionFeatures() {
          if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
              // All permissions requested at install time
              enableGeofences();
              return;
          }

          // We may already have permission so we need to check
          // If you are not using geofences you should request ACCESS_COARSE_LOCATION:
          // String permission = Manifest.permission.ACCESS_COARSE_LOCATION;
          String permission = Manifest.permission.ACCESS_FINE_LOCATION;
          if (checkSelfPermission(permission) == PackageManager.PERMISSION_GRANTED) {
              // Location permission has already been granted
              enableGeofences();
              return;
          }

          // Location access has not been granted

          /*
           * Check if we should provide an additional rationale to the user if the permission was not
           * granted and the user would benefit from additional context for the use of the permission.
           *
           * Will return `false` if the permission is disabled on the device or if the user has
           * checked "Don't ask me again!". Will also be `false` the first time this permission
           * is being requested. Thus this returns `true` only if we've requested the
           * permission once before and were denied. This is a potential signal that the user
           * might be confused about the app behavior and why this permission is necessary.
           */
          if (shouldShowRequestPermissionRationale(permission)) {
              Toast.makeText(
                      this,
                      "Location access is needed to trigger both campaigns in the background and " +
                              "those attached to geofences",
                      Toast.LENGTH_LONG
              ).show();
          }
          requestPermissions(new String[]{permission}, REQUEST_LOCATION_ACCESS);
      }
  }
  ```

  Lastly, be sure to review the latest [guides](https://developer.android.com/training/permissions/index.html) on requesting permissions.


## Adding Geofence Support

Since not everyone uses geofences, and using geofences requires some additional
app setup, they are disabled by default. Below are basic the steps necessary to
configure an app to use geofences with Google Play Services.

1. Install Google Play Services.

  Due to the differences between Eclipse and Android Studio please refer to the
  [Google Setup docs](https://developers.google.com/android/guides/setup#ensure)
  for the proper IDE instructions.

2. If using gradle (such as with Android Studio), include Google Play Services
   as a dependency in `app/build.gradle`:

  ```gradle
  // PROJECT_ROOT/app/build.gradle
  dependencies {
      compile 'com.google.android.gms:play-services:8.3.0'
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile 'com.radiusnetworks:campaignkit-android:0+@aar'
  }
  ```

3. Declare the Google Play Service in the app's `AndroidManifest.xml` under the
   `<application>` section:

  ```xml
  <meta-data
      android:name="com.google.android.gms.version"
      android:value="@integer/google_play_services_version" />
  ```

4. Declare that the app needs to request `ACCESS_FINE_LOCATION`.

  To request this permission, add the following element as a child element of
  the `<manifest>` element:

  ```xml
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
  ```

5. Check for Google Play support.

  For the most recent suggestions by Google, please refer to the [Android
  documentation on checking for Google Play services support](https://developers.google.com/android/guides/setup#ensure).

  > Because each app uses Google Play services differently, it's up to you
  > decide the appropriate place in your app to check verify the Google Play
  > services version. For example, if Google Play services is required for your
  > app at all times, you might want to do it when your app first launches. On
  > the other hand, if Google Play services is an optional part of your app,
  > you can check the version only once the user navigates to that portion of
  > your app.
  >
  > ...
  >
  > Another approach is to use the `isGooglePlayServicesAvailable()` method.
  > You might call this method in the `onResume()` method of the main activity.
  > If the result code is `SUCCESS`, then the Google Play services APK is
  > up-to-date and you can continue to make a connection. If, however, the
  > result code is `SERVICE_MISSING`, `SERVICE_VERSION_UPDATE_REQUIRED`, or
  > `SERVICE_DISABLED`, then the user needs to install an update. In this case,
  > call the `getErrorDialog()` method and pass it the result error code. The
  > method returns a `Dialog` you should show, which provides an appropriate
  > message about the error and provides an action that takes the user to
  > Google Play Store to install the update.

  In the reference app we use some helpers in the activity so that we can
  bundle the logic for enabling and disabling geofences. The disabling is
  straight forward we simply ask our application to delegate the call.

  For enabling we need to make sure we have Google Play Services first. Per the
  above advice we've created our own check `isGooglePlayServicesAvailable`
  (this name mirrors the Google Play API name). Only if this is `true` do we
  attempt enable geofences.

  ```java
  private void disableGeofences() {
      MyApplication.disableGeofences();
  }

  /**
   * Check Google Play services, location status, and enable geofences in Campaign Kit.
   */
  private void enableGeofences() {
      if (isGooglePlayServicesAvailable()) {
          // As a safety mechanism, `enableGeofences()` throws a checked exception in case the
          // app does not properly handle Google Play support.
          try {
              MyApplication.enableGeofences();
          } catch (GooglePlayServicesException e) {
              Log.e(TAG, "Expected Google Play to be available but enabling geofences failed", e);
          }
      }
  }
  ```

  The logic for our `isGooglePlayServicesAvailable` helper comes directly from
  the Google Play API guides. We use a field to track if we are already
  attempting to resolve an issue with Google Play, so that we do not notify the
  user twice during a pending action.

  We then ask the Google Play API, via
  `GoogleApiAvailability#isGooglePlayServicesAvailable`, if the phone supports
  our required Google Play version. If yes we return `true` which will allow
  geofences to be enabled. Otherwise, we mark that we need to resolve an error
  and we let the Google Play API display the appropriate notification.

  ```java
  /**
   * Bool to track whether the app is already resolving a Google Play Services error
   */
  private boolean resolvingError = false;

  /**
   * Verify that Google Play services is available.
   */
  private boolean isGooglePlayServicesAvailable() {
      if (resolvingError) {
          // Already attempting to resolve an error.
          return false;
      }

      GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();

      // Check that Google Play services is available
      int statusCode = apiAvailability.isGooglePlayServicesAvailable(this);
      switch (statusCode) {
          case ConnectionResult.SUCCESS:
              Log.d(TAG, "Google Play Service available");
              return true;
          case ConnectionResult.SERVICE_MISSING:
          case ConnectionResult.SERVICE_UPDATING:
          case ConnectionResult.SERVICE_VERSION_UPDATE_REQUIRED:
          case ConnectionResult.SERVICE_DISABLED:
          case ConnectionResult.SERVICE_INVALID:
              // Taking the easy way out: log it.
              Log.w(TAG, apiAvailability.getErrorString(statusCode));
              showGooglePlayErrorDialog(statusCode);
              resolvingError = true;
      }

      return false;
  }
  ```

  We've used the boiler plate setup for displaying the Google Play API error
  per the Google docs. It creates a new dialog fragment with the error details.
  The `GoogleApiAvailability#getErrorDialog` call will setup the dialog to open
  the appropriate setting or link in the Google Play store based on the error
  received.

  If the user cancels the dialog we mark that we're done resolving the error,
  but take no action as Google Play services has already been denied.

  ```java
  /**
   * Unique tag for the error code in the dialog fragment bundle
   */
  private static final String DIALOG_ERROR = "dialog_error";

  /**
   * Request code to use when launching the Google Play Services resolution activity
   */
  private static final int REQUEST_RESOLVE_ERROR = 1001;

  /**
   * Called from ErrorDialogFragment when the dialog is dismissed.
   */
  public void onGooglePlayDialogDismissed() {
      resolvingError = false;
  }

  /**
   * Display a dialog to the user explaining the Google Play Service error.
   */
  private void showGooglePlayErrorDialog(int errorCode) {
      // Create a fragment for the error dialog
      ErrorDialogFragment dialogFragment = new ErrorDialogFragment();

      // Pass the error that should be displayed
      Bundle args = new Bundle();
      args.putInt(DIALOG_ERROR, errorCode);
      dialogFragment.setArguments(args);
      dialogFragment.show(getSupportFragmentManager(), "errordialog");
  }

  /**
   * A fragment to display an error dialog
   */
  public static class ErrorDialogFragment extends DialogFragment {
      public ErrorDialogFragment() {
      }

      @Override
      public Dialog onCreateDialog(Bundle savedInstanceState) {
          // Get the error code and retrieve the appropriate dialog
          int errorCode = getArguments().getInt(DIALOG_ERROR);
          return GoogleApiAvailability.getInstance().getErrorDialog(
                  getActivity(),
                  errorCode,
                  REQUEST_RESOLVE_ERROR
          );
      }

      @Override
      public void onDismiss(DialogInterface dialog) {
          ((MainActivity) getActivity()).onGooglePlayDialogDismissed();
      }
  }
  ```

  If the user takes an action to resolve the error, the dialog will notify our
  activity via the standard `onActivityResult`. Here we determine if we can
  enable geofences. In the failure case we explicitly disable geofences. This
  covers the condition where we previously had geofences enabled, the user
  disables Google Play in their settings, then the next time the app opens they
  do not adjust the setting. In that case we need to disable geofences
  appropriately.

  ```java
  /**
   * Handle the result from the Google Play Services error dialog.
   */
  @Override
  protected void onActivityResult(int requestCode, int resultCode, Intent data) {
      if (requestCode == REQUEST_RESOLVE_ERROR) {
          resolvingError = false;
          if (resultCode == RESULT_OK) {
              enableGeofences();
          } else {
              disableGeofences();
          }
      }
  }
  ```

  Lastly, it is possible that while the Google Play Services dialog is being
  display the user rotates the screen, or performs another action, which causes
  the activity to be recreated. To handle this we save our error resolving
  state, `resolvingError`, in the saved instance data. We reload it when the
  new activity is created in `onCreate`.

  ```java
  /**
   * Name of the saved {@link #resolvingError} value in the saved bundle.
   */
  private static final String STATE_RESOLVING_ERROR = "resolving_error";

  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);

      /*
       * While the Google Play Services dialog is showing the user may rotate the screen or
       * perform another action which causes the activity to be recreated. We reload the state
       * we were in so that when `onStart()` is called again we know to abort the permission
       * check as it is still pending.
       */
      resolvingError = savedInstanceState != null &&
              savedInstanceState.getBoolean(STATE_RESOLVING_ERROR, false);

      // ...
  }

  /**
   * Save the state of any outstanding Google Play Services requests.
   */
  @Override
  protected void onSaveInstanceState(Bundle outState) {
      super.onSaveInstanceState(outState);
      outState.putBoolean(STATE_RESOLVING_ERROR, resolvingError);
  }
  ```

