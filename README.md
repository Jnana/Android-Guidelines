# Android guidelines & best pratices

### Android SDK

Place your [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) somewhere in your home directory or some other application-independent location. Some IDEs include the SDK when installed, and may place it under the same directory as the IDE. This can be bad when you need to upgrade (or reinstall) the IDE, or when changing IDEs. Also avoid putting the SDK in another system-level directory that might need sudo permissions, if your IDE is running under your user and not under root.

### Build system

Your default option should be [Gradle](http://tools.android.com/tech-docs/new-build-system). Ant is much more limited and also more verbose. With Gradle, it's simple to:

- Build different flavours or variants of your app
- Make simple script-like tasks
- Manage and download dependencies
- Customize keystores
- And more  

### Project structure

There are two popular options: the old Ant & Eclipse ADT project structure, and the new Gradle & Android Studio project structure. You should choose the new project structure. If your project uses the old structure, consider it legacy and start porting it to the new structure.

Old structure:

```
old-structure
├─ assets
├─ libs
├─ res
├─ src
│  └─ is/gangverk/project
├─ AndroidManifest.xml
├─ build.gradle
├─ project.properties
└─ proguard-rules.pro
```

New structure:

```
new-structure
├─ library-foobar
├─ app
│  ├─ libs
│  ├─ src
│  │  ├─ androidTest
│  │  │  └─ java
│  │  │     └─ is/gangverk/project
│  │  └─ main
│  │     ├─ java
│  │     │  └─ com/gangverk/project
│  │     ├─ res
│  │     └─ AndroidManifest.xml
│  ├─ build.gradle
│  └─ proguard-rules.pro
├─ build.gradle
└─ settings.gradle
```

The main difference is that the new structure explicitly separates 'source sets' (`main`, `androidTest`), a concept from Gradle. You could, for instance, add source sets 'paid' and 'free' into `src` which will have source code for the paid and free flavours of your app.

Having a top-level `app` is useful to distinguish your app from other library projects (e.g., `library-foobar`) that will be referenced in your app. The `settings.gradle` then keeps references to these library projects, which `app/build.gradle` can reference to.  

### Gradle configuration

**General structure.** Follow [Google's guide on Gradle for Android](http://tools.android.com/tech-docs/new-build-system/user-guide)

**Small tasks.** Instead of (shell, Python, Perl, etc) scripts, you can make tasks in Gradle. Just follow [Gradle's documentation](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF) for more details.

**Passwords.** In your app's `build.gradle` you will need to define the `signingConfigs` for the release build. Here is what you should avoid:

_Don't do this_. This would appear in the version control system.

```groovy
signingConfigs {
    release {
        storeFile file("myapp.keystore")
        storePassword "password123"
        keyAlias "thekey"
        keyPassword "password789"
    }
}
```

Instead, make a `gradle.properties` file which should _not_ be added to the version control system:

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```

That file is automatically imported by gradle, so you can use it in `build.gradle` as such:

```groovy
signingConfigs {
    release {
        try {
            storeFile file("myapp.keystore")
            storePassword KEYSTORE_PASSWORD
            keyAlias "thekey"
            keyPassword KEY_PASSWORD
        }
        catch (ex) {
            throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
        }
    }
}
```

**Prefer Maven dependency resolution instead of importing jar files.** If you explicitly include jar files in your project, they will be of some specific frozen version, such as `2.1.1`. Downloading jars and handling updates is cumbersome, this is a problem that Maven solves properly, and is also encouraged in Android Gradle builds. For example:

```groovy
dependencies {
    compile 'com.squareup.okhttp:okhttp:2.2.0'
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.2.0'
}
```    

**Avoid Maven dynamic dependency resolution**
Avoid the use of dynamically versioned, such as `2.1.+` as this may result in different and unstable builds or subtle, untracked differences in behavior between builds. The use of static versions such as `2.1.1` helps create a more stable, predictable and repeatable development environment.

### Java code style
We use the [Android Open-Source Project (AOSP)](http://source.android.com/source/code-style.html) Code Style Guidelines.

We strongly suggest that you use the `AndroidStyle.xml` template file in Android Studio to format your code:

1. Place [AndroidStyle.xml](https://github.com/android/platform_development/blob/master/ide/intellij/codestyles/AndroidStyle.xml) in your Android Studio `/codestyles` directory (e.g., `~/Library/Preferences/AndroidStudio/codestyles/`)
2. Restart Android Studio.
3. Go to "File->Settings->Code Style", and under "Scheme" select "AndroidStyle" and click "Ok".
4. Right-click on the files that contain your contributions and select "Reformat Code", check "Optimize imports", and select "Run".  

## Code review 
At each pull request, assign someone to check out your code, here is what we look for: 
### Code Consistency

The code should be consistently formatted. Consistent code reduces syntax related distractions and helps us focus on the important logic. Use a code linter if necessary to help you spot these inconsistencies.

*Example of Inconsistent Code:*

```java
if (a == b) {
  //
} else if (a == c)
{
  //
}
else {
  //
}
```

### Variable Naming Clarity

We should be able to understand the logic of code blocks by reading the names of variables and functions. A clearly named variable means we won't need to look up the definition of the variable or function right away in order to understand what it does.

*Example of Clear Code:*

```java
private void loadMedia(String mediaId, boolean autoPlay) throws
        TransientNetworkDisconnectionException, NoConnectionException, JSONException {
    String musicId = MediaIDHelper.extractMusicIDFromMediaID(mediaId);
    android.media.MediaMetadata track = mMusicProvider.getMusic(musicId);
    if (track == null) {
        throw new IllegalArgumentException("Invalid mediaId " + mediaId);
    }
    if (!TextUtils.equals(mediaId, mCurrentMediaId)) {
        mCurrentMediaId = mediaId;
        mCurrentPosition = 0;
    }
    JSONObject customData = new JSONObject();
    customData.put(ITEM_ID, mediaId);
    MediaInfo media = toCastMediaMetadata(track, customData);
    mCastManager.loadMedia(media, autoPlay, mCurrentPosition, customData);
}
```
### Useful Comments

A comment should be used in cases where the code cannot be formatted to fix clarity. The comment should allow us to skip over code blocks when looking for an overview of the code. We can come back later to the smaller bits of code to confirm they do what the comment says.

*Example of Useful Comments:*

```java
/**
 * Implementation of the Playback.Callback interface
 */
@Override
public void onCompletion() {
    // The media player finished playing the current song, so we go ahead
    // and start the next.
    if (mPlayingQueue != null && !mPlayingQueue.isEmpty()) {
        // In this sample, we restart the playing queue when it gets to the end:
        mCurrentIndexOnQueue++;
        if (mCurrentIndexOnQueue >= mPlayingQueue.size()) {
            mCurrentIndexOnQueue = 0;
        }
        handlePlayRequest();
    } else {
        // If there is nothing to play, we stop and release the resources:
        handleStopRequest(null);
    }
}
```

### Follow field naming conventions (from AOSP)  

- Non-public, non-static field names start with m.  
- Static field names start with s.
- Other fields start with a lower case letter.
- Public static final fields (constants) are ALL_CAPS_WITH_UNDERSCORES.  

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
```
