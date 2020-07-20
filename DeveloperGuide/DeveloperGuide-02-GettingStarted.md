#  开发者指南 [【原文】](<https://exoplayer.dev/>)

## 2. 开始使用

### 2.1 [**Hello world!**](https://exoplayer.dev/hello-world.html)

> 也可以使用ExoPlayer codelab 来开始。Another way to get started is to work through the [ExoPlayer codelab](https://codelabs.developers.google.com/codelabs/exoplayer-intro/).

简单使用`ExoPlayer`包括以下几个步骤：

1. 在你的项目中添加 ExoPlayer 依赖
2. 创建 SimpleExoPlayer 实例
3. 把 ExoPlayer 和 View 关联（实现视频输出和处理用户操作）
4. 为播放器准备 MediaSource 开始播放
5. 播放结束时释放播放器

以下是步骤详细说明。完整示例请参参考[参考应用](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/)中的`PlayerActivity`.

#### 2.1.1 将ExoPlayer添加为依赖项

##### 1. 添加存储库

在项目根目录的`build.gradle`中添加Google和JCenter库。

```groovy
repositories {
    google()
    jcenter()
}
```

##### 2. 添加ExoPlayer依赖

接下来，在 app module 的`build.gradle` 中添加依赖项。以下内容将对完整的ExoPlayer库添加依赖项：

```groovy
implementation 'com.google.android.exoplayer:exoplayer:2.X.X'
```

`2.X.X`替换为首选版本（可通过参考[release note](https://github.com/google/ExoPlayer/tree/release-v2/RELEASENOTES.md)找到最新版本）。

作为完整库的替代方法，您可以仅依赖实际需要的库模块。例如，以下内容将添加对Core，DASH和UI库模块的依赖关系，这可能是播放DASH内容的应用程序所必需的：

```
implementation 'com.google.android.exoplayer:exoplayer-core:2.X.X'
implementation 'com.google.android.exoplayer:exoplayer-dash:2.X.X'
implementation 'com.google.android.exoplayer:exoplayer-ui:2.X.X'
```

可用的库模块在下面列出。添加完整ExoPlayer依赖等于分别添加所有依赖项。

- `exoplayer-core`：核心功能（必填）。
- `exoplayer-dash`：支持DASH内容。
- `exoplayer-hls`：支持HLS内容。
- `exoplayer-smoothstreaming`：支持SmoothStreaming内容。
- `exoplayer-ui`：与ExoPlayer一起使用的UI组件和资源。

除了库模块之外，ExoPlayer还具有多个扩展模块，这些扩展模块依赖于外部库来提供附加功能。浏览 [扩展目录](https://github.com/google/ExoPlayer/tree/release-v2/extensions/)及其各自的自述文件以了解详细信息。

##### 3. Java 8

在`build.gradle`中添加以下内容到ExoPlayer 来打开所有文件中的Java 8支持：

```groovy
compileOptions {
  targetCompatibility JavaVersion.VERSION_1_8
}
```

#### 2.1.2 创建播放器

您可以使用`SimpleExoPlayer.Builder`或`ExoPlayer.Builder`创建`ExoPlayer`实例。builder提供了一系列用于创建`ExoPlayer`实例的定制选项。绝大多数情况都应该使用 `SimpleExoPlayer.Builder`。此构建器返回 `SimpleExoPlayer`，它扩展自`ExoPlayer`以便添加其他高级播放器功能。以下代码是创建的示例`SimpleExoPlayer`。

```java
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
```

+ 关于线程的注释

必须从单个应用程序线程访问ExoPlayer实例。在绝大多数情况下，这应该是应用程序的主线程。使用ExoPlayer的UI组件或IMA扩展时，必须使用应用程序的主线程。

访问`ExoPlayer`的线程必须明确指定创建了`Exoplayer`的线程的`Looper`（注，这块的翻译很绕，我理解就是创建和访问ExoPlayer实例的线程必须是一个，即处在同一个`Looper`循环下）。如果没有指定`Looper` ，则使用创建播放器的线程的`Looper`，如果该线程没有`Looper`，则使用应用程序主线程的`Looper`。在所有情况下，访问播放器的线程都应该使用 `Player.getApplicationLooper`。

> 如果出现了“在错误的线程上访问了播放器”警告，则表明应用程序中的某些代码正在错误的线程上访问`SimpleExoPlayer`实例（错误堆栈跟踪显示了代码位置）。这是不安全的，并且可能导致意外或模糊的错误。

有关ExoPlayer线程模型的更多信息，请参见[ExoPlayer Javadoc “线程模型”部分](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ExoPlayer.html) 。

#### 2.1.3 将播放器添加到视图

ExoPlayer库提供了一个`PlayerView`，其中封装了 `PlayerControlView`，`SubtitleView`和 `Surface`来渲染视频。`PlayerView`可以包含在应用程序的布局XML。将播放器绑定到视图很简单：

```java
// Bind the player to the view.
playerView.setPlayer(player);
```

如果您需要在播放器控制细粒度的控制和`Surface` 在其上渲染视频，可以设置玩家的目标`SurfaceView`， `TextureView`，`SurfaceHolder`或`Surface`直接使用`SimpleExoPlayer`的 `setVideoSurfaceView`，`setVideoTextureView`，`setVideoSurfaceHolder`和 `setVideoSurface`对应的方法。您也可以`PlayerControlView`用作独立组件，或实现自己的播放控件，这些控件直接与播放器进行交互。`SimpleExoPlayer`的`addTextOutput`方法可用于在播放期间接收字幕。

#### 2.1.4 初始化 Player

在ExoPlayer中，每种媒体都由`MediaSource`表示。要播放某种媒体，您必须先创建一个对应的媒体`MediaSource`，然后将此对象传递给`ExoPlayer.prepare`。ExoPlayer库提供 `MediaSource`DASH（`DashMediaSource`），SmoothStreaming（`SsMediaSource`），HLS（`HlsMediaSource`）和常规媒体文件（`ProgressiveMediaSource`）的实现。以下代码显示了如何准备`MediaSource`适合播放MP4文件的播放器。

```java
// Produces DataSource instances through which media data is loaded.
DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(context,
    Util.getUserAgent(context, "yourApplicationName"));
// This is the MediaSource representing the media to be played.
MediaSource videoSource =
    new ProgressiveMediaSource.Factory(dataSourceFactory)
        .createMediaSource(mp4VideoUri);
// Prepare the player with the source.
player.prepare(videoSource);
```

#### 2.1.5 控制播放器

准备好播放器后，可以通过调用播放器上的方法来控制播放。例如，`setPlayWhenReady`开始和暂停播放，各种`seekTo`方法都可以在媒体中搜索，`setRepeatMode`控制是否循环媒体以及如何循环播放，`setShuffleModeEnabled`控制播放列表改组以及 `setPlaybackParameters`调整播放速度和音高。

如果播放器绑定到`PlayerView`或`PlayerControlView`，则用户与这些组件的交互将导致调用播放器上的相应方法。

#### 2.1.6 释放播放器

在不再需要播放器时将其释放十分重要的，这样可以释放有限的资源（例如视频解码器）供其他应用程序使用。这可以通过调用来完成`ExoPlayer.release`。



### 2.2 监听播放器事件 [**Listening to player events**](https://exoplayer.dev/listening-to-player-events.html)

状态更改和播放错误等事件将报告给已注册 [`Player.EventListener`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/Player.EventListener.html)实例。注册监听器以接收此类事件：

```java
// Add a listener to receive events from the player.
player.addListener(eventListener);
```

`Player.EventListener`有空的默认方法，因此您只需要实现需要的方法即可。有关方法及其调用时间的完整说明，请参见[Javadoc](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/Player.EventListener.html)。其中最重要的两个是 `onPlayerStateChanged`和`onPlayerError`，下面将对其进行详细说明。

##### a) 播放器状态变更

播放器状态的变化可以通过`onPlayerStateChanged(boolean playWhenReady, int playbackState)`在已注册的`Player.EventListener`中接收。播放器可以处于以下四种播放状态之一：

- `Player.STATE_IDLE`：这是初始状态，即播放器停止和播放失败时的状态。
- `Player.STATE_BUFFERING`：由于需要加载更多数据，播放器无法立即从当前位置播放。
- `Player.STATE_READY`：播放器可以立即从当前位置播放。
- `Player.STATE_ENDED`：播放器播放完所有媒体。

除了这些状态之外，播放器还具有`playWhenReady`标记用于准备好后立即播放。

使用`Player.isPlaying`检查播放器是否正在播放（即，位置正在前进） 。

```java
@Override
public void onIsPlayingChanged(boolean isPlaying) {
  if (isPlaying) {
    // Active playback.
  } else {
    // Not playing because playback is paused, ended, suppressed, or the player
    // is buffering, stopped or failed. Check player.getPlaybackState,
    // player.getPlayWhenReady, player.getPlaybackError and
    // player.getPlaybackSuppressionReason for details.
  }
}
```

##### b) 播放器错误

播放错误可以通过`onPlayerError(ExoPlaybackException error)`在已注册的`Player.EventListener`中监听。发生故障时，该方法会在播放状态转换为`Player.STATE_IDLE`的回调之前立即调用。失败或停止播放后可以通过`ExoPlayer.retry`来重试。

[`ExoPlaybackException`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ExoPlaybackException.html)有一个`type`字段，以及对应的 getter 方法，该字段返回故障的更多信息。下例显示了如何检测由于HTTP网络问题而导致的播放失败。

```java
@Override
public void onPlayerError(ExoPlaybackException error) {
  if (error.type == ExoPlaybackException.TYPE_SOURCE) {
    IOException cause = error.getSourceException();
    if (cause instanceof HttpDataSourceException) {
      // An HTTP error occurred.
      HttpDataSourceException httpError = (HttpDataSourceException) cause;
      // This is the request for which the error occurred.
      DataSpec requestDataSpec = httpError.dataSpec;
      // It's possible to find out more about the error both by casting and by
      // querying the cause.
      if (httpError instanceof HttpDataSource.InvalidResponseCodeException) {
        // Cast to InvalidResponseCodeException and retrieve the response code,
        // message and headers.
      } else {
        // Try calling httpError.getCause() to retrieve the underlying cause,
        // although note that it may be null.
      }
    }
  }
}
```

##### c) 搜索 //TODO 机翻调整

调用`Player.seekTo`方法会导致对已注册`Player.EventListener`实例的一系列回调 ：

1. `onPositionDiscontinuity`与`reason=DISCONTINUITY_REASON_SEEK`。这是调用的直接结果`Player.seekTo`。
2. `onPlayerStateChanged`以及与搜索相关的任何即时状态更改。请注意，状态可能不会发生任何变化，例如，是否可以在已加载的缓冲区中解析搜索。
3. `onSeekProcessed`。这表明玩家已完成搜索，并且已进行了所有必要的更改。如果播放器需要从搜寻到的位置缓冲新数据，则播放状态将 `Player.STATE_BUFFERING`在这一点上。

如果您使用`AnalyticsListener`，则会在`onSeekStarted`之前有一个附加事件 `onPositionDiscontinuity`，以指示在搜寻开始之前的播放位置。

#### 其他SimpleExoPlayer监听器

使用时`SimpleExoPlayer`，可以在播放器中注册更多监听器。

- `addAnalyticsListener`：聆听详细事件，这些事件可能用于分析和报告的目的。
- `addVideoListener`：监听与视频渲染有关的事件，可能用于调整UI（例如，`Surface`正在渲染视频的长宽比）。
- `addAudioListener`：监听与音频相关的事件，例如设置了音频session ID 和播放器音量改变。
- `addTextOutput`：监听字幕或字幕提示的改变。
- `addMetadataOutput`：监听定时的元数据事件，例如定时的ID3和EMSG数据。

ExoPlayer的UI组件（例如`PlayerView`）将自己注册他们所需要的事件的侦听器。因此，使用上述方法进行手动注册仅对实现自己的播放器UI或需要出于其他目的监听事件的应用程序有用。

#### 使用EventLogger

默认情况下，ExoPlayer仅记录错误。要将播放事件记录到控制台， 可以使用`EventLogger`类。它提供的其他日志记录有助于理解播放器的操作以及调试播放问题。`EventLogger`实现了`AnalyticsListener`，因此使用`SimpleExoPlayer`注册实例将十分简单：

```java
player.addAnalyticsListener(new EventLogger(trackSelector));
```

#### 输出日志

查看日志的最简单方法是使用Android Studio的[logcat选项卡](https://developer.android.com/studio/debug/am-logcat)。您可以通过程序包名称（如果使用的是演示应用，包名为`com.google.android.exoplayer2.demo`）将您的应用程序选择为调试进程， 并通过选择”仅显示所选应用程序”来告诉logcat选项卡仅记录该应用程序。可以使用表达式 `EventLogger|ExoPlayerImpl`过滤`EventLogger`和播放器本身的日志。

Android Studio的logcat标签的替代方法是使用shell命令。例如：

```shell
adb logcat EventLogger:* ExoPlayerImpl:* *:s
```

##### a) 播放器信息

`ExoPlayerImpl`类输出包含应用使用的播放器版本，设备和操作系统的两行日志：

```
ExoPlayerImpl: Release 2cd6e65 [ExoPlayerLib/2.9.6] [marlin, Pixel XL, Google, 26] [goog.exo.core, goog.exo.ui, goog.exo.dash]
ExoPlayerImpl: Init 2e5194c [ExoPlayerLib/2.9.6] [marlin, Pixel XL, Google, 26]
```

##### b) 播放状态

播放状态更改将输出为以下格式。在此示例中，在初始缓冲之后，播放没有再进入缓冲状态，并且被用户暂停了一次：

```
EventLogger: state [eventTime=0.00, mediaPos=0.00, window=0, true, BUFFERING]
EventLogger: state [eventTime=0.92, mediaPos=0.04, window=0, period=0, true, READY]
EventLogger: state [eventTime=11.53, mediaPos=10.60, window=0, period=0, false, READY]
EventLogger: state [eventTime=14.26, mediaPos=10.60, window=0, period=0, true, READY]
EventLogger: state [eventTime=131.89, mediaPos=128.27, window=0, period=0, true, ENDED]
```

方括号内的元素是：

- `[eventTime=float]`：创建播放器以来的时间。
- `[mediaPos=float]`：当前播放位置。
- `[window=int]`：当前窗口索引。
- `[period=int]`：该窗口中的当前时间段。
- `[boolean]`：`playWhenReady`标志。
- `[string]`：当前播放状态。

##### c) 媒体曲目

当可用或选定的曲目更改时，`EventLogger`将记录曲目信息。在播放开始时至少发生一次。以下示例显示了自适应流的跟踪记录：

```
EventLogger: tracksChanged [2.32, 0.00, window=0, period=0,
EventLogger:   Renderer:0 [
EventLogger:     Group:0, adaptive_supported=YES [
EventLogger:       [X] Track:0, id=133, mimeType=video/avc, bitrate=261112, codecs=avc1.4d4015, res=426x240, fps=30.0, supported=YES
EventLogger:       [X] Track:1, id=134, mimeType=video/avc, bitrate=671331, codecs=avc1.4d401e, res=640x360, fps=30.0, supported=YES
EventLogger:       [X] Track:2, id=135, mimeType=video/avc, bitrate=1204535, codecs=avc1.4d401f, res=854x480, fps=30.0, supported=YES
EventLogger:       [X] Track:3, id=160, mimeType=video/avc, bitrate=112329, codecs=avc1.4d400c, res=256x144, fps=30.0, supported=YES
EventLogger:       [X] Track:4, id=136, mimeType=video/avc, bitrate=2400538, codecs=avc1.4d401f, res=1280x720, fps=30.0, supported=YES
EventLogger:     ]
EventLogger:   ]
EventLogger:   Renderer:1 [
EventLogger:     Group:0, adaptive_supported=YES_NOT_SEAMLESS [
EventLogger:       [ ] Track:0, id=139, mimeType=audio/mp4a-latm, bitrate=48582, codecs=mp4a.40.5, channels=2, sample_rate=22050, supported=YES
EventLogger:       [X] Track:1, id=140, mimeType=audio/mp4a-latm, bitrate=127868, codecs=mp4a.40.2, channels=2, sample_rate=44100, supported=YES
EventLogger:     ]
EventLogger:   ]
EventLogger: ]
```

播放自适应流时，在播放过程中会记录正在播放格式的变化以及所选音轨的属性：

```
EventLogger: downstreamFormatChanged [3.64, 0.00, window=0, period=0, id=134, mimeType=video/avc, bitrate=671331, codecs=avc1.4d401e, res=640x360, fps=30.0]
EventLogger: downstreamFormatChanged [3.64, 0.00, window=0, period=0, id=140, mimeType=audio/mp4a-latm, bitrate=127868, codecs=mp4a.40.2, channels=2, sample_rate=44100]
```

##### d) 解码器选择

在大多数情况下，ExoPlayer使用`MediaCodec`从基础平台获取的媒体来渲染媒体。在报告任何播放状态之前，日志记录将告诉您已初始化了哪些解码器。例如：

```
EventLogger: decoderInitialized [0.77, 0.00, window=0, period=0, video, OMX.qcom.video.decoder.avc]
EventLogger: decoderInitialized [0.79, 0.00, window=0, period=0, audio, OMX.google.aac.decoder]
```



### 2.3 媒体来源 **[Media sources](https://exoplayer.dev/media-sources.html)**

在ExoPlayer中，每种媒体都由`MediaSource`标识。ExoPlayer库提供了`MediaSource`的几种流类型的实现：

- `DashMediaSource`用于[DASH](https://exoplayer.dev/dash.html)。
- `SsMediaSource`用于[SmoothStreaming](https://exoplayer.dev/smoothstreaming.html)。
- `HlsMediaSource`用于[HLS](https://exoplayer.dev/hls.html)。
- `ProgressiveMediaSource`用于[常规媒体文件](https://exoplayer.dev/progressive.html)。

`PlayerActivity`在[主演示应用程序中](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/)可以找到实例化这四个实例的[示例](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/)。

#### 2.3.1 MediaSource组成

除上述MediaSource，ExoPlayer库还提供`ConcatenatingMediaSource`，`ClippingMediaSource`， `LoopingMediaSource`和`MergingMediaSource`。这些`MediaSource` 实现可通过合成实现更复杂的播放功能。下面描述了一些常见的用例。注意，尽管以下一些示例是在视频播放的上下文中描述的，但它们同样适用于仅音频的播放，实际上也适用于任何支持的媒体类型的播放。

##### a) 播放播放列表

播放列表使用`ConcatenatingMediaSource`，该播放列表允许按顺序播放多个`MediaSource`。`ConcatenatingMediaSource`允许播放期间动态的添加和删除`MediaSource`。有关更多信息，请参见[播放列表页面](https://exoplayer.dev/playlists.html)。

##### b) 剪辑视频

`ClippingMediaSource`可用于剪辑`MediaSource`来仅播放其中的一部分。以下示例将视频播放剪辑为从5秒开始到10秒结束。

```java
MediaSource videoSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(videoUri);
// Clip to start at 5 seconds and end at 10 seconds.
ClippingMediaSource clippingSource =
    new ClippingMediaSource(
        videoSource,
        /* startPositionUs= */ 5_000_000,
        /* endPositionUs= */ 10_000_000);
```

要仅剪切源的开始，可以将`endPositionUs`设置为 `C.TIME_END_OF_SOURCE`。为了只剪辑特定的持续时间，可以使用带有`durationUs`参数的构造函数。

> 剪辑视频文件的开头时，请尽可能使起始位置与关键帧对齐。如果起始位置未与关键帧对齐，则播放器将需要解码并丢弃从前一个关键帧直到起始位置的数据，然后才能开始播放。这将在播放开始时引入短暂的延迟，包括当播放器将 `ClippingMediaSource`作为列表或循环播放时。

##### c) 循环播放视频

> 无限循环，推荐使用`ExoPlayer.setRepeatMode`而非 `LoopingMediaSource`。

可以使用`LoopingMediaSource`来将视频无缝循环固定次数 。以下示例将视频播放两次。

```java
MediaSource source =
    new ProgressiveMediaSource.Factory(...).createMediaSource(videoUri);
// Plays the video twice.
LoopingMediaSource loopingSource = new LoopingMediaSource(source, 2);
```

##### d) 侧面加载字幕文件

给定一个视频文件和一个单独的字幕文件，可以使用`MergingMediaSource`将它们合并为一个源以进行播放。

```java
// Build the video MediaSource.
MediaSource videoSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(videoUri);
// Build the subtitle MediaSource.
Format subtitleFormat = Format.createTextSampleFormat(
    id, // An identifier for the track. May be null.
    MimeTypes.APPLICATION_SUBRIP, // The mime type. Must be set correctly.
    selectionFlags, // Selection flags for the track.
    language); // The subtitle language. May be null.
MediaSource subtitleSource =
    new SingleSampleMediaSource.Factory(...)
        .createMediaSource(subtitleUri, subtitleFormat, C.TIME_UNSET);
// Plays the video with the sideloaded subtitle.
MergingMediaSource mergedSource =
    new MergingMediaSource(videoSource, subtitleSource);
```

#### 2.3.2 更多组合类型

可以进一步组合`MediaSource`以用于更特殊的用例。给定两个视频A和B，以下示例显示了如何 一起使用`LoopingMediaSource`和`ConcatenatingMediaSource`使他们成为播放序列（A，A，B）。

```java
MediaSource firstSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(firstVideoUri);
MediaSource secondSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(secondVideoUri);
// Plays the first video twice.
LoopingMediaSource firstSourceTwice = new LoopingMediaSource(firstSource, 2);
// Plays the first video twice, then the second video.
ConcatenatingMediaSource concatenatedSource =
    new ConcatenatingMediaSource(firstSourceTwice, secondSource);
```

以下示例是等效的，表明可以有多种方法来获得相同的结果。

```java
MediaSource firstSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(firstVideoUri);
MediaSource secondSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(secondVideoUri);
// Plays the first video twice, then the second video.
ConcatenatingMediaSource concatenatedSource =
    new ConcatenatingMediaSource(firstSource, firstSource, secondSource);
```



###  2.4 播放列表 [**Playlists**](https://exoplayer.dev/playlists.html)

使用`ConcatenatingMediaSource`实现播放列表，该播放列表允许按顺序播放多个`MediaSource`。以下示例表示由两个视频组成的播放列表。

```java
MediaSource firstSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(firstVideoUri);
MediaSource secondSource =
    new ProgressiveMediaSource.Factory(...).createMediaSource(secondVideoUri);
// Plays the first video, then the second video.
ConcatenatingMediaSource concatenatedSource =
    new ConcatenatingMediaSource(firstSource, secondSource);
```

播放列表中项目之间的转换是无缝的。不需要它们具有相同的格式（例如，播放列表中可以同时包含H264和VP9视频）。它们甚至可能具有不同的类型（例如，播放列表同时包含视频和音频流都可以）。允许`MediaSource`在播放列表中多次使用相同的内容。

#### 2.4.1 修改播放列表

`ConcatenatingMediaSource`允许播放期间动态的添加和删除`MediaSource`。可以在播放之前和播放期间通过调用对应的`ConcatenatingMediaSource` 方法来完成此操作。播放器会在播放过程中以正确的方式自动处理修改。例如，如果`MediaSource`移动了当前播放的文件，则播放不会中断，并且新的后继文件将在完成后播放。如果`MediaSource`删除了当前正在播放的播放器，则播放器将自动移动到播放剩余的第一个后继播放器，如果不存在此后继播放器，则过渡到结束状态。

#### 2.4.2 识别播放列表项

为了简化播放列表项的标识，`MediaSource`可以在工厂类中为每个项设置自定义标签。这可以是uri，标题或任何其他自定义对象。可以使用`player.getCurrentTag`查询当前播放项目的标签。调用`player.getCurrentTimeline`返回的当前`Timeline` 中也在`Timeline.Window`对象中包括了所有的标签 。

```java
public void addItem() {
  // Add mediaId (e.g. uri) as tag to the MediaSource.
  MediaSource mediaSource =
      new ProgressiveMediaSource.Factory(...)
          .setTag(mediaId)
          .createMediaSource(uri);
  concatenatedSource.addMediaSource(mediaSource);
}

@Override
public void onPositionDiscontinuity(@Player.DiscontinuityReason int reason) {
  // Load metadata for mediaId and update the UI.
  CustomMetadata metadata = CustomMetadata.get(player.getCurrentTag());
  titleView.setText(metadata.getTitle());
}
```

#### 2.4.3 检测播放何时过渡到其他项目

当前播放项目更改时可能有三种类型的事件回调：

1. `EventListener.onPositionDiscontinuity`中回调`reason = Player.DISCONTINUITY_REASON_PERIOD_TRANSITION`。当播放自动从一项过渡到另一项时，会触发此回调。
2. `EventListener.onPositionDiscontinuity`中回调`reason = Player.DISCONTINUITY_REASON_SEEK`。当当前播放项作为搜索操作的一部分而发生更改时（例如，在调用 `Player.next`时）。
3. `EventListener.onTimelineChanged`中回调`reason = Player.TIMELINE_CHANGE_REASON_DYNAMIC`。当播放列表发生更改时（例如，添加，移动或删除项目）。

在任何情况下，当您的应用程序代码接收到该事件时，您可以查询播放器以确定正在播放播放列表中的哪个项目。例如，可以使用`Player.getCurrentWindowIndex`和`Player.getCurrentTag`。如果您只想检测播放列表项的更改，则有必要与最新的已知窗口索引或标签进行比较，因为这些事件可能由于其他原因而触发。



### 2.5 轨道选择[**Track selection**](https://exoplayer.dev/track-selection.html)

轨道选择确定播放器播放哪些媒体轨道。每个`ExoPlayer`创建后都可以提供一个`TrackSelector`实例，用于实现轨道选择。

```java
DefaultTrackSelector trackSelector = new DefaultTrackSelector(context);
SimpleExoPlayer player =
    new SimpleExoPlayer.Builder(context)
        .setTrackSelector(trackSelector)
        .build();
```

`DefaultTrackSelector`是`TrackSelector`一种灵活实现，适用于大多数情况。使用`DefaultTrackSelector`可以通过修改`Parameters`来控制选择的轨道。可以在播放之前或播放期间完成。例如，以下代码告诉选择器将视频轨道选择限制为SD，并选择德语轨道（如果有的话）：

```java
trackSelector.setParameters(
    trackSelector
        .buildUponParameters()
        .setMaxVideoSizeSd()
        .setPreferredAudioLanguage("deu"));
```

这是基于约束的轨道选择的示例，其中在不了解实际可用轨道的情况下指定约束。可以使用`Parameters`来指定许多不同类型的约束。`Parameters` 也可用于从可用轨道中选择特定轨道。有关更多详细信息，请参见[`DefaultTrackSelector`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/trackselection/DefaultTrackSelector.html), [`Parameters`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/trackselection/DefaultTrackSelector.Parameters.html)和[`ParametersBuilder`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/trackselection/DefaultTrackSelector.ParametersBuilder.html)文档。



### 2.6 UI组件 [**UI components**](https://exoplayer.dev/ui-components.html)

应用程序播放媒体需要用于显示媒体和控制播放的用户界面组件。ExoPlayer库包括一个UI模块，其中包含许多UI组件。要依赖UI模块，请添加一个依赖项，如下所示，其中`2.X.X`是您的首选版本（最新版本可以参考[release notes](https://github.com/google/ExoPlayer/tree/release-v2/RELEASENOTES.md)）。

```groovy
implementation 'com.google.android.exoplayer:exoplayer-ui:2.X.X'
```

核心组件是`PlayerControlView`和`PlayerView`。

- [`PlayerControlView`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ui/PlayerControlView.html)是用于控制播放的视图。它显示标准的播放控件，包括播放/暂停按钮，快进和快退按钮以及搜索栏。
- [`PlayerView`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ui/PlayerView.html)是播放的高级视图。它在播放期间显示视频，字幕和专辑封面，以及使用播放控件 `PlayerControlView`。

这两个视图都具有`setPlayer`一种用于绑定和解绑（通过置为`null`）播放器实例的方法。

#### 2.6.1 PlayerView

`PlayerView`可用于视频和音频播放。在播放视频时，它会渲染视频和字幕，并可以显示作为元数据包含在音频文件中的图稿。您可以像其他任何UI组件一样在布局文件中添加`PlayerView`：

```xml
<com.google.android.exoplayer2.ui.PlayerView
    android:id="@+id/player_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:show_buffering="when_playing"
    app:show_shuffle_button="true"/>
```

上面的代码段说明了`PlayerView`提供几个属性。这些属性可用于自定义视图的行为以及其外观。这些属性中的大多数都有相应的setter方法，可用于在运行时自定义视图。查阅 [`PlayerView`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ui/PlayerView.html)的Javadoc文档来获得这些属性和setter方法的更多细节。

`PlayerView`在布局文件中声明后，就可以在Activity的`onCreate`动的方法中找到 ：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  // ...
  playerView = findViewById(R.id.player_view);
}
```

初始化播放器后，可以通过调用`setPlayer`将该播放器绑定到view ：

```java
// Instantiate the player.
player = new SimpleExoPlayer.Builder(context).build();
// Attach player to the view.
playerView.setPlayer(player);
// Prepare the player with the media source.
player.prepare(createMediaSource());
```

##### 选择surface type（~~这个词不知道该咋翻合适~~）

`PlayerView`的`surface_type`属性可让您设置用于视频播放的surface type。除了值`spherical_gl_surface_view`（用于球形视频~~(???这是个啥)~~播放一个特殊值）和 `video_decoder_gl_surface_view`（用于使用扩展渲染器渲染的视频），可选值包括`surface_view`，`texture_view`和`none`。

由于surface的开销很高，如果视图仅用于音频播放，应该选择`none	`而不应该创建surface。用于常规视频播放则应该使用`surface_view`或`texture_view` 。与`TextureView`相比，`SurfaceView`有许多优点：

+ 在许多设备上的功耗大大降低。
+ 更加精确的帧定时，从而使视频播放更加流畅。
+ 播放受DRM保护的内容时支持安全输出。
+ 能够在可扩展UI层的Android TV设备上以全分辨率显示视频内容。

因此，在可能的情况下`SurfaceView`比`TextureView`应该优先使用。 `TextureView`仅在`SurfaceView`不满足您的需求时使用。例如是在Android N之前需要平滑的动画或视频表面滚动，如下所述。对于这种情况，仍然应该仅在[`SDK_INT`](https://developer.android.com/reference/android/os/Build.VERSION.html#SDK_INT)小于24（Android N）时使用`TextureView`， 否则使用`SurfaceView`。

> 在Android N之前，`SurfaceView`渲染还不能与视图动画正确同步。在早期版本中，将 `SurfaceView`放入滚动容器中或对其进行动画处理时，这可能会导致不良效果 。意外效果包括视图的内容似乎稍微滞后于应显示的位置，并且在进行动画处理后视图变为黑色。为了在Android N之前实现平滑的动画或视频滚动，因此必须使用`TextureView`而不是`SurfaceView`。

> 某些Android TV设备以低于显示器全分辨率的分辨率运行在其UI层，从而将其放大以呈现给用户。例如，UI层可以在具有4K显示屏的Android TV上以1080p的分辨率运行。在此类设备上，`SurfaceView`必须用于以全分辨率显示内容。可以使用[`Util.getPhysicalDisplaySize`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/util/Util.html#getPhysicalDisplaySize-android.content.Context-)来查询显示器的全分辨率（在其当前显示模式下）。可以使用Android的[`Display.getSize`](https://developer.android.com/reference/android/view/Display.html#getSize(android.graphics.Point))API 查询UI层分辨率。

#### 2.6.2 PlayerControlView

`PlayerView`内部使用`PlayerControlView`提供播放控制。对于特定的用例`PlayerControlView`，也可以用作独立组件。它可以像其他任何UI组件一样添加至布局文件中：

```xml
<com.google.android.exoplayer2.ui.PlayerControlView
    android:id="@+id/player_control_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

与一样`PlayerView`，[`PlayerControlView`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ui/PlayerControlView.html)Javadoc更详细地记录了可用的属性和setter方法。类似于`PlayerView`，使用以下代码找到`PlayerControlView`并将播放器附加到view：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  // ...
  playerControlView = findViewById(R.id.player_control_view);
}

private void initializePlayer() {
  // Instantiate the player.
  player = new SimpleExoPlayer.Builder(context).build();
  // Attach player to the view.
  playerControlView.setPlayer(player);
  // Prepare the player with the dash media source.
  player.prepare(createMediaSource());
}
```

#### 2.6.3 定制

在需要大量自定义的地方，我们希望应用程序开发人员将实现自己的UI组件，而不是使用ExoPlayer的UI模块提供的组件。也就是说，所提供的UI组件确实可以通过设置属性（如上所述），覆盖可绘制对象，覆盖布局文件以及指定自定义布局文件来进行自定义。

##### 覆盖drawables

`PlayerControlView`（与它的默认布局文件）使用的 drawables 可以通过在你的应用程序中定义的名称相同的绘图资源覆盖。有关[`PlayerControlView`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ui/PlayerControlView.html)可覆盖的可绘制对象的列表，请参见 Javadoc。由于`PlayerView`使用`PlayerControlView`，因此覆盖这些可绘制对象`PlayerView`也适用。

##### 覆盖布局文件

当`PlayerView`从布局文件`exo_player_view.xml`加载；`PlayerControlView`从布局文件 `exo_player_control_view.xml`加载。要自定义这些布局，应用程序可以在其自己的`res/layout*`目录中定义具有相同名称的布局文件。这些布局文件将覆盖ExoPlayer库提供的文件。

例如，假设我们希望播放控件仅由位于视图中心的播放/暂停按钮组成。我们可以通过在应用程序的`res/layout` 目录中创建文件`exo_player_control_view.xml`来实现此目的，该文件包含：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <ImageButton android:id="@id/exo_play"
      android:layout_width="100dp"
      android:layout_height="100dp"
      android:layout_gravity="center"
      android:background="#CC000000"
      style="@style/ExoMediaButton.Play"/>

  <ImageButton android:id="@id/exo_pause"
      android:layout_width="100dp"
      android:layout_height="100dp"
      android:layout_gravity="center"
      android:background="#CC000000"
      style="@style/ExoMediaButton.Pause"/>

</FrameLayout>
```

与标准控件相比，视觉外观的变化如下所示。

![将标准播放控件（左）替换为自定义控件（右）](https://exoplayer.dev/images/overriding-layoutfiles.png)

##### 自定义布局文件

如果想在整个应用程序中更改布局，覆盖布局文件是绝佳的解决方案，但是如果仅在单个位置需要自定义布局怎么办？为此，首先定义一个布局文件，就像覆盖默认布局之一一样，但是这次给它一个不同的文件名，例如`custom_controls.xml`。其次，使用属性指示在放大视图时应使用此布局。例如，当使用时 `PlayerView`，可以使用`controller_layout_id`属性指定加载布局以提供播放控件：

```xml
<com.google.android.exoplayer2.ui.PlayerView android:id="@+id/player_view"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     app:controller_layout_id="@layout/custom_controls"/>
```



### 2.7 下载媒体 [Downloading media](https://exoplayer.dev/downloading-media.html)

ExoPlayer提供了下载媒体功能以供离线播放。在大多数使用情况下，即使您的应用程序处于后台，也希望继续下载。对于这些用例，您的应用程序应为`DownloadService`的子类，并将命令发送至服务以添加，删除和控制下载。下图显示了涉及的主要类。

![用于下载媒体的类。 箭头方向指示数据流。](https://exoplayer.dev/images/downloading.svg)

- `DownloadService`：包装 `DownloadManager`并将命令转发给它。该服务允许`DownloadManager`即使应用在后台运行也可以继续运行。
- `DownloadManager`：管理多个下载，从`DownloadIndex`加载（和存储）它们的状态，并根据网络连接等要求开始和停止下载。要下载内容，管理员通常会从中读取正在下载的数据 `HttpDataSource`，然后将其写入中`Cache`。
- `DownloadIndex`：保留下载状态。

#### 2.7.1 创建DownloadService

要创建`DownloadService`，您需要对其进行子类化并实现其抽象方法：

- `getDownloadManager()`：返回要使用的`DownloadManager`。
- `getScheduler()`：返回可选`Scheduler`，当满足待完成的下载进度所需的需求时，可以重新启动服务。ExoPlayer提供了以下实现：
  - `PlatformScheduler`，使用[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler)（最小API为21）。有关应用程序权限要求，请参见[PlatformScheduler](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/scheduler/PlatformScheduler.html) javadocs。
  - `WorkManagerScheduler`，使用[WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager/)。
  - `JobDispatcherScheduler`，使用[Firebase JobDispatcher](https://github.com/firebase/firebase-jobdispatcher-android) （不建议使用）。有关应用程序权限要求，请参见[JobDispatcherScheduler](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ext/jobdispatcher/JobDispatcherScheduler.html) javadocs。
- `getForegroundNotification()`：返回服务在前台运行时要显示的通知。您可以用来 `DownloadNotificationHelper.buildProgressNotification`以默认样式创建通知。

最后，您需要在`AndroidManifest.xml`文件中定义服务：

```xml
<service android:name="com.myapp.MyDownloadService"
    android:exported="false">
  <!-- This is needed for Scheduler -->
  <intent-filter>
    <action android:name="com.google.android.exoplayer.downloadService.action.RESTART"/>
    <category android:name="android.intent.category.DEFAULT"/>
  </intent-filter>
</service>
```

有关具体示例，请在ExoPlayer演示应用程序中查看[`DemoDownloadService`](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/DemoDownloadService.java)和[`AndroidManifest.xml`](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/AndroidManifest.xml)。

#### 2.7.2 创建DownloadManager

以下代码段演示了如何实例化`DownloadManager`，随后可以在`DownloadService`调用`getDownloadManager()`获取到它：

```java
// Note: This should be a singleton in your app.
databaseProvider = new ExoDatabaseProvider(context);

// A download cache should not evict media, so should use a NoopCacheEvictor.
downloadCache = new SimpleCache(
    downloadDirectory,
    new NoOpCacheEvictor(),
    databaseProvider);

// Create a factory for reading the data from the network.
dataSourceFactory = new DefaultHttpDataSourceFactory(userAgent);

// Create the download manager.
downloadManager = new DownloadManager(
    context,
    databaseProvider,
    downloadCache,
    dataSourceFactory);

// Optionally, setters can be called to configure the download manager.
downloadManager.setRequirements(requirements);
downloadManager.setMaxParallelDownloads(3);
```

有关[`DemoApplication`](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/DemoApplication.java)具体示例，请参见演示应用程序。

> 演示应用程序中的示例从旧版`ActionFile`实例导入下载状态。仅在您的应用程序使用2.10.0版本之前的ExoPlayer 中使用`ActionFile`时才需要这样做。

#### 2.7.3 添加下载

要添加下载，您需要创建一个`DownloadRequest`并将其发送到 `DownloadService`。对于自适应流，`DownloadHelper`可以使用它来帮助构建 `DownloadRequest`，如[本页下所述](https://exoplayer.dev/downloading-media.html#downloading-and-playing-adaptive-streams)。以下示例展示了如何为渐进式流创建下载请求：

```java
DownloadRequest downloadRequest = new DownloadRequest(
    contentId,
    DownloadRequest.TYPE_PROGRESSIVE,
    contentUri,
    /* streamKeys= */ Collections.emptyList(),
    /* customCacheKey= */ null,
    appData);
```

其中`contentId`是内容的唯一标识符，`appData`是应用程序希望与下载相关联的任何数据。在简单的情况下， `contentUri`通常可以将用作`contentId`，但是应用程序可以自由使用最适合其用例的ID方案。

创建完成后，可以将请求发送到`DownloadService`以添加下载：

```java
DownloadService.sendAddDownload(
    context,
    MyDownloadService.class,
    downloadRequest,
    /* foreground= */ false)
```

`MyDownloadService`是应用程序`DownloadService`的子类，其中`foreground`参数控制服务是否将在前台启动。如果您的应用程序已经在前台，则`foreground` 通常应将该参数设置为`false`，因为`DownloadService`如果它确定有工作要做，则会将自己置于前台。

#### 2.7.4 移除下载

一个下载可以通过发送一个删除命令到被移除`DownloadService`，其中，`contentId`标识要移除的下载：

```java
DownloadService.sendRemoveDownload(
    context,
    MyDownloadService.class,
    contentId,
    /* foreground= */ false)
```

您也可以使用删除所有下载的数据 `DownloadService.sendRemoveAllDownloads`。

#### 2.7.5 开始和停止下载

如果满足以下四个条件，则下载将持续进行：

- 下载没有停止原因（请参见下文）。
- 下载没有暂停。
- 满足持续下载的要求。需求可以指定对允许的网络类型的限制，以及设备应处于空闲状态还是连接至充电器。
- 不超过并行下载的最大数量。

通过向您的`DownloadService`发送命令，可以控制所有这些条件 。

##### 设置和清除下载停止原因

可以为单个或全部下载设置被停止的原因：

```java
// Set the stop reason for a single download.
DownloadService.sendSetStopReason(
    context,
    MyDownloadService.class,
    contentId,
    stopReason,
    /* foreground= */ false);

// Clear the stop reason for a single download.
DownloadService.sendSetStopReason(
    context,
    MyDownloadService.class,
    contentId,
    Download.STOP_REASON_NONE,
    /* foreground= */ false);
```

其中`stopReason`可以是任何非零值（`Download.STOP_REASON_NONE = 0`是一个特殊值，表示不停止下载）。拥有复杂停止下载原因的应用可以设置不同的值来跟踪停止原因。设置和清除所有下载的停止原因与设置和清除单个下载的停止原因的工作方式相同，即将`contentId`设置为`null`。

当下载具有非零停止原因时，它将处于 `Download.STATE_STOPPED`状态。

##### 暂停和恢复所有下载

可以按以下方式暂停和恢复所有下载：

```java
// Pause all downloads.
DownloadService.sendPauseDownloads(
    context,
    MyDownloadService.class,
    /* foreground= */ false);

// Resume all downloads.
DownloadService.sendResumeDownloads(
    context,
    MyDownloadService.class,
    /* foreground= */ false);
```

暂停下载后，它将处于`Download.STATE_QUEUED`状态。

##### 设置下载进度的要求

[`Requirements`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/scheduler/Requirements.html)可以用于指定下载必须满足的约束。如上例[所示](https://exoplayer.dev/downloading-media.html#creating-a-downloadmanager)，可以在创建`DownloadManager`通过调用`DownloadManager.setRequirements()`来设置要求 。也可以通过将命令发送到`DownloadService`来动态更改它们：

```java
// Set the download requirements.
DownloadService.sendSetRequirements(
    context,
    MyDownloadService.class,
    requirements,
    /* foreground= */ false);
```

当由于不满足要求而无法进行下载时，它将处于`Download.STATE_QUEUED`状态。您可以使用`DownloadManager.getNotMetRequirements()`查询未满足的要求的下载。

##### 设置最大并行下载数

可以通过调用`DownloadManager.setMaxParallelDownloads()`设置最大并行下载数 。如上例[所示](https://exoplayer.dev/downloading-media.html#creating-a-downloadmanager)，通常在创建`DownloadManager`时完成此操作。

当由于达到并行下载的最大数量而无法进行下载时，它将处于此`Download.STATE_QUEUED`状态。

#### 2.7.6 查询下载

每个`DownloadManager`中都包含`DownloadIndex`用于可以查询所有下载的状态，包括那些已完成或失败的状态。可以通过调用`DownloadManager.getDownloadIndex()`来获得`DownloadIndex`。然后，可以通过调用`DownloadIndex.getDownloads()`遍历获取所有下载 。或者，可以通过调用`DownloadIndex.getDownload()`查询单个下载的状态。

`DownloadManager`还提供了`DownloadManager.getCurrentDownloads()`，它仅返回当前（即未完成或失败）下载的状态。此方法对于更新通知和其他显示当前下载进度和状态的UI组件很有用。

#### 2.7.7 监听下载

您可以为`DownloadManager`添加一个侦听器，以在当前下载更改状态时得到通知：

```java
downloadManager.addListener(
    new DownloadManager.Listener() {
      // Override methods of interest here.
    });
```

有关具体示例，请参见`DownloadManagerListener`演示应用程序的[`DownloadTracker`](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/DownloadTracker.java)类。

> 下载进度更新不会触发`DownloadManager.Listener`。如果您需要更新显示下载进度的UI组件，您应该以所需的更新速率定期查询`DownloadManager`。[`DownloadService`](https://github.com/google/ExoPlayer/tree/release-v2/library/core/src/main/java/com/google/android/exoplayer2/offline/DownloadService.java) 包含一个示例，该示例会定期更新前台服务通知。

#### 2.7.8 播放下载的内容

播放下载的内容类似于播放在线内容，区别是将从下载`Cache`而不是通过网络读取数据。

> 请注意，请勿尝试直接从下载目录读取文件。而是使用ExoPlayer库类，如下所述。

要播放下载的内容，使用与下载时相同的 `Cache`实例创建一个`CacheDataSourceFactory`。使用该工厂，构造一个`MediaSource`用于播放的文件。您应该使用原始文件`contentUri`（即从中下载内容的文件）而不是指向下载目录或其中的任何文件的URI来构建`MediaSource`。

```java
CacheDataSourceFactory dataSourceFactory = new CacheDataSourceFactory(
    downloadCache, upstreamDataSourceFactory);
ProgressiveMediaSource mediaSource = new ProgressiveMediaSource
    .Factory(dataSourceFactory)
    .createMediaSource(contentUri);
player.prepare(mediaSource);
```

#### 2.7.9 下载和播放自适应流

自适应流（例如DASH，SmoothStreaming和HLS）通常包含多个媒体轨道。通常会有多条轨道包含质量不同的相同内容（例如SD，HD和4K视频轨道）。也可能有多个相同类型的轨道包含不同的内容（例如，多种语言的音频轨道）。

对于流式播放，可以使用曲目选择器选择播放哪些曲目。同样，对于下载的自适应流，可以使用`DownloadHelper`选择要下载的曲目。`DownloadHelper`的典型用法如下：

1. 使用`DownloadHelper.forXXX`方法构建`DownloadHelper`。

2. 使用`prepare(DownloadHelper.Callback)`初始化`helper`并等待回调。

   ```java
   DownloadHelper downloadHelper =
       DownloadHelper.forDash(
           context,
           contentUri,
           dataSourceFactory,
           new DefaultRenderersFactory(context));
   downloadHelper.prepare(myCallback);
   ```

3. 可以使用`getMappedTrackInfo`和`getTrackSelections`检查默认选定的曲目 ，使用`clearTrackSelections`， `replaceTrackSelections`和`addTrackSelection`进行调整。

4. 通过调用`getDownloadRequest`为选定的曲目创建`DownloadRequest`。如上所述，可以将请求传递给您`DownloadService`以添加下载。

5. 使用`release()`释放helper。

您可以通过调用`DownloadHelper.createMediaSource`以下命令创建`MediaSource`来播放 ：

```java
MediaSource mediaSource =
    DownloadHelper.createMediaSource(downloadRequest, dataSourceFactory);
```

创建的文件`MediaSource`会知道已下载了哪些曲目，因此只会尝试在播放期间使用这些曲目。有关[`PlayerActivity`](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/PlayerActivity.java) 具体示例，请参见演示应用程序。



### 2.8 [Ad insertion](https://exoplayer.dev/ad-insertion.html)

略