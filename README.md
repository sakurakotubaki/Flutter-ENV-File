# Flutterで環境変数を使う
iOS,Androidで、API KEYを隠している環境変数を読み込むには、ネイティブ側で設定をする必要があります。

## iOSの場合
ios/Runner/配下に、Environment.swift作成して配置しますが、こちらのファイルは、x-codeを開いて、ファイルをドラッグ＆ドロップで配置しないと、iOS独特の謎のエラーが発生する原因になるようです。

ios/Runner/Environment.swift
```swift
import Foundation

struct Env {
  static let googleMapApiKey = "API_KEY_HERE"
}
```

ios/Runner/内に配置してあるAppDelegete.swiftに、今回は環境変数の読み込みをするGoogleMapAPIのコードを追加しました。
ios/Runner/AppDelegete.swift
```swift
import UIKit
import Flutter
import GoogleMaps

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GMSServices.provideAPIKey(Env.googleMapApiKey)
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

## Androidの場合
androidディレクトリ配下にsecret.propertiesを作成します。
API KEYは、""で囲まなくてOKです。
android/secret.properties
```
googleMap.apiKey=API_KEY_HERE
```

android/app/配下のbuild.gradleに環境変数を読み込めるように、設定を追加します。
android/app/build.gradle
```java
def localProperties = new Properties()
def localPropertiesFile = rootProject.file('local.properties')
if (localPropertiesFile.exists()) {
    localPropertiesFile.withReader('UTF-8') { reader ->
        localProperties.load(reader)
    }
}

// 環境変数を呼び出すコード
def secretProperties = new Properties()
def secretPropertiesFile = rootProject.file('secret.properties')
if (secretPropertiesFile.exists()) {
    secretPropertiesFile.withReader('UTF-8') { reader ->
        secretProperties.load(reader)
    }
}

def flutterRoot = localProperties.getProperty('flutter.sdk')
if (flutterRoot == null) {
    throw new GradleException("Flutter SDK not found. Define location with flutter.sdk in the local.properties file.")
}

def flutterVersionCode = localProperties.getProperty('flutter.versionCode')
if (flutterVersionCode == null) {
    flutterVersionCode = '1'
}

def flutterVersionName = localProperties.getProperty('flutter.versionName')
if (flutterVersionName == null) {
    flutterVersionName = '1.0'
}

apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply from: "$flutterRoot/packages/flutter_tools/gradle/flutter.gradle"

android {
    compileSdkVersion 33 // 33を指定
    ndkVersion flutter.ndkVersion

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }

    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
    }

    defaultConfig {
        // TODO: Specify your own unique Application ID (https://developer.android.com/studio/build/application-id.html).
        applicationId "com.example.api_env_app"
        // You can update the following values to match your application needs.
        // For more information, see: https://docs.flutter.dev/deployment/android#reviewing-the-gradle-build-configuration.
        minSdkVersion 21 // 21を指定
        targetSdkVersion 33 // 33を指定
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
        // 環境変数を呼び出すコード
        manifestPlaceholders = [
            googleMapApiKey: secretProperties.getProperty('googleMap.apiKey'),
        ]
    }

    buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig signingConfigs.debug
        }
    }
}

flutter {
    source '../..'
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
}
```

android/app/src/main/配下のAndroidManifext.xmlで環境変数を読み込めるように設定を追加します。
android/app/src/main/AndroidManifext.xml
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.api_env_app">

    <!-- 権限の追加 ここから-->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <!-- ここまで -->

    <!-- android:name="${applicationName}を削除する" -->
   <application
        android:label="api_env_app"
        android:icon="@mipmap/ic_launcher">

        <!-- google_maps_flutterのAPI KEYの設定 ここから-->
        <meta-data android:name="com.google.android.geo.API_KEY" android:value="${googleMapApiKey}"/>
        <!-- ここまで-->
        
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:launchMode="singleTop"
            android:theme="@style/LaunchTheme"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|smallestScreenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
            android:hardwareAccelerated="true"
            android:windowSoftInputMode="adjustResize">
            <!-- Specifies an Android theme to apply to this Activity as soon as
                 the Android process has started. This theme is visible to the user
                 while the Flutter UI initializes. After that, this theme continues
                 to determine the Window background behind the Flutter UI. -->
            <meta-data
              android:name="io.flutter.embedding.android.NormalTheme"
              android:resource="@style/NormalTheme"
              />
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        <!-- Don't delete the meta-data below.
             This is used by the Flutter tool to generate GeneratedPluginRegistrant.java -->
        <meta-data
            android:name="flutterEmbedding"
            android:value="2" />
    </application>
</manifest>
```

.gitignoreにAPI KEYを貼り付けたコードをコミットの対象から外す設定をする
```
/android/secret.properties

# iOS
/ios/Runner/Environment.swift
```