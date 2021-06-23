It is important to sign both debug and release version with same key to test links. Server side configuration relies on application signature. By default, for debug signature is unique for each environment. So please use
```
cordova run android --buildConfig build.json
```
For production environment it is better do not have keystore password in build configuration, so
```
cordova run android --buildConfig build.json --password ...
```

