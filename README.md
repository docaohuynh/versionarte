# versionarte

Force update, show update indicator and disable the app for maintenance with total freedom over the UI.

<img src="https://raw.githubusercontent.com/kamranbekirovyz/versionarte/main/assets/cover.png" alt="cover_picture" />

Features can be implemented with versionarte:
- ✋ Force users to update to the latest version
- 💆🏻‍♂️ Have separate values for each platform
- 🚧 Disable app for maintenance with custom informative text
- 🆕 Inform users about an optional update availability
- 🔗 Launch the App Store on iOS and Play Store on Android

## 🚀 Getting Started

Add the package to your `pubspec.yaml` file:

```yaml
dependencies:
  versionarte: <latest_version>
```

Import the package in your Dart code:

```dart
import 'package:versionarte/versionarte.dart';
```

## 📡 Obtain the status

Call `Versionarte.check` method by providing it a `VersionarteProvider` (an object responsible for fetching the versioning information from a remote service) to get a `VersionarteResult` (an object containing app's versioning and availability information).

There are 2 built-in providers, [RemoteConfigVersionarteProvider](#1-using-firebase-remote-config) and [RestfulVersionarteProvider](#2-using-restful-api), which fetches the versioning information from Firebase Remote Config and RESTful API respectively. You can also create your own custom provider by extending the [VersionarteProvider](#3-using-custom-versionarteprovider) class.

### 1. Using Firebase Remote Config

The `RemoteConfigVersionarteProvider` fetches information stored in Firebase Remote Config with the key name of "versionarte". You need to set up the Firebase Remote Config service before using this provider. See <a href="https://github.com/kamranbekirovyz/versionarte/blob/main/firebase_remote_config_setup.md#-firebase-remote-config-setup-guide" target="_blank">Firebase Remote Config setup guide</a> to learn more about configuration.

Example:

```dart
final result = await Versionarte.check(
    versionarteProvider: RemoteConfigVersionarteProvider(),
);
```

Optional parameters:
- `keyName`: key name for the Firebase Remote Config to fetch. By default, it's set to "versionarte". Specify if you upload the <a href="https://github.com/kamranbekirovyz/versionarte?tab=readme-ov-file#%EF%B8%8F-configuration-json">Configuration JSON</a> using a different key name.
- `initializeInternally`: if your project already initializes and configures Firebase Remote Config, set this to `false`. By default, it's set to `true`.
- `remoteConfigSettings`: settings for Firebase Remote Config if `initializeInternally` set to true. By default, `fetchTimeout` and `minimumFetchInterval` are set to `10 seconds`.

### 2. Using RESTful API

The `RestfulVersionarteProvider` fetches versioning and availability information by sending HTTP GET request to the specified URL with optional headers. The response body should be a JSON string that follows the <a href="https://github.com/kamranbekirovyz/versionarte?tab=readme-ov-file#%EF%B8%8F-configuration-json">Configuration JSON</a> format.

Example:

```dart
final result = await Versionarte.check(
    versionarteProvider: RestfulVersionarteProvider(
        url: 'https://myapi.com/getVersioning',
    ),
);
```

Optional parameters:
- `headers`: headers to send with the HTTP GET request. By default, it's set to an empty map.

### 3. Using custom VersionarteProvider

To use remote services to provide versioning and availability information of your app, extend the `VersionarteProvider` class and override the `getStoreVersioning` method which is responsible for fetching the information and returning it as a `StoreVersioning` object.

```dart
class MyCustomVersionarteProvider extends VersionarteProvider {
  @override
  Future<StoreVersioning> getStoreVersioning() async {
    final result = MyCustomService.fetchVersioning();

    final decodedResult = jsonDecode(result);

    return StoreVersioning.fromJson(decodedResult);
  }
```
    
Example:
```dart
final result = await Versionarte.check(
    versionarteProvider: MyCustomVersionarteProvider(),
);
```

## 🎯 Handle the status

Obtained `VersionarteResult` has 3 parameters:

- `status`: (VersionarteResult) the status of the app. It can be one of the following values:
    - `VersionarteStatus.inactive`: the app is inactive for usage.
    - `VersionarteStatus.forcedUpdate`:  user must update before continuing.
    - `VersionarteStatus.outdated`: user can continue with and without updating.
    - `VersionarteStatus.upToDate`: the user's version is up to date.
    - `VersionarteStatus.unknown`: error occured while checking status.
- `details`: (StorePlatformDetails) Details for the current platform, including messages for when the app is inactive. 

Then, based on `status` do the if-else checks:

```dart
if (result.status == VersionarteResult.inactive) {
    final message = result.details.status.getMessageForLanguage('en');
    // TODO: Handle the case where the app is inactive
} else if (result == VersionarteResult.forcedUpdate) {
    // TODO: Handle the case where an update is required
} else if (result == VersionarteResult.upToDate) {
    // TODO: Handle the case where an update is optional
} 
```

## 🔗 Launching the download stores

To launch download page of the app use `Versionarte.launchDownloadUrl`: 

```dart
final Map<TargetPlatform, String?> downloadUrls = result.storeVersioning!.downloadUrls;

await Versionarte.launchDownloadUrl(downloadUrls);
```

💡 Don't forget to add "download_url" property to each platform that you support on <a href="https://github.com/kamranbekirovyz/versionarte?tab=readme-ov-file#%EF%B8%8F-configuration-json">Configuration JSON</a>
💡 Launching store won't work on iOS simulator due to its limitations.

See the <a href="https://github.com/kamranbekirovyz/versionarte/tree/main/example">example</a> directory for a complete sample app.

## 🖋️ Configuration JSON

For providing app's status and availability information, versionarte requires a specific JSON format. Whether you're using `RemoteConfigVersionarteProvider`, `RestfulVersionarteProvider`, or a custom `VersionarteProvider`, make sure to use this JSON format.

💡 Information for all platforms in the JSON is not necessary: you can provide information for only one platform, or for two platforms, or for all three platforms.   
💡 While the app status is active, the `message` can be left empty or set to `null`.

```js
{
    "android": {
        "version": {
            "minimum": "2.7.0",
            "latest": "2.8.0"
        },
        "download_url": "https://play.google.com/store/apps/details?id=app.librokit",
        "status": {
            "active": true,
            "message": {
                "en": "App is in maintanence mode, please come back later.",
                "es": "La aplicación está en modo de mantenimiento, vuelva más tarde."
            }
        }
    },
    "iOS": {
        // Same stucture as above
    }
    "macOS": {
        // Same stucture as above
    },
    "windows": {
        // Same stucture as above
    },
    "linux": {
        // Same stucture as above
    }
}
```

This JSON represents information stored separately for three platforms, containing the minimum and latest versions, and the availability status.

Each platform contains two objects:

- `version`:
    - `minimum`: The minimum version of the app users can use. 
    - `latest`: The latest version of the app available. 
    - `download_url`: The URL to download the app from the store.
- `status`:
    - `active`: A boolean that indicates whether the app is currently active or not.
    - `message`: A Map that contains the messages for different languages to be displayed to the user when app is inactive. The keys of the map represent the language codes (e.g., "en" for English, "es" for Spanish), and the values represent the message in that language.

## 🐞 Faced issues?

If you encounter any problems or you feel the library is missing a feature, please raise a ticket on <a href=https://github.com/kamranbekirovyz/versionarte/issues>GitHub</a> and I'll look into it. 

## 📃 License

<a href="https://github.com/kamranbekirovyz/versionarte/blob/main/LICENSE">MIT License</a>