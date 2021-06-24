## Cordova Deep Link Sample

This is a sample app that can be started by clicking a specific link in the browser on an Android device.

### Server side

You should host some web page that has a link pointing to some location you want to handle by the app. You also
should provide a way for the Android OS to verify you own the site. To do this, you should put
`.well-known/assetlinks.json` file with the app signature on the same domain. Official documentation on verification
[available here](https://developer.android.com/training/app-links/verify-site-associations). You need
to get your app signing certificate fingerprint by

```
keytool -list -v -keystore my-release-key.keystore
```

and put it into the following JSON:

```
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "your-package-name",
    "sha256_cert_fingerprints": ["your-fingerprint"]
  }
}]
```

Sample files hosted on the server you can find in the server folder. You can use the following test page for this app:
[sandbox.sprinting.io/test.html](https://sandbox.sprinting.io/test.html).

It is important to sign both debug and release versions with the same certificate to test links. By default, for debug,
the certificate is unique for each environment. So please use

```
cordova run android --buildConfig build.json
```

For the production environment, it is better do not to have a keystore password in the build configuration, so

```
cordova run android --buildConfig build.json --password ...
```

## Application configuration

First, you should create an intent filter handling deep links in your application manifest. You can take a look at the
[offical documentation](https://developer.android.com/training/app-links/deep-linking) before doing this. 
You need an entry in the Cordova config creating an additional intent filter in the Android manifest:

```
<platform name="android">
  <config-file target="AndroidManifest.xml" parent="/manifest/application/activity[@android:name='MainActivity']">
    <intent-filter android:autoVerify="true">
      ...
```

Now you have to react on deep link Intent. To do this, you should have a dependency on the plugin allowing you to handle
intents. In this sample `snakelizzard/cordova-plugin-intent` is used. See [www/js/index.js](www/js/index.js) for
reference.

```
window.plugins.intentShim.onIntent(intent => doSomething(intent));
window.plugins.intentShim.getIntent(intent => doSomething(intent),
        () => console.log('Error getting launch intent'));
```

The first line will handle the case when the intent is delivered for running the application. The second line is for the case when the
application is initially started with deep link intent. You can check `intent.action` to be
`android.intent.action.VIEW` for links and `intent.data` should hold the URL for this case.

### Testing without server

You can just do

```
adb shell am start -W -a android.intent.action.VIEW -d \"your-url\" your-app-package
```

to send test intent.
