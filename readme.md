## Cordova Deep Link Sample

This is sample app which can be started by clicking specific link in broswer on Android device.

### Server side

You should host some web page that have a link pointing to some location you whant to handle by the app. You also
should provide way for the Android OS to verify you own the site. To do this, you should put
`.well-known/assetlinks.json` file with the app signature on same domain. Official documentation on verification
[available here](https://developer.android.com/training/app-links/verify-site-associations). Basically you need
to get your app signing certificate fingerprint by

```
keytool -list -v -keystore my-release-key.keystore
```

and put it into following json:

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

Sample for files hosted on the server you can find in server folder. You can use following test page for this app:
[sandbox.sprinting.io/test.html](https://sandbox.sprinting.io/test.html).

It is important to sign both debug and release version with same certificate to test links. By default, for debug
certificate is unique for each environment. So please use

```
cordova run android --buildConfig build.json
```

For production environment it is better do not have keystore password in build configuration, so

```
cordova run android --buildConfig build.json --password ...
```

## Application configuration

First you should create intent filter handling deepl links it your application manifest. You can take a look on the
[offical documentation](https://developer.android.com/training/app-links/deep-linking) before doing this. Basiacally
you need entry in the Cordova config creating additional intent filter in Android manifest:

```
<platform name="android">
  <config-file target="AndroidManifest.xml" parent="/manifest/application/activity[@android:name='MainActivity']">
    <intent-filter android:autoVerify="true">
      ...
```

Now you have to react on deep link Intent. To do this, you should have dependency on pluging allowinh to handle
intents. In this sample `snakelizzard/cordova-plugin-intent` is used. See [www/js/index.js](www/js/index.js) for
reference.

```
window.plugins.intentShim.onIntent(intent => doSomething(intent));
window.plugins.intentShim.getIntent(intent => doSomething(intent),
        () => console.log('Error getting launch intent'));
```

First line will handle case when intent delivered for running application. Second line is for the case when the
application is initially started with deep link intent. You can check `intent.action` to be
`android.intent.action.VIEW` for links and `intent.data` should hold the url for this case.

### Testing without server

You can just do

```
adb shell am start -W -a android.intent.action.VIEW -d \"your-url" your-app-package
```

to send test intent.
