[开发者指南原文](https://exoplayer.dev/)

# 3. 媒体类型

## 3.1 [DASH](https://exoplayer.dev/dash.html)

ExoPlayer支持多种文件格式的DASH。必须对媒体流进行解复用，这意味着DASH清单中需要用不同AdaptationSet元素定义视频，音频和文本（CEA-608是一个例外，如下表所述）。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [样本格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。

| Feature                       | Supported | Comment                                                      |
| ----------------------------- | :-------: | :----------------------------------------------------------- |
| **Containers**                |           |                                                              |
| FMP4                          |    YES    | Demuxed streams only                                         |
| WebM                          |    YES    | Demuxed streams only                                         |
| Matroska                      |    YES    | Demuxed streams only                                         |
| MPEG-TS                       |    NO     | No support planned                                           |
| **Closed captions/subtitles** |           |                                                              |
| TTML                          |    YES    | Raw, or embedded in FMP4 according to ISO/IEC 14496-30       |
| WebVTT                        |    YES    | Raw, or embedded in FMP4 according to ISO/IEC 14496-30       |
| CEA-608                       |    YES    | Carried in SEI messages embedded in FMP4 video streams       |
| **Metadata**                  |           |                                                              |
| EMSG metadata                 |    YES    | Embedded in FMP4                                             |
| **Content protection**        |           |                                                              |
| Widevine                      |    YES    | API 19+ (“cenc” scheme) and 25+ (“cbcs”, “cbc1” and “cens” schemes) |
| PlayReady SL2000              |    YES    | Android TV only                                              |
| ClearKey                      |    YES    | API 21+                                                      |

### 3.1.1 创建 MediaSource

要播放DASH流，需要创建`DashMediaSource`并像往常一样创建并准备播放器。

```java
// Create a data source factory.
DataSource.Factory dataSourceFactory =
    new DefaultHttpDataSourceFactory(Util.getUserAgent(context, "app-name"));
// Create a DASH media source pointing to a DASH manifest uri.
MediaSource mediaSource = new DashMediaSource.Factory(dataSourceFactory)
    .createMediaSource(dashUri);
// Create a player instance.
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
// Prepare the player with the media source.
player.prepare(mediaSource);
```

考虑到可用带宽和设备功能，ExoPlayer将自动在清单中定义的表示方式之间进行调整。

### 3.1.2 访问manifest

您可以通过`Player.getCurrentManifest`调用检索当前清单。对于DASH，您应该将返回的对象转换为`DashManifest`。每当加载清单时，也会触发`Player.EventListener`的 `onTimelineChanged`回调。对于点播内容，这将发生一次，而对于直播内容，则可能会发生多次。下面的代码片段显示了清单加载后应用程序如何执行操作。

```java
player.addListener(
    new Player.EventListener() {
      @Override
      public void onTimelineChanged(
          Timeline timeline, @Player.TimelineChangeReason int reason) {
        Object manifest = player.getCurrentManifest();
        if (manifest != null) {
          DashManifest dashManifest = (DashManifest) manifest;
          // Do something with the manifest.
        }
      }
    });
```

### 3.1.3 Sideloading 清单

对于特定的用例可以使用另外一种方法获得清单，将`DashManifest`对象传递给构造函数。

```java
DataSource.Factory dataSourceFactory =
    new DefaultHttpDataSourceFactory(Util.getUserAgent(context, "app-name"));
// Create a dash media source with a dash manifest.
MediaSource mediaSource = new DashMediaSource.Factory(dataSourceFactory)
    .createMediaSource(dashManifest);
// Create a player instance which gets an adaptive track selector by default.
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
// Prepare the player with the media source.
player.prepare(mediaSource);
```

### 3.1.4 自定义DASH播放

ExoPlayer为您提供了多种方式来根据您的应用程序定制播放体验。以下各节简要介绍了构建`DashMediaSource`时可用的一些自定义选项。有关更多常规自定义选项，请参见“ [自定义”页面](https://exoplayer.dev/customization.html)。

#### 自定义服务器交互

某些应用可能想要拦截HTTP请求和响应。您可能想要注入自定义请求标头，读取服务器的响应标头，修改请求的URI等。例如，您的应用程序可以通过在请求媒体段时注入令牌作为标头来对自身进行身份验证。

下面的例子演示了通过将自定义[HttpDataSources](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/HttpDataSource.html)注入`DashMediaSource`来实现这些行为：

```java
DashMediaSource dashMediaSource =
    new DashMediaSource.Factory(
            () -> {
              HttpDataSource dataSource = new DefaultHttpDataSource(userAgent);
              // Set a custom authentication request header.
              dataSource.setRequestProperty("Header", "Value");
              return dataSource;
            })
        .createMediaSource(dashUri);
```

在上面的代码段中，由`dashMediaSource`触发的HTTP请求，都通过注入的`HttpDataSource`中`"Header: Value"` 添加了请求头。对于与`dashMediaSource`的每次HTTP交互，此行为都是*固定*的。

对于更精细的方法，您可以使用`ResolvingDataSource`来注入即时行为 。以下代码段显示了如何在与HTTP源进行交互之前注入请求标头：

```java
DashMediaSource dashMediaSource =
    new DashMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time request headers.
            (DataSpec dataSpec) ->
                dataSpec.withRequestHeaders(getCustomHeaders(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

您还可以使用`ResolvingDataSource` 来对URI进行即时修改，如以下代码片段所示：

```
DashMediaSource dashMediaSource =
    new DashMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time URI resolution logic.
            (DataSpec dataSpec) -> dataSpec.withUri(resolveUri(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

#### 自定义错误处理

实现自定义[LoadErrorHandlingPolicy](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/LoadErrorHandlingPolicy.html)可使应用程序自定义ExoPlayer对加载错误的处理方式。例如，某个应用可能想要快速失败而不是多次重试，或者可能想要自定义退避逻辑，以控制播放器在每次重试之间等待多长时间。以下代码段显示了创建`DashMediaSource`时如何实现自定义退避逻辑：

```java
DashMediaSource dashMediaSource =
    new DashMediaSource.Factory(dataSourceFactory)
        .setLoadErrorHandlingPolicy(
            new DefaultLoadErrorHandlingPolicy() {
              @Override
              public long getRetryDelayMsFor(...) {
                // Implement custom back-off logic here.
              }
            })
        .createMediaSource(dashUri);
```

您可以在我们的[Medium post about error handling](https://medium.com/google-exoplayer/load-error-handling-in-exoplayer-488ab6908137)找到更多信息。

### 3.1.5 BehindLiveWindowException

在播放具有有限可用性的实时流时，如果播放器暂停或缓冲了足够长的时间，则播放器可能会落在此实时窗口后面。在这种情况下，将抛出`BehindLiveWindowException`异常，可以将其捕获以在实时边缘恢复播放器。演示应用程序的[PlayerActivity](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/PlayerActivity.java)就是这种方法的例证。

```java
@Override
public void onPlayerError(ExoPlaybackException e) {
  if (isBehindLiveWindow(e)) {
    // Re-initialize player at the live edge.
  } else {
    // Handle other errors
  }
}

private static boolean isBehindLiveWindow(ExoPlaybackException e) {
  if (e.type != ExoPlaybackException.TYPE_SOURCE) {
    return false;
  }
  Throwable cause = e.getSourceException();
  while (cause != null) {
    if (cause instanceof BehindLiveWindowException) {
      return true;
    }
    cause = cause.getCause();
  }
  return false;
}
```



## 3.2 [HLS](https://exoplayer.dev/hls.html)

ExoPlayer支持多种文件格式的HLS。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [样本格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。我们强烈鼓励HLS内容制作者产生高品质的HLS流，详见 [这里](https://medium.com/google-exoplayer/hls-playback-in-exoplayer-a33959a47be7)。

| Feature                       | Supported | Comment                                         |
| ----------------------------- | :-------: | :---------------------------------------------- |
| **Containers**                |           |                                                 |
| MPEG-TS                       |    YES    |                                                 |
| FMP4/CMAF                     |    YES    |                                                 |
| ADTS (AAC)                    |    YES    |                                                 |
| MP3                           |    YES    |                                                 |
| **Closed captions/subtitles** |           |                                                 |
| CEA-608                       |    YES    |                                                 |
| WebVTT                        |    YES    |                                                 |
| **Metadata**                  |           |                                                 |
| ID3 metadata                  |    YES    |                                                 |
| **Content protection**        |           |                                                 |
| AES-128                       |    YES    |                                                 |
| Sample AES-128                |    NO     |                                                 |
| Widevine                      |    YES    | API 19+ (“cenc” scheme) and 25+ (“cbcs” scheme) |
| PlayReady SL2000              |    YES    | Android TV only                                 |

### 3.2.1 创建一个MediaSource

要播放HLS流，请创建`HlsMediaSource`，并按照正常流程准备播放器。

```java
// Create a data source factory.
DataSource.Factory dataSourceFactory =
    new DefaultHttpDataSourceFactory(Util.getUserAgent(context, "app-name"));
// Create a HLS media source pointing to a playlist uri.
HlsMediaSource hlsMediaSource =
    new HlsMediaSource.Factory(dataSourceFactory).createMediaSource(uri);
// Create a player instance.
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
// Prepare the player with the media source.
player.prepare(hlsMediaSource);
```

传递给`HlsMediaSource.Factory.createMediaSource()`的URI可能指向媒体播放列表或主播放列表。如果URI指向声明了多个`#EXT-X-STREAM-INF`标签的主播放列表，则ExoPlayer将自动在变体之间进行调整，同时考虑可用带宽和设备功能。

### 3.2.3 访问 manifest

您可以通过调用`Player.getCurrentManifest`检索当前清单。对于HLS，您应该将返回的对象强制转换为`HlsManifest`。每当加载清单时，也会触发`Player.EventListener`的 `onTimelineChanged`回调。对于点播内容，这将发生一次，而对于直播内容，则可能会发生多次。下面的代码片段显示了清单加载后应用程序如何执行操作。

```java
player.addListener(
    new Player.EventListener() {
      @Override
      public void onTimelineChanged(
          Timeline timeline, @Player.TimelineChangeReason int reason) {
        Object manifest = player.getCurrentManifest();
        if (manifest != null) {
          HlsManifest hlsManifest = (HlsManifest) manifest;
          // Do something with the manifest.
        }
      }
    });
```

### 3.2.4 自定义HLS播放

ExoPlayer为您提供了多种方式来根据您的应用程序定制播放体验。以下各节简要介绍了构建`HlsMediaSource`时可用的一些自定义选项。有关更多常规自定义选项，请参见“ [自定义”页面](https://exoplayer.dev/customization.html)。

#### 更快的启动时间

通过启用无块准备，可以显著缩短HLS启动时间。当启用无块准备并且`#EXT-X-STREAM-INF`标签包含该 `CODECS`属性时，ExoPlayer将避免下载媒体段作为准备的一部分。以下代码段显示了如何启用无块准备：

```java
HlsMediaSource hlsMediaSource =
    new HlsMediaSource.Factory(dataSourceFactory)
        .setAllowChunklessPreparation(true)
        .createMediaSource(hlsUri);
```

您可以在我们的[有关无块准备的文章](https://medium.com/google-exoplayer/faster-hls-preparation-f6611aa15ea6)找到更多详细信息。

#### 自定义服务器交互

某些应用可能想要拦截HTTP请求和响应。您可能想要注入自定义请求标头，读取服务器的响应标头，修改请求的URI等。例如，您的应用程序可以通过在请求媒体段时注入token作为标头来对自身进行身份验证。

下面的例子演示了如何通过将自定义[HttpDataSources](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/HttpDataSource.html)注入到`HlsMediaSource`来实现这些行为：

```java
HlsMediaSource hlsMediaSource =
    new HlsMediaSource.Factory(
            dataType -> {
              HttpDataSource dataSource =
                  new DefaultHttpDataSource(userAgent);
              if (dataType == C.DATA_TYPE_MEDIA) {
                // The data source will be used for fetching media segments. We
                // set a custom authentication request header.
                dataSource.setRequestProperty("Header", "Value");
              }
              return dataSource;
            })
        .createMediaSource(hlsUri);
```

在上面的代码段中，由`hlsMediaSource`触发的HTTP请求，都通过注入的`HttpDataSource`中`"Header: Value"` 添加了请求头。对于与`hlsMediaSource`的每次HTTP交互，此行为都是*固定*的。

对于更精细的方法，您可以使用`ResolvingDataSource`来注入即时行为 。以下代码段显示了如何在与HTTP源进行交互之前注入请求标头：

```java
HlsMediaSource hlsMediaSource =
    new HlsMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time request headers.
            (DataSpec dataSpec) ->
                dataSpec.withRequestHeaders(getCustomHeaders(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

您还可以使用`ResolvingDataSource` 来对URI进行即时修改，如以下代码片段所示：

```java
HlsMediaSource hlsMediaSource =
    new HlsMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time URI resolution logic.
            (DataSpec dataSpec) -> dataSpec.withUri(resolveUri(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

#### 自定义错误处理

实施自定义[LoadErrorHandlingPolicy](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/LoadErrorHandlingPolicy.html)可使应用程序自定义ExoPlayer对加载错误的反应方式。例如，某个应用可能想要快速失败而不是多次重试，或者可能想要自定义退避逻辑，以控制播放器在每次重试之间等待多长时间。以下代码段显示了创建时如何实现自定义退避逻辑`HlsMediaSource`：

```java
HlsMediaSource hlsMediaSource =
    new HlsMediaSource.Factory(dataSourceFactory)
        .setLoadErrorHandlingPolicy(
            new DefaultLoadErrorHandlingPolicy() {
              @Override
              public long getRetryDelayMsFor(...) {
                // Implement custom back-off logic here.
              }
            })
        .createMediaSource(hlsUri);
```

您可以在我们的[有关错误处理的文章](https://medium.com/google-exoplayer/load-error-handling-in-exoplayer-488ab6908137)中获取更多信息。

### 3.2.5 创建高质量的HLS内容

为了充分利用ExoPlayer，可以遵循某些准则来改进HLS内容。阅读[有关ExoPlayer中HLS播放的文章](https://medium.com/google-exoplayer/hls-playback-in-exoplayer-a33959a47be7)，以获取完整说明。要点如下：

- 使用精确的段持续时间。
- 使用连续媒体流；避免跨段的媒体结构发生变化。
- 使用`#EXT-X-INDEPENDENT-SEGMENTS`标签。
- 与包含视频和音频的文件相比，更推荐使用解复用的流。
- 在“主播放列表”中包括所有可以包含的信息。

以下准则专门适用于实时流：

- 使用`#EXT-X-PROGRAM-DATE-TIME`标签。
- 使用`#EXT-X-DISCONTINUITY-SEQUENCE`标签。
- 提供一个长生命周期的窗口。最好能够达到一分钟或更长时间。

# BehindLiveWindowException

如果播放了具有有限可用性的实时流，播放器暂停或缓冲了足够长的时间，则播放器可能会落在此实时窗口后面。在这种情况下，将抛出`BehindLiveWindowException`，可以将其捕获以在实时边缘恢复播放器。演示应用程序的[PlayerActivity](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/PlayerActivity.java)就是这种方法的例证。

```java
@Override
public void onPlayerError(ExoPlaybackException e) {
  if (isBehindLiveWindow(e)) {
    // Re-initialize player at the live edge.
  } else {
    // Handle other errors
  }
}

private static boolean isBehindLiveWindow(ExoPlaybackException e) {
  if (e.type != ExoPlaybackException.TYPE_SOURCE) {
    return false;
  }
  Throwable cause = e.getSourceException();
  while (cause != null) {
    if (cause instanceof BehindLiveWindowException) {
      return true;
    }
    cause = cause.getCause();
  }
  return false;
}
```



## 3.3 [SmoothStreaming](https://exoplayer.dev/smoothstreaming.html)

ExoPlayer支持FMP4容器格式的SmoothStreaming。必须对媒体流进行解复用，这意味着必须在SmoothStreaming清单的不同StreamIndex元素中定义视频，音频和文本。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [样本格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。

| Feature                       | Supported | Comment              |
| ----------------------------- | :-------: | :------------------- |
| **Containers**                |           |                      |
| FMP4                          |    YES    | Demuxed streams only |
| **Closed captions/subtitles** |           |                      |
| TTML                          |    YES    | Embedded in FMP4     |
| **Content protection**        |           |                      |
| PlayReady SL2000              |    YES    | Android TV only      |

### 3.3.1 创建MediaSource

要播放SmoothStreaming流，请创建`SsMediaSource`并像平常创建并准备播放器。

```java
// Create a data source factory.
DataSource.Factory dataSourceFactory =
    new DefaultHttpDataSourceFactory(Util.getUserAgent(context, "app-name"));
// Create a SmoothStreaming media source pointing to a manifest uri.
MediaSource mediaSource =
    new SsMediaSource.Factory(dataSourceFactory).createMediaSource(ssUri);
// Create a player instance.
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
// Prepare the player with the media source.
player.prepare(mediaSource);
```

考虑到可用带宽和设备功能，ExoPlayer将自动在清单中定义的流之间进行调整。

### 3.3.2 访问manifest

您可以通过调用`Player.getCurrentManifest`检索当前清单。对于SmoothStreaming，应将返回的对象转换为`SsManifest`。每当加载清单时，也会触发`Player.EventListener`的 `onTimelineChanged`回调。对于点播内容，这将发生一次，而对于直播内容，则可能会发生多次。下面的代码片段显示了清单加载后应用程序如何执行操作。

```java
player.addListener(
    new Player.EventListener() {
      @Override
      public void onTimelineChanged(
          Timeline timeline, @Player.TimelineChangeReason int reason) {
        Object manifest = player.getCurrentManifest();
        if (manifest != null) {
          SsManifest ssManifest = (SsManifest) manifest;
          // Do something with the manifest.
        }
      }
    });
```

### 3.3.3 Sideloading清单

对于特定的用例，可以通过将`SsManifest`对象传递给构造函数获取清单。

```java
DataSource.Factory dataSourceFactory =
    new DefaultHttpDataSourceFactory(Util.getUserAgent(context, "app-name"));
// Create a smooth streaming media source with a smooth streaming  manifest.
MediaSource mediaSource =
    new SsMediaSource.Factory(dataSourceFactory).createMediaSource(ssManifest);
// Create a player instance which gets an adaptive track selector by default.
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
// Prepare the player with the media source.
player.prepare(mediaSource);
```

### 3.3.4 自定义SmoothStreaming播放

ExoPlayer为您提供了多种方式来根据您的应用程序定制播放体验。以下各节简要介绍了构建时可用的一些自定义选项`SsMediaSource`。有关更多常规自定义选项，请参见“ [自定义”页面](https://exoplayer.dev/customization.html)。

#### 自定义服务器交互

某些应用可能想要拦截HTTP请求和响应。您可能想要注入自定义请求头，读取服务器的响应头，修改请求的URI等。例如，您的应用程序可以通过在请求媒体段时注入token作为请求头来对自身进行身份验证。

下面的例子演示了如何通过将自定义[HttpDataSources](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/HttpDataSource.html)注入到`SsMediaSource`实现这些行为：

```java
SsMediaSource ssMediaSource =
    new SsMediaSource.Factory(
            () -> {
              HttpDataSource dataSource = new DefaultHttpDataSource(userAgent);
              // Set a custom authentication request header.
              dataSource.setRequestProperty("Header", "Value");
              return dataSource;
            })
        .createMediaSource(ssUri);
```

在上面的代码段中，由`ssMediaSource`触发的HTTP请求，都通过注入的`HttpDataSource`中`"Header: Value"` 添加了请求头。对于与`ssMediaSource`的每次HTTP交互，此行为都是*固定*的。

对于更精细的方法，您可以使用`ResolvingDataSource`来注入即时行为 。以下代码段显示了如何在与HTTP源进行交互之前注入请求头：

```java
SsMediaSource ssMediaSource =
    new SsMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time request headers.
            (DataSpec dataSpec) ->
                dataSpec.withRequestHeaders(getCustomHeaders(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

您还可以使用`ResolvingDataSource` 来对URI进行即时修改，如以下代码片段所示：

```java
SsMediaSource ssMediaSource =
    new SsMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time URI resolution logic.
            (DataSpec dataSpec) -> dataSpec.withUri(resolveUri(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

#### 自定义错误处理

实现自定义[LoadErrorHandlingPolicy](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/LoadErrorHandlingPolicy.html)可使应用程序自定义ExoPlayer对加载错误的处理方式。例如，某个应用可能想要快速失败而不是多次重试，或者可能想要自定义退避逻辑，以控制播放器在每次重试之间等待多长时间。以下代码段显示了创建时如何实现自定义退避逻辑`SsMediaSource`：

```java
SsMediaSource ssMediaSource =
    new SsMediaSource.Factory(dataSourceFactory)
        .setLoadErrorHandlingPolicy(
            new DefaultLoadErrorHandlingPolicy() {
              @Override
              public long getRetryDelayMsFor(...) {
                // Implement custom back-off logic here.
              }
            })
        .createMediaSource(ssUri);
```

您可以在我们的[有关错误处理的文章](https://medium.com/google-exoplayer/load-error-handling-in-exoplayer-488ab6908137)中获得更多信息。

### 3.3.5 BehindLiveWindowException

如果播放了具有有限可用性的实时流，播放器暂停或缓冲了足够长的时间，则播放器可能会落在此实时窗口后面。在这种情况下，将抛出`BehindLiveWindowException`，可以将其捕获以在实时边缘恢复播放器。演示应用程序的[PlayerActivity](https://github.com/google/ExoPlayer/tree/release-v2/demos/main/src/main/java/com/google/android/exoplayer2/demo/PlayerActivity.java)就是这种方法的例证。

```java
@Override
public void onPlayerError(ExoPlaybackException e) {
  if (isBehindLiveWindow(e)) {
    // Re-initialize player at the live edge.
  } else {
    // Handle other errors
  }
}

private static boolean isBehindLiveWindow(ExoPlaybackException e) {
  if (e.type != ExoPlaybackException.TYPE_SOURCE) {
    return false;
  }
  Throwable cause = e.getSourceException();
  while (cause != null) {
    if (cause instanceof BehindLiveWindowException) {
      return true;
    }
    cause = cause.getCause();
  }
  return false;
}
```

## 3.4 [Progressive](https://exoplayer.dev/progressive.html)
ExoPlayer可以直接播放以下容器格式的流。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [样本格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。

| Container format | Supported | Comment                                                      |
| ---------------- | :-------: | :----------------------------------------------------------- |
| MP4              |    YES    |                                                              |
| M4A              |    YES    |                                                              |
| FMP4             |    YES    |                                                              |
| WebM             |    YES    |                                                              |
| Matroska         |    YES    |                                                              |
| MP3              |    YES    | Some streams only seekable using constant bitrate seeking**  |
| Ogg              |    YES    | Containing Vorbis, Opus and FLAC                             |
| WAV              |    YES    |                                                              |
| MPEG-TS          |    YES    |                                                              |
| MPEG-PS          |    YES    |                                                              |
| FLV              |    YES    | Not seekable*                                                |
| ADTS (AAC)       |    YES    | Only seekable using constant bitrate seeking**               |
| FLAC             |    YES    | Using the [FLAC extension](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac) or the FLAC extractor in the [core library](https://github.com/google/ExoPlayer/tree/release-v2/library/core)*** |
| AMR              |    YES    | Only seekable using constant bitrate seeking**               |

*不支持查找，因为容器不提供元数据（例如样本索引）以允许媒体播放器以有效方式执行查找。如果需要查找，我们建议使用更合适的文件格式。

**这些提取器具有`FLAG_ENABLE_CONSTANT_BITRATE_SEEKING`用于使用恒定比特率假设进行近似搜索的标志。默认情况下不启用此功能。最简单的方法使用`DefaultExtractorsFactory.setConstantBitrateSeekingEnabled`来启用此功能支持所有提取，[这里是具体描述](https://exoplayer.dev/progressive.html#enabling-constant-bitrate-seeking)。

*** [FLAC扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac)提取器输出原始音频，框架可以在所有API级别上对其进行处理。所述[核心库](https://github.com/google/ExoPlayer/tree/release-v2/library/core) FLAC提取器输出FLAC音频帧等依赖于具有FLAC解码器（例如，`MediaCodec` 用于处理FLAC（从API级27需要解码器），或者 [FFmpeg的扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ffmpeg)使能与FLAC）。该`DefaultExtractorsFactory`如果应用程序使用内置使用扩展提取[FLAC扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac)。否则，它将使用[核心库](https://github.com/google/ExoPlayer/tree/release-v2/library/core)提取器。

### 3.4.1 创建MediaSource

要播放渐进流，请`ProgressiveMediaSource`像往常一样创建一个并准备播放器。

```java
// Create a data source factory.
DataSource.Factory dataSourceFactory =
    new DefaultHttpDataSourceFactory(Util.getUserAgent(context, "app-name"));
// Create a progressive media source pointing to a stream uri.
MediaSource mediaSource = new ProgressiveMediaSource.Factory(dataSourceFactory)
    .createMediaSource(progressiveUri);
// Create a player instance.
SimpleExoPlayer player = new SimpleExoPlayer.Builder(context).build();
// Prepare the player with the media source.
player.prepare(mediaSource);
```

### 3.4.2 自定义渐进播放

ExoPlayer为您提供了多种方式来根据您的应用程序定制播放体验。以下各节简要介绍了构建`ProgressiveMediaSource`时可用的一些自定义选项。有关更多常规自定义选项，请参见“ [自定义”页面](https://exoplayer.dev/customization.html)。

#### 设置提取器标志

提取器标志可用于控制如何提取单个格式。它们可以设置在上`DefaultExtractorsFactory`，然后可以在实例化a时使用`ProgressiveMediaSource.Factory`。下面的示例传递一个标志，该标志禁用MP4流的编辑列表解析。

```java
DefaultExtractorsFactory extractorsFactory =
    new DefaultExtractorsFactory()
        .setMp4ExtractorFlags(Mp4Extractor.FLAG_WORKAROUND_IGNORE_EDIT_LISTS);
ProgressiveMediaSource progressiveMediaSource =
    new ProgressiveMediaSource.Factory(dataSourceFactory, extractorsFactory)
        .createMediaSource(progressiveUri);
```

#### 启用恒定比特率搜索

对于MP3，ADTS和AMR流，您可以使用带有`FLAG_ENABLE_CONSTANT_BITRATE_SEEKING`标志的恒定比特率假设来启用近似搜索。可以使用上述方法为各个提取程序设置这些标志。要为支持该能力的所有提取程序启用恒定比特率搜索，请使用`DefaultExtractorsFactory.setConstantBitrateSeekingEnabled`。

```java
DefaultExtractorsFactory extractorsFactory =
    new DefaultExtractorsFactory().setConstantBitrateSeekingEnabled(true);
ProgressiveMediaSource progressiveMediaSource =
    new ProgressiveMediaSource.Factory(dataSourceFactory, extractorsFactory)
        .createMediaSource(progressiveUri);
```

#### 自定义服务器交互

某些应用可能想要拦截HTTP请求和响应。您可能想要注入自定义请求标头，读取服务器的响应标头，修改请求的URI等。例如，您的应用程序可以通过在请求媒体段时注入令牌作为标头来对自身进行身份验证。

下面的例子演示了如何通过将自定义[HttpDataSources](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/HttpDataSource.html)注入到`ProgressiveMediaSource`来实现上述行为：

```
ProgressiveMediaSource progressiveMediaSource =
    new ProgressiveMediaSource.Factory(
            () -> {
              HttpDataSource dataSource =
                  new DefaultHttpDataSource(userAgent);
              // Set a custom authentication request header.
              dataSource.setRequestProperty("Header", "Value");
              return dataSource;
            })
        .createMediaSource(progressiveUri);
```

在上面的代码段中，由`progressiveMediaSource`触发的HTTP请求，都通过注入的`HttpDataSource`中`"Header: Value"` 添加了请求头。对于与`progressiveMediaSource`的每次HTTP交互，此行为都是*固定*的。

对于更精细的方法，您可以使用`ResolvingDataSource`来注入即时行为 。以下代码段显示了如何在与HTTP源进行交互之前注入请求标头：

```java
ProgressiveMediaSource progressiveMediaSource =
    new ProgressiveMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time request headers.
            (DataSpec dataSpec) ->
                dataSpec.withRequestHeaders(getCustomHeaders(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

您还可以使用`ResolvingDataSource` 来对URI进行即时修改，如以下代码片段所示：

```java
ProgressiveMediaSource progressiveMediaSource =
    new ProgressiveMediaSource.Factory(
        new ResolvingDataSource.Factory(
            new DefaultHttpDataSourceFactory(userAgent),
            // Provide just-in-time URI resolution logic.
            (DataSpec dataSpec)-> dataSpec.withUri(resolveUri(dataSpec.uri))))
        .createMediaSource(customSchemeUri);
```

#### 自定义错误处理

实现自定义[LoadErrorHandlingPolicy](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/LoadErrorHandlingPolicy.html)可使应用程序自定义ExoPlayer对加载错误的反应方式。例如，某个应用可能想要快速失败而不是多次重试，或者可能想要自定义退避逻辑，以控制播放器在每次重试之间等待多长时间。以下代码段显示了创建时如何实现自定义退避逻辑 `ProgressiveMediaSource`：

```java
ProgressiveMediaSource progressiveMediaSource =
    new ProgressiveMediaSource.Factory(dataSourceFactory)
        .setLoadErrorHandlingPolicy(
            new DefaultLoadErrorHandlingPolicy() {
              @Override
              public long getRetryDelayMsFor(...) {
                // Implement custom back-off logic here.
              }
            })
        .createMediaSource(progressiveUri);
```

您可以在我们[有关错误处理的文章](https://medium.com/google-exoplayer/load-error-handling-in-exoplayer-488ab6908137)中获取更多信息。