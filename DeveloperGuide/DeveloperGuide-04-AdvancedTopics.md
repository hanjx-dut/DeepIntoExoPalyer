[开发者指南原文](https://exoplayer.dev/)

# 4. 进阶主题

## 4.1 数字版权管理 [DRM](https://exoplayer.dev/drm.html)

略



## 4.2 问题排查 [Troubleshooting](https://exoplayer.dev/troubleshooting.html)

- [修复“不允许明文HTTP通信”错误](#q1)
- [修复“ SSLHandshakeException”和“ CertPathValidatorException”错误](#q2)
- [为什么有些媒体文件不可搜索？](#q3)
- [为什么在某些MP3文件中查找不准确？](#q4)
- [为什么有些MPEG-TS文件无法播放？](#q5)
- [为什么某些MP4 / FMP4文件播放不正确？](#q6)
- [为什么有些流失败并显示HTTP响应代码301或302？](#q7)
- [为什么有些流因UnrecognizedInputFormatException而失败？](#q8)
- [为什么setPlaybackParameters在某些设备上不能正常工作？](#q9)
- [“在错误的线程上访问播放器”警告是什么意思？](#q10)
- [如何解决“意外状态行：ICY 200 OK”？](#q11)
- [如何查询正在播放的流是否为实时流？](#q12)
- [当我的应用程序后台运行时，如何保持音频播放？](#q13)
- [为什么ExoPlayer支持我的内容但Cast扩展不支持我的内容？](#q14)
- [为什么内容无法播放，但没有出现错误？](#q15)
- [如何获得解码扩展以加载并用于播放？](#q16)
- [我可以直接使用ExoPlayer播放YouTube视频吗？](#q17)

------

#### <a name="q1">修复“不允许明文HTTP通信”错误</a>

如果您的应用在其网络安全配置不允许的情况下请求明文通信（例如`http://` 而不是`https://`），则会发生此错误。如果您的应用定位到Android 9（API级别28）或更高版本，则默认配置会禁用明文通信。

如果您的应用需要使用明文通信（例如，通过进行流媒体传输 `http://`），则需要使用允许它的网络安全配置。 有关详细信息，请参阅Android的 [网络安全文档](https://developer.android.com/training/articles/security-config.html)。要启用所有明文流量，您只需将其添加 `android:usesCleartextTraffic="true"`到`application`您应用的元素中即可 `AndroidManifest.xml`。

#### <a name="q2">修复“ SSLHandshakeException”和“ CertPathValidatorException”错误</a>

`SSLHandshakeException`并`CertPathValidatorException`都表示与服务器的SSL证书有问题。这些错误不是ExoPlayer特有的。请参阅 [Android的SSL文档](https://developer.android.com/training/articles/security-ssl#CommonProblems) 以了解更多详细信息。

#### <a name="q3">为什么有些媒体文件不可定位？</a>

如果在一种媒体中，执行准确定位操作的唯一方法是让播放器扫描并索引整个文件，那么在默认情况下ExoPlayer不支持定位。ExoPlayer认为此类文件不可定位。大多数现代媒体容器格式包括用于定位的元数据（例如，样本索引），具有定义明确的定位算法（例如，用于Ogg的内插二分搜索）或指示其内容是恒定比特率。在这些情况下，有效的定位操作是可能的，并且由ExoPlayer支持。

如果您需要定位但媒体不可定位，建议您将内容转换为使用更合适的容器格式。对于MP3，ADTS和AMR文件，你还可以启用假设文件具有恒定比特率，如[这里](https://exoplayer.dev/progressive.html#enabling-constant-bitrate-seeking)所描述。

#### <a name="q4">为什么在某些MP3文件中定位不准确？</a>

从根本上讲，可变比特率（VBR）MP3文件不适用于需要精确定位的使用场景。有两个原因：

1. 为了进行精确定位，理想情况下，容器格式将在标头中提供精确的时间到字节的映射。该映射允许播放器将请求的寻道时间映射到相应的字节偏移量，并从该偏移量开始请求，解析和播放媒体。不幸的是，可用于在MP3中指定此映射的标头（例如XING标头）通常不准确。
2. 对于没有提供精确的时间到字节映射（或根本没有任何时间到字节映射）的容器格式，如果容器在流中包含绝对样本时间戳，仍然可以执行精确定位。在这种情况下，播放器可以将寻道时间映射到相应字节偏移的最佳猜测，从该偏移开始请求媒体，解析第一个绝对采样时间戳，并有效地对媒体进行引导式二进制定位，直到找到正确的采样为止。不幸的是，MP3在流中不包括绝对采样时间戳，因此这种方法是不可能的。

由于这些原因，对VBR MP3文件执行精确定位的唯一方法是扫描整个文件并在播放器中手动建立时间到字节的映射。这不适用于大型MP3文件，特别是如果用户尝试在开始播放后不久就尝试定位到流的结尾处，这将要求播放器等到下载且索引整个流后再执行定位。对于ExoPlayer，在这种情况下，我们决定针对速度进行优化。

> [问题＃6787](https://github.com/google/ExoPlayer/issues/6787)通过在播放器中建立时间到字节的映射来跟踪添加对精确搜索的支持，但是，添加支持后，默认情况下它可能会被禁用，并且仍然会受到上述基本扩展问题的影响。

如果您控制正在播放的媒体，我们强烈建议您使用更合适的容器格式，例如MP4。没有任何场景表明MP3是媒体格式的最佳选择。

#### <a name="q5">为什么有些MPEG-TS文件无法播放？</a>

某些MPEG-TS文件不包含访问单元定界符（AUD）。默认情况下，ExoPlayer依靠AUD低开销地检测帧边界。类似的，某些MPEG-TS文件不包含IDR关键帧。默认情况下，这些是ExoPlayer唯一参考的关键帧类型。

当要求播放ExoPlayer缺少AUD或IDR关键帧的MPEG-TS文件时，ExoPlayer似乎停留在缓冲状态。如果需要播放此类文件，可以分别使用[`FLAG_DETECT_ACCESS_UNITS`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/ts/DefaultTsPayloadReaderFactory.html#FLAG_DETECT_ACCESS_UNITS)和播放[`FLAG_ALLOW_NON_IDR_KEYFRAMES`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/ts/DefaultTsPayloadReaderFactory.html#FLAG_ALLOW_NON_IDR_KEYFRAMES)。可以在`DefaultExtractorsFactory`中使用[`setTsExtractorFlags`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/DefaultExtractorsFactory#setTsExtractorFlags-int-)设置这些标志 。使用 `FLAG_DETECT_ACCESS_UNITS`相对于基于AUD的帧边界检测除了在计算上开销更高之外，没有其他副作用。使用 `FLAG_ALLOW_NON_IDR_KEYFRAMES`可能会导致在播放开始时以及播放某些MPEG-TS文件时在定位之后立即出现暂时的视觉损坏。

#### <a name="q6">为什么某些MP4 / FMP4文件播放不正确？</a>

一些MP4 / FMP4文件包含编辑列表，这些编辑列表通过跳过，移动或重复采样列表来重写媒体时间轴。ExoPlayer对应用编辑列表有部分支持。例如，它可以延迟或重复从同步样本开始的样本组，但是它不会截断音频样本或预卷媒体，以用于非同步样本开始的编辑。

如果看到介质的一部分意外丢失或重复，请尝试设置[`Mp4Extractor.FLAG_WORKAROUND_IGNORE_EDIT_LISTS`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/mp4/Mp4Extractor.html#FLAG_WORKAROUND_IGNORE_EDIT_LISTS)或 [`FragmentedMp4Extractor.FLAG_WORKAROUND_IGNORE_EDIT_LISTS`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/mp4/FragmentedMp4Extractor.html#FLAG_WORKAROUND_IGNORE_EDIT_LISTS)，这将使提取器完全忽略编辑列表。可以使用[`setMp4ExtractorFlags`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/DefaultExtractorsFactory#setMp4ExtractorFlags-int-)或在DefaultExtractorsFactory中进行设置[`setFragmentedMp4ExtractorFlags`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/extractor/DefaultExtractorsFactory#setFragmentedMp4ExtractorFlags-int-)。

#### <a name="q7">为什么有些流失败并显示HTTP响应代码301或302？</a>

HTTP响应代码301和302均指示重定向。简要说明可以在[Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)上找到。当ExoPlayer发出请求并收到状态码为301或302的响应时，它将正常地跟随重定向并正常开始播放。默认情况下不会发生这种情况的一种情况是跨协议重定向。跨协议重定向是一种从HTTPS重定向到HTTP或反之亦然（或者在另一对协议之间不太常见）的重定向。您可以使用[wget](https://www.gnu.org/software/wget/manual/wget.html)命令行工具测试URL是否导致跨协议重定向，如下所示：

```shell
wget "https://yourserver.com/test.mp3" 2>&1  | grep Location
```

输出应如下所示：

```shell
$ wget "https://yourserver.com/test.mp3" 2>&1  | grep Location
Location: https://second.com/test.mp3 [following]
Location: http://third.com/test.mp3 [following]
```

在此示例中，有两个重定向。第一个重定向是从 `https://yourserver.com/test.mp3`到`https://second.com/test.mp3`。两者都是HTTPS，因此这不是跨协议重定向。第二个重定向是从 `https://second.com/test.mp3`到`http://third.com/test.mp3`。这从HTTPS重定向到HTTP，跨协议重定向也是如此。ExoPlayer在默认配置下不会遵循此重定向，这意味着播放将失败。

如果需要，可以在实例化`HttpDataSource.Factory`应用程序中的ExoPlayer使用的实例时，将ExoPlayer配置为遵循跨协议重定向。要实现此功能，可以使用[`DefaultHttpDataSourceFactory`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/upstream/DefaultHttpDataSourceFactory.html)接受`allowCrossProtocolRedirects`参数的构造函数，其他`HttpDataSource.Factory`的实现也是类似的。将这些参数设置为true以启用跨协议重定向。

#### <a name="q8">为什么有些流因UnrecognizedInputFormatException而失败？</a>

此问题与以下形式的播放失败有关：

```
UnrecognizedInputFormatException: None of the available extractors
(MatroskaExtractor, FragmentedMp4Extractor, ...) could read the stream.
```

导致此失败的可能原因有两种。最常见的原因是您尝试使用`ProgressiveMediaSource`播放DASH（mpd），HLS（m3u8）或SmoothStreaming（ism，isml）内容。要播放这些流，你必须使用正确`MediaSource`的实现，分别为`DashMediaSource`， `HlsMediaSource`和`SsMediaSource`。如果您不知道媒体的类型，那么[`Util.inferContentType`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/util/Util.html#inferContentType-android.net.Uri-)通常可以使用它，如 `PlayerActivity`ExoPlayer演示应用程序所示。

第二个较不常见的原因是ExoPlayer不支持您要播放的媒体的容器格式。在这种情况下，故障将按预期进行，但是可以随时向我们的[问题跟踪器](https://github.com/google/ExoPlayer/issues)提交功能请求 ，包括容器格式和测试流的详细信息。在提交新的功能请求之前，请先搜索现有功能请求。

#### <a name="q9">为什么setPlaybackParameters在某些设备上不能正常工作？</a>

在Android M及更早版本上运行应用程序的调试版本时，使用[`setPlaybackParameters`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/Player.html#setPlaybackParameters-com.google.android.exoplayer2.PlaybackParameters-)API 时，您可能会遇到性能不稳定，audible artifacts，以及CPU使用率高的情况。这是因为对于在这些版本的Android上运行的调试版本，禁用了对此API重要的优化。

请务必注意，此问题仅影响调试版本。它并*不会* 影响release版本，release版本中优化始终启用。因此，您提供给最终用户的发行版不应受到此问题的影响。

#### <a name="q10">“在错误的线程上访问播放器”警告是什么意思？</a>

如果看到此警告，则表明您的应用程序中的某些代码正在`SimpleExoPlayer`错误的线程上访问 （请检查所报告的堆栈跟踪！）。ExoPlayer实例仅需要从单个线程访问。在大多数情况下，这应该是应用程序的主线程。有关详细信息，请阅读[ExoPlayer Javadoc](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ExoPlayer.html)的[“线程模型”部分](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ExoPlayer.html)。

#### <a name="q11">如何解决“意外状态行：ICY 200 OK”？</a>

如果服务器响应包括ICY状态行而不是HTTP兼容行，则可能发生此问题。ICY状态行已弃用，不应使用，因此，如果您控制服务器，则应更新它以提供HTTP兼容响应。如果您无法执行此操作，则使用 [OkHttp扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/okhttp)名将解决此问题，因为它能够正确处理ICY状态行。

#### <a name="q12">如何查询正在播放的流是否为实时流？</a>

您可以查询ExoPlayer的[`isCurrentWindowLive`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ExoPlayer.html#isCurrentWindowLive--)方法。另外，您可以检查[`isCurrentWindowDynamic`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/ExoPlayer.html#isCurrentWindowDynamic--)窗口是否是动态的（即，仍会随着时间而更新）。

#### <a name="q13">当我的应用程序后台运行时，如何保持音频播放？</a>

您需要采取一些步骤来确保在后台运行应用程序时继续播放音频：

1. 您需要运行[前台服务](https://developer.android.com/guide/components/services.html#Foreground)。这样可以防止系统杀死您的进程以释放资源。
2. 您需要持[`WifiLock`](https://developer.android.com/reference/android/net/wifi/WifiManager.WifiLock.html)和[`WakeLock`](https://developer.android.com/reference/android/os/PowerManager.WakeLock.html)。这些可确保系统使WiFi和CPU保持唤醒状态。如果使用了[`SimpleExoPlayer`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/SimpleExoPlayer.html)，通过调用[`setWakeMode`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/SimpleExoPlayer.html#setWakeMode-int-)可以轻松完成此操作 ，该操作将在正确的时间自动获取并释放所需的锁。

重要的是，如果没有使用`setWakeMode`，一旦不再播放音频，请释放锁并停止服务。

#### <a name="q14">为什么ExoPlayer支持我的内容但Cast扩展不支持我的内容？</a>

您尝试播放的内容可能未 [启用CORS](https://www.w3.org/wiki/CORS_Enabled)。在[铸造框架](https://developers.google.com/cast/docs/chrome_sender/advanced#cors_requirements)要求的内容是为了发挥它启用CORS。

#### <a name="q15">为什么内容无法播放，但没有出现错误？</a>

您正在其上播放内容的设备可能不支持特定的媒体样本格式。可以通过向[`EventLogger`](https://exoplayer.dev/listening-to-player-events.html#using-eventlogger)您的播放器添加一个侦听器，然后在Logcat中查找与此行类似的行来轻松确认这一点：

```
[ ] Track:x, id=x, mimeType=mime/type, ... , supported=NO_UNSUPPORTED_TYPE
```

`NO_UNSUPPORTED_TYPE`表示设备无法解码所指定的媒体样本格式`mimeType`。有关受支持的示例格式的信息，请参见[Android媒体格式文档](https://developer.android.com/guide/topics/media/media-formats#core)。[如何获得解码扩展以加载并用于播放？](https://exoplayer.dev/troubleshooting.html#how-can-i-get-a-decoding-extension-to-load-and-be-used-for-playback)可能也有用。

#### <a name="q16">如何获得解码扩展以加载并用于播放？</a>

- 大多数扩展程序都有手动步骤来切换并建立依赖关系，因此请确保已按照自述文件中的步骤进行了相关扩展。例如，对于FFmpeg扩展，必须遵循[extensions / ffmpeg / README.md](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ffmpeg/README.md)中的说明，包括传递配置标志以[使解码器启用](https://exoplayer.dev/supported-formats.html#ffmpeg-extension)您要播放的格式。
- 对于具有native代码的扩展，请确保您使用的是自述文件中指定的正确版本的Android NDK，并请注意在配置和构建过程中是否出现任何错误。按照自述文件中的步骤操作后，应该会看到`.so` 文件显示在`libs`每种受支持体系结构的扩展路径的子目录中。
- 要在[演示应用程序中](https://exoplayer.dev/demo-application.html)使用扩展名尝试回放，请参阅 [启用扩展名解码器](https://exoplayer.dev/demo-application.html#enabling-extension-decoders)。有关从您自己的应用程序使用扩展的说明，请参阅扩展的自述文件。
- 如果使用[`DefaultRenderersFactory`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/DefaultRenderersFactory.html)，扩展加载时，您应该在Logcat中看到一条info级别的日志，例如“ Loaded FfmpegAudioRenderer”。如果丢失，请确保应用程序对扩展名具有依赖性。
- 如果您[`LibraryLoader`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/util/LibraryLoader.html)在Logcat中看到warning级别的日志，则表明加载扩展的本机组件失败。如果发生这种情况，请检查您是否正确执行了扩展程序自述文件中的步骤，并且在按照说明进行操作时没有错误输出。

如果您仍然在使用扩展程序时遇到问题，请检查ExoPlayer [问题跟踪器](https://github.com/google/ExoPlayer/issues)，以获取相关的最新问题。如果您需要提出新的问题，并且它与构建扩展的本机部分有关，请包括运行README指令的完整命令行输出，以帮助我们诊断问题。

#### <a name="q17">我可以直接使用ExoPlayer播放YouTube视频吗？</a>

不可以，ExoPlayer无法播放来自YouTube的视频，即“https://www.youtube.com/watch?v=…”形式的网址。相反，您应该使用[YouTube Android Player API](https://developers.google.com/youtube/android/player/)，这是在Android上播放YouTube视频的官方方法。



## 4.3 定制化 [Customization](https://exoplayer.dev/customization.html)

`ExoPlayer`接口是ExoPlayer库的核心。`ExoPlayer`提供传统高层次的媒体播放器功能，比如缓冲介质，播放，暂停和寻求的能力。需要实现的是对正在播放的媒体类型，媒体的存储方式和存储方式以及呈现方式做出很少的假设（并因此而施加了很少的限制）。`ExoPlayer`的实现方式不是直接实现媒体的加载和渲染，而是将这项工作委托给在创建播放器或准备播放时注入的组件。所有`ExoPlayer`实现共有的组件 是：

- `MediaSource`定义了要播放的媒体，加载媒体，和已加载的媒体中哪些可以访问。在播放开始时`MediaSource`通过`ExoPlayer.prepare`注入。
- `Renderer`呈现媒体的各个组成部分。`Renderer`在创建播放器时注入。
- 每个可用的`Renderer`消费`MediaSource`时通过所提供的`TrackSelector`选择轨道。`TrackSelector`在创建播放器时注入。
- `LoadControl`用于控制何时`MediaSource`缓冲更多媒体，以及缓冲多少媒体。`LoadControl`在创建播放器时注入。

该库为常见用例提供了这些组件的默认实现。ExoPlayer`可以使用这些组件，但也可以使用自定义的实现是否需要非标准行为建。自定义实现的一些用例是：

- `Renderer`–您可能想要实现自定义`Renderer`以处理库提供的默认实现不支持的媒体类型。
- `TrackSelector`–实现自定义`TrackSelector`允许应用程序开发人员更改`MediaSource`为每个可用`Renderer`选择由公开的轨道以供消费的方式。
- `LoadControl`–实现自定义`LoadControl`允许应用程序开发人员更改播放器的缓冲策略。
- `Extractor`–如果需要支持库当前不支持的容器格式，请考虑实现一个自定义`Extractor`类，然后将该类用于`ProgressiveMediaSource`播放该类型的媒体。
- `MediaSource`– 如果您希望获取媒体样本以自定义方式馈送到渲染器，或者希望实现自定义`MediaSource`合成行为，则实现自定义`MediaSource`可能是合适的。
- `DataSource`– ExoPlayer的上游软件包已经包含许多`DataSource`用于不同用例的 实现。您可能想要实现自己的`DataSource`类，以其他方式加载数据，例如通过自定义协议，使用自定义HTTP堆栈或从自定义持久性缓存中加载数据。

在整个库中都存在注入实现播放器功能的组件的概念。组件的默认实现将工作委托给其他注入的组件。这允许将许多子组件分别替换为自定义实现。例如，默认`MediaSource`实现要求一个或多个 `DataSource`工厂通过自己的工厂注入。通过提供定制`DataSource`工厂，可以从非标准来源或通过其他网络堆栈加载数据。

在构建自定义组件时，我们建议以下内容：

- 如果自定义组件需要将事件报告回应用程序，我们建议您使用与现有ExoPlayer组件相同的模型，将事件监听器与`Handler`一起传递给组件的构造函数。
- 我们建议自定义组件使用与现有ExoPlayer组件相同的模型，以允许应用在回放过程中进行重新配置，如下所述。为此，自定义组件应实现 `PlayerMessage.Target`并接收`handleMessage`方法中的配置更改 。应用程序代码应通过调用ExoPlayer的`createMessage`方法，配置消息并将其发送到组件来传递配置更改`PlayerMessage.send`。

### 向组件发送消息

可以将消息发送到ExoPlayer组件。可以使用`ExoPlayer.createMessage`创建，然后使用`PlayerMessage.send`发送。默认情况下，消息会尽快在播放线程上传递，但是可以通过设置另一个回调线程（使用 `PlayerMessage.setHandler`）或通过指定传递播放位置（使用`PlayerMessage.setPosition`）来自定义消息。发送要在播放线程上传递的消息可确保它们与在播放器上执行的任何其他操作顺序执行。

大多数ExoPlayer的现成渲染器都支持消息，这些消息允许在播放过程中更改其配置。例如，音频渲染器接受消息以设置音量，而视频渲染器接受消息以设置surface。这些消息应在播放线程上传递，以确保线程安全。



## 4.4 电池消耗[Battery consumption](https://exoplayer.dev/battery-consumption.html)

### 由媒体播放引起的电池消耗有多重要？

避免不必要的电池消耗是开发良好的Android应用程序的重要方面。媒体播放可能是造成电池耗电的主要原因，但是其对特定应用程序的重要性在很大程度上取决于其使用模式。如果每天仅使用一个应用播放少量媒体，则相应的电池消耗将仅占设备总消耗的一小部分。这样，在选择使用哪个播放器时，优先考虑功能集和可靠性优先于优化电池。另一方面，如果每天经常使用某个应用来播放大量媒体，则在多个可行选项之间进行选择时，应更加权衡电池消耗的优化。

### ExoPlayer的能效如何？

Android设备和媒体内容生态系统的多样性意味着难以就ExoPlayer的电池消耗，尤其是与Android MediaPlayer API的比较作出广泛适用的声明。绝对性能和相对性能都会因硬件，Android版本和播放的媒体而异。因此，以下提供的信息应仅作为指导。

#### 视频播放

对于视频播放，我们的测量表明，ExoPlayer和MediaPlayer消耗的电量相似。在两种情况下，显示和解码视频流所需的功率都相同，并且这些占了回放过程中消耗的大部分功率。

无论使用哪种媒体播放器，在`SurfaceView`和`TextureView`之间进行选择都会对功耗产生重大影响。 `SurfaceView`具有更高的电源效率，在某些设备上，`TextureView`视频播放期间的总功耗增加了多达30％。`SurfaceView` 因此在可能的情况下应首选。阅读更多有关在`SurfaceView`和`TextureView`之间进行选择的  [信息](https://exoplayer.dev/ui-components.html#choosing-a-surface-type)。

以下是[Monsoon power monitor](https://www.msoon.com/battery-configuration)在Pixel 2上测量的播放1080p和480p视频的一些功耗测量。如上所述，这些数字不应该用于得出有关Android设备和媒体内容生态系统功耗的一般结论。

|                   | MediaPlayer | ExoPlayer |
| ----------------- | :---------: | :-------- |
| SurfaceView 1080p |   202 mAh   | 214 mAh   |
| TextureView 1080p |   219 mAh   | 221 mAh   |
| SurfaceView 480p  |   194 mAh   | 207 mAh   |
| TextureView 480p  |   212 mAh   | 215 mAh   |

#### 音频播放

对于音频播放，我们的测量表明，ExoPlayer可以比MediaPlayer消耗更多电量。对于支持音频卸载的设备更甚，它允许将音频处理从CPU卸载到专用信号处理器。MediaPlayer能够利用音频卸载来降低功耗，而ExoPlayer则不能，因为Android框架中没有用于启用它的公共API。请注意，这还意味着任何其他应用程序级别的媒体播放器或`AudioTrack`直接使用的应用程序都不能使用音频卸载。

ExoPlayer在MediaPlayer上提供的增强的鲁棒性，灵活性和功能集是否值得在仅音频用例中增加功耗是值得应用程序开发人员考虑到他们的要求和应用程序使用方式的事情。

> Android Q中新的公共API将使ExoPlayer以及其他应用程序级别的媒体播放器能够利用音频卸载功能。我们计划在将来的ExoPlayer版本中使用这些API。





## 4.5 缩减APK大小 [APK shrinking](https://exoplayer.dev/shrinking.html)

缩小APK大小是开发良好的Android应用程序的重要方面。在针对发展中市场以及开发Android Instant App时尤其如此。对于此类情况，可能需要最小化APK中包含的ExoPlayer库的大小。该页面概述了一些有助于实现此目标的简单步骤。

### 使用模块化依赖

使用ExoPlayer的最方便方法是向完整库添加依赖项：

```
implementation 'com.google.android.exoplayer:exoplayer:2.X.X'
```

但是，这可能带来超出您的应用程序需求的更多功能。相反，仅取决于您实际需要的库模块。例如，以下内容将添加对Core，DASH和UI库模块的依赖关系，这可能是播放DASH内容的应用程序所必需的：

```
implementation 'com.google.android.exoplayer:exoplayer-core:2.X.X'
implementation 'com.google.android.exoplayer:exoplayer-dash:2.X.X'
implementation 'com.google.android.exoplayer:exoplayer-ui:2.X.X'
```

### 使用ProGuard并缩减资源

可以通过在应用模块的`build.gradle`文件中启用ProGuard来删除应用未使用的类：

```groovy
buildTypes {
   release {
       minifyEnabled true
       shrinkResources true
       useProguard true
       proguardFiles = [
           getDefaultProguardFile('proguard-android.txt'),
           'proguard-rules.pro'
       ]
   }
}
```

ExoPlayer的结构允许ProGuard删除未使用的功能。例如，对于播放DASH内容的应用，ExoPlayer对APK大小的贡献可以减少大约40％。

在您的应用程序模块的`build.gradle`中启用`shrinkResources`可以进一步减小大小。

### 指定您的应用需要哪些extractors

如果您的应用使用`ProgressiveMediaSource`，请注意默认情况下它将使用 `DefaultExtractorsFactory`。`DefaultExtractorsFactory`取决于`Extractor`ExoPlayer库中提供的所有实现，因此ProGuard将不会删除所有实现。如果您知道您的应用仅需要播放少量容器格式，则可以指定自己的容器格式 `ExtractorsFactory`。例如，仅需要播放mp4文件的应用可以定义一个工厂，例如：

```java
private class Mp4ExtractorsFactory implements ExtractorsFactory {
  @Override
  public Extractor[] createExtractors() {
      return new Extractor[] {new Mp4Extractor()};
  }
}
```

并在实例化`ProgressiveMediaSource`实例时使用它，例如：

```java
new ProgressiveMediaSource.Factory(
        mediaDataSourceFactory, new Mp4ExtractorsFactory())
    .createMediaSource(uri);
```

这将允许`Extractor`ProGuard删除其他实施方式，从而可以大大减小尺寸。

### 指定您的应用需要哪些renderer

如果您的应用使用`SimpleExoPlayer`，请注意默认情况下，播放器的渲染器将使用创建`DefaultRenderersFactory`。 `DefaultRenderersFactory`取决于`Renderer`ExoPlayer库中提供的所有实现，因此ProGuard将不会删除所有实现。如果您知道您的应用仅需要一部分渲染器，则可以指定自己的渲染器`RenderersFactory`。例如，仅播放音频的应用可以定义如下工厂：

```java
private class AudioOnlyRenderersFactory implements RenderersFactory {

  private final Context context;

  public AudioOnlyRenderersFactory(Context context) {
    this.context = context;
  }

  @Override
  public Renderer[] createRenderers(
      Handler eventHandler,
      VideoRendererEventListener videoRendererEventListener,
      AudioRendererEventListener audioRendererEventListener,
      TextOutput textRendererOutput,
      MetadataOutput metadataRendererOutput,
      DrmSessionManager<FrameworkMediaCrypto> drmSessionManager) {
    return new Renderer[] {new MediaCodecAudioRenderer(
        MediaCodecSelector.DEFAULT, eventHandler, audioRendererEventListener)};
  }

}
```

并在实例化`SimpleExoPlayer`实例时使用它，例如：

```java
SimpleExoPlayer player = new SimpleExoPlayer.Builder(
    context, new AudioOnlyRenderersFactory(context)).build();
```

这将允许`Renderer`ProGuard删除其他实现。在此特定示例中，视频，文本和元数据渲染器已删除。



## 4.6 OEM测试 [OEM testing](https://exoplayer.dev/oems.html)

略



## 4.7 设计文档 [Design documents](https://exoplayer.dev/design-documents.html)

为了获得开发人员的早期反馈，我们在此页面上发布了有关较大更改的设计文档。随时对仍处于RFC状态的文档发表评论。请注意，一旦实施更改，我们通常不会更新文档。

### Design documents in RFC status

- [Add support for partially fragmented MP4s](https://docs.google.com/document/d/1NUheADYlqIVVPT8Ch5UbV8DJHLDoxMoOD_L8mvU8tTM) (July 2020)
- [Audio offload](https://docs.google.com/document/d/1r6wi6OtJUaI1QU8QLrLJTZieQBFTN1fyBK4U_PoPp3g) (April 2020)
- [Sniffing order optimization](https://docs.google.com/document/d/1w2mKaWMxfz2Ei8-LdxqbPs1VLe_oudB-eryXXw9OvQQ) (April 2020)
- [Masking seek state changes](https://docs.google.com/document/d/1XeOduvYus9HfwXtOtoRC185T4PK-L4u7JRmNM46Ee4w) (March 2020)
- [Frame-accurate pausing](https://docs.google.com/document/d/1xXGvIMAYDWN4BGUNqAplNN-T7rjrW_1EAVXpyCcAqUI) (January 2020)
- [Index seeking in MP3 streams](https://docs.google.com/document/d/1ZtQsCFvi_LiwFqhHWy20dJ1XwHLOXE4BJ5SzXWJ9a9E) (January 2020)
- [Low-latency live playback](https://docs.google.com/document/d/1z9qwuP7ff9sf3DZboXnhEF9hzW3Ng5rfJVqlGn8N38k) (September 2019)
- [Unwrapping Nested Metadata](https://docs.google.com/document/d/1TS13CVmexaLG1C4TdD-4NkX-BCSr_76FaHVOPo6XP1E) (September 2019)
- [Playlist API](https://docs.google.com/document/d/11h0S91KI5TB3NNZUtsCzg0S7r6nyTnF_tDZZAtmY93g) (July 2019)
- [Bandwidth estimation analysis](https://docs.google.com/document/d/1e3jVkZ6nxNWgCqTNibqV8uJcKo8d597XVl3nJkY7P8c) (July 2019)