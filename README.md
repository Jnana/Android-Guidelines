# Android Guidelines

List of guidelines that we use at [Gangverk](http://gangverk.is) when developing for the __Android__ platform.

* [Project and code style guidelines](project_and_code_guidelines.md)

## 1 Code review 
At each pull request, assign someone to check out your code, in addition to respecting those [guidelines](project_and_code_guidelines.md), here is also what we look for: 

### 1.1 Code Consistency

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

### 1.2 Variable Naming Clarity

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
### 1.3 Useful Comments

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
