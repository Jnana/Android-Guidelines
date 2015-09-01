# Android-Guidelines
## Java code style
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
