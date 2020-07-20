[开发者指南原文](https://exoplayer.dev/)

# 1. 概述

## 1.0 首页 [Home](https://exoplayer.dev/)

ExoPlayer是 Android 平台的应用程序级媒体播放器。提供了Android MediaPlayer API的替代方法，可以播放本地和网络上的音频和视频。ExoPlayer 支持 Android MediaPlayer API 当前不支持的功能，包括 _DASH_ 和 _SmoothStreaming_ 自适应播放。与 MediaPlayer API 不同，ExoPlayer 易于自定义和扩展，可以通过 Play 商店应用程序更新进行更新。

本网站(注：指 <https://exoplayer.dev/>)提供了大量信息，可帮助您入门。此外，您还可以：

+ 通过[完成代码实验室](https://codelabs.developers.google.com/codelabs/exoplayer-intro/)或阅读[Hello world](https://exoplayer.dev/hello-world.html)文档，了解如何将ExoPlayer添加到您的应用程序。

+ 在我们的[开发人员博客](https://medium.com/google-exoplayer)上阅读新闻，提示和技巧。
+ 阅读最新的[发行说明](https://github.com/google/ExoPlayer/tree/release-v2/RELEASENOTES.md)。
+ 浏览库[Javadoc](https://exoplayer.dev/doc/reference)。
+ 浏览源代码以获取[最新版本](https://github.com/google/ExoPlayer/tree/release-v2)和[最新分支](https://github.com/google/ExoPlayer/tree/dev-v2)。

## 1.1 优劣 [pros-and-cons](https://exoplayer.dev/pros-and-cons.html)

与 Android 内置的 MediaPlayer 相比，ExoPlayer 具有许多优势：

+ 出现在不同设备和不同 Android 版本上的特定问题更少。
+ 具有随应用程序一起更新播放器的功能。由于 ExoPlayer 是一个包含在应用程序 apk 中的库，因此您可以控制使用的版本，并且可以在常规应用程序更新中轻松地更新到较新的版本。
+ 可以根据开发者的使用场景[自定义和扩展播放器](https://exoplayer.dev/customization.html)。为此，ExoPlayer 专门针对此问题进行了设计，并允许将许多组件替换为自定义实现。
+ 支持[播放列表](https://exoplayer.dev/playlists.html)，以及[剪辑，合并和循环播放媒体](https://exoplayer.dev/media-sources.html#mediasource-composition)。
+ 支持 MediaPlayer 不支持的 [_DASH_](https://exoplayer.dev/dash.html) 和 [_SmoothStreaming_](https://exoplayer.dev/smoothstreaming.html)。同时也支持许多其他格式。有关详细信息，请参见[支持的格式](https://exoplayer.dev/supported-formats.html)。
+ 支持高级 [_HLS_](https://exoplayer.dev/hls.html) 功能，例如正确处理 `#EXT-X-DISCONTINUITY`标签。
+ 在Android 4.4（API级别19）及更高版本上支持[Widevine通用加密](https://exoplayer.dev/drm.html)。
+ 使用官方扩展可以快速与许多其他库集成。例如，使用[IMA扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ima)可以轻松的集成[Interactive Media Ads SDK](https://developers.google.com/interactive-media-ads)从而通过内容获利。

重要的是要注意，还有一些缺点：

+ 对于某些设备上的纯音频播放，ExoPlayer可能比MediaPlayer消耗更多的电池。有关详细信息，请参见[电池消耗页面](https://exoplayer.dev/battery-consumption.html)。
+ 在您的应用程序中包含ExoPlayer会使APK大小增加几百千字节。对于极轻量级的应用程序，这可能仅是一个问题。可以在[APK缩小页面](https://exoplayer.dev/shrinking.html)上找到缩小ExoPlayer的指南。

# 1.2 演示应用 [Demo application](https://exoplayer.dev/demo-application.html)

ExoPlayer 的主要演示应用程序具有两个主要目的：

> 1. 提供一个相对简单但功能齐全的 ExoPlayer 使用示例。您开发自己的应用时，可以将演示应用作为便捷起点。
> 2. 尝试使用 ExoPlayer。除了随附的样本外，该演示应用程序还可用于测试播放您自己的内容。

本节介绍如何获取，编译和运行演示应用，以及如何使用它播放您自己的媒体。

### 1.2.1 获取代码

主演示应用的源代码可以在 [GitHub项目](https://github.com/google/ExoPlayer) 的 `demos/main` 目录找到。如果您尚未克隆项目，请将项目克隆到本地目录中：
(注：github clone 十分缓慢，可以尝试导入到码云再克隆或自行搜索其他办法，本人仓库可用于 clone [ExoPlayer (更新时间2020.7.14)](https://gitee.com/han_jx_dut/ExoPlayer.git))

```shell
git clone https://github.com/google/ExoPlayer.git
```

接下来，在Android Studio中打开项目。您应该在 Android 项目视图中看到以下内容（演示应用程序的相关文件夹已展开）：

![Android Studio中的项目](https://exoplayer.dev/images/demo-app-project.png)

### 1.2.2 编译并运行

在 Android Studio 中选择 `demo` 项目即可编译和运行演示应用程序。该演示应用程序将在连接的 Android 设备上安装并运行。我们建议您尽可能使用物理设备。如果您想使用模拟器，请阅读[受支持设备](https://exoplayer.dev/supported-devices.html)的仿模拟器部分，并确保您的虚拟设备使用的 API 级别至少为 23 的系统映像。

![SampleChooserActivity和PlayerActivity](https://exoplayer.dev/images/demo-app-screenshots.png)



该演示应用程序显示了一个示例列表（`SampleChooserActivity`）。选择一个示例将打开第二个页面（`PlayerActivity`）进行播放。该演示具有播放控件和曲目选择功能。它还使用ExoPlayer的`EventLogger`实用程序类将有用的调试信息输出到系统日志。可以使用以下命令查看此日志记录（以及其他标签的错误级别日志记录）：

```shell
adb logcat EventLogger:V *:E
```

### 1.2.3 启用互动媒体广告

ExoPlayer 具有[IMA扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ima)，可使用 [Interactive Media Ads SDK](https://developers.google.com/interactive-media-ads) 轻松通过您的内容获利。要在演示应用程序中启用扩展，请打开 Android Studio 的 Build Variants 视图，并将演示模块的构建变量设置为 `withExtensionsDebug` 或 `withExtensionsRelease` 如图 3 所示。

![选择withExtensionsDebug构建变体](https://exoplayer.dev/images/demo-app-build-variants.png)

启用IMA扩展程序后，您可以在演示应用程序的示例列表中的“ IMA示例广告代码”下找到获利内容的示例。

### 1.2.4 启用扩展解码器

ExoPlayer 具有许多扩展，允许使用捆绑的软件解码器，包括 AV1, VP9, Opus, FLAC 和 FFmpeg（仅音频）。通过以下步骤您可以构建包括这些扩招的演示应用程序：

1. 构建您要包括的每个扩展。请注意，这需要您手动操作。有关说明，请参阅`README.md`每个扩展名中的文件。
2. 在 Android Studio 的 Build Variants 视图中，将演示模块的构建变量设置为 `withExtensionsDebug` 或 `withExtensionsRelease` 如图 3 所示。
3. 正常编译，安装和运行 `demo` 配置。

![选择demo_extDebug构建变体](https://exoplayer.dev/images/demo-app-build-variants.png)

默认情况下，仅当不存在合适的平台解码器时才使用扩展解码器。可以指定扩展解码器为首选，如以下各节所述。

### 1.2.5播放自己的内容

有多种方法可以在演示应用程序中播放自己的内容。

#### a). 编辑assets/media.exolist.json

演示应用程序中列出的示例是从 `assets/media.exolist.json` 中加载的。通过编辑此 JSON 文件，可以在演示应用程序中添加和删除示例。模式如下，其中 [O] 表示可选属性。

```json
[
  {
    "name": "Name of heading",
    "samples": [
      {
        "name": "Name of sample",
        "uri": "The URI of the sample",
        "extension": "[O] Sample type hint. Values: mpd, ism, m3u8",
        "drm_scheme": "[O] Drm scheme if protected. Values: widevine, playready, clearkey",
        "drm_license_url": "[O] URL of the license server if protected",
        "drm_key_request_properties": "[O] Key request headers if protected",
        "drm_multi_session": "[O] Enables key rotation if protected",
        "ad_tag_uri": "[O] The URI of an ad tag, if using the IMA extension"
        "spherical_stereo_mode": "[O] Enables spherical view. Values: mono, top_bottom, left_right",
        "subtitle_uri": "[O] The URI of a subtitle sidecar file",
        "subtitle_mime_type": "[O] The MIME type of subtitle_uri (required if subtitle_uri is set)",
        "subtitle_language": "[O] The BCP47 language code of the subtitle file (ignored if subtitle_uri is not set)",
      },
      ...etc
    ]
  },
  ...etc
]
```

可以使用以下模式指定示例的播放列表：

```json
[
  {
    "name": "Name of heading",
    "samples": [
      {
        "name": "Name of playlist sample",
        "playlist": [
          {
            "uri": "The URI of the first sample in the playlist",
            "extension": "[O] Sample type hint. Values: mpd, ism, m3u8"
            "drm_scheme": "[O] Drm scheme if protected. Values: widevine, playready, clearkey",
            "drm_license_url": "[O] URL of the license server if protected",
            "drm_key_request_properties": "[O] Key request headers if protected",
            "drm_multi_session": "[O] Enables key rotation if protected"
          },
          {
            "uri": "The URI of the first sample in the playlist",
            "extension": "[O] Sample type hint. Values: mpd, ism, m3u8"
            "drm_scheme": "[O] Drm scheme if protected. Values: widevine, playready, clearkey",
            "drm_license_url": "[O] URL of the license server if protected",
            "drm_key_request_properties": "[O] Key request headers if protected",
            "drm_multi_session": "[O] Enables key rotation if protected"
          },
          ...etc
        ]
      },
      ...etc
    ]
  },
  ...etc
]
```

如果需要，将密钥请求标头指定为包含每个标头的字符串属性的对象：

```json
"drm_key_request_properties": {
  "name1": "value1",
  "name2": "value2",
  ...etc
}
```

在样本选择器活动中，拓展菜单用于指定是否使用扩展解码器和选择应使用哪种 ABR 算法。

#### b). 加载外部 exolist.json 文件

演示应用程序可以使用上面的架构加载外部 JSON 文件，并按照 `*.exolist.json` 约定命名。例如，如果您将这样的文件托管在 `https://yourdomain.com/samples.exolist.json`，则可以使用以下示例在演示应用中将其打开：

```shell
adb shell am start -a com.android.action.VIEW \
    -d https://yourdomain.com/samples.exolist.json
```

单击`*.exolist.json`安装了演示应用程序的设备上的链接（例如，在浏览器或电子邮件客户端中），也会在演示应用程序中将其打开。因此，托管`*.exolist.json`JSON文件提供了一种将内容分发给其他人以在演示应用程序中尝试的简单方法。

#### c). 使用 Intent

Intent 可用于绕过样本列表并直接启动播放。要播放单个样本，请将 Intent 的 action 设置为 `com.google.android.exoplayer.demo.action.VIEW` 并将其数据 URI 设置为要播放的样本。可以使用以下方法从终端启动此 Intent：

```shell
adb shell am start -a com.google.android.exoplayer.demo.action.VIEW \
    -d https://yourdomain.com/sample.mp4
```

单个样本意图支持的可选附加功能是：

- 样本配置额外信息：
  - `extension` [String]样本类型。有效值：`mpd`，`ism`，`m3u8`。
  - `prefer_extension_decoders` [Boolean]相比于平台解码器是否更偏向使用扩展解码器。
  - `drm_scheme`[String]DRM方案（如果受保护）。有效值为`widevine`， `playready`和`clearkey`。也接受DRM方案UUID。
  - `drm_license_url` [String]许可证服务器的网址（如果受保护）。
  - `drm_key_request_properties` [字符串数组]密钥请求标头，打包为name1，value1，name2，value2等（如果受保护）。
  - `drm_multi_session`：[Boolean]如果受保护，则启用键旋转。
  - `ad_tag_uri` [String]广告代码的URI（如果使用IMA扩展名）。
  - `spherical_stereo_mode`[String]启用球形视图。值：`mono`， `top_bottom`和`left_right`。
  - `subtitle_uri` [String]字幕Sidecar文件的URI。
  - `subtitle_mime_type`：[String] subtitle_uri的MIME类型（如果设置了subtitle_uri，则为必需）。
  - `subtitle_language`：[String]字幕文件的BCP47语言代码（如果未设置subtitle_uri则忽略）。
- 播放器配置附加功能：
  - `abr_algorithm`[String] ABR算法，用于自适应播放。有效值为`default`和`random`。
  - `tunneling` [Boolean]是否启用多媒体隧道。

当使用`adb shell am start`用于启动Intetnt时，使用`--es`（例如`--es extension mpd`）设置额外的可选字符串。使用`--ez`（例如`--ez prefer_extension_decoders TRUE`）设置可选的布尔附加值。使用`--esa`（例如 `--esa drm_key_request_properties name1,value1`）设置额外的可选字符串数组。

要播放样本的播放列表，请将Intent的action设置为 `com.google.android.exoplayer.demo.action.VIEW_LIST`。样本配置的额外内容与相同`com.google.android.exoplayer.demo.action.VIEW`，但有两个区别：

- Extras的键应带有下划线，并将示例的从0开始的索引作为后缀。例如，`extension_0`将提示第一个样本的样本类型。`drm_scheme_1`将为第二个样本设置DRM方案。
- 样本的uri使用`uri_<sample-index>`为key值作为额外的参数传递。

不依赖样本的其他附加功能不会更改。例如，您可以在终端中运行以下命令来播放包含两个项目的播放列表，从而覆盖第二个项目的扩展名：

```
adb shell am start -a com.google.android.exoplayer.demo.action.VIEW_LIST \
    --es uri_0 https://a.com/sample1.mp4 \
    --es uri_1 https://b.com/sample2.fake_mpd \
    --es extension_1 mpd
```

## 1.3 支持的格式 [Supported formats](https://exoplayer.dev/supported-formats.html)

在定义ExoPlayer支持的格式时，请务必注意，“媒体格式”是在多个级别上定义的。从最低级别到最高级别分别为：

- 各个媒体样本的采样格式（例如，视频帧或音频帧）。这些是*采样格式*。请注意，典型的视频文件将包含至少两种采样格式的媒体。一种用于视频（例如H.264），另一种用于音频（例如AAC）。
- 容纳媒体样本和关联的元数据的文件的格式。这些是*文件格式*。媒体文件具有单一文件格式（例如，MP4），通常由文件扩展名指示。请注意，对于某些仅音频格式（例如MP3），样本和采样格式可能相同。
- 自适应流技术，例如DASH，SmoothStreaming和HLS。这些不是媒体格式，但是仍然有必要定义ExoPlayer提供的支持级别。

以下各节从最高到最低分别定义了ExoPlayer的支持。最后两节介绍了对独立字幕格式和HDR视频播放的支持。

### 1.3.1自适应流

#### a). DASH

ExoPlayer支持具有多种文件格式的DASH。必须对媒体流进行解复用，这意味着必须在DASH清单中的不同AdaptationSet元素中定义视频，音频和文本（CEA-608是一个例外，如下表所述）。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [样本格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。

| 特征                | 支持的 | 评论                                                         |
| ------------------- | :----: | :----------------------------------------------------------- |
| **货柜**            |        |                                                              |
| FMP4                |   是   | 仅解复用的流                                                 |
| WebM                |   是   | 仅解复用的流                                                 |
| Matroska            |   是   | 仅解复用的流                                                 |
| MPEG-TS             |   否   | 没有支持计划                                                 |
| **隐藏式字幕/字幕** |        |                                                              |
| TTML                |   是   | 根据ISO / IEC 14496-30原始或嵌入到FMP4中                     |
| WebVTT              |   是   | 根据ISO / IEC 14496-30原始或嵌入到FMP4中                     |
| CEA-608             |   是   | 嵌入嵌入在FMP4视频流中的SEI消息                              |
| **元数据**          |        |                                                              |
| EMSG metadata       |   是   | 嵌入FMP4                                                     |
| **内容保护**        |        |                                                              |
| Widevine            |   是   | API 19 +（“ cenc”方案）和25 +（“ cbcs”，“ cbc1”和“ cens”方案） |
| PlayReady SL2000    |   是   | 仅Android TV                                                 |
| ClearKey            |   是   | API 21+                                                      |

#### b). SmoothStreaming

ExoPlayer支持FMP4容器格式的SmoothStreaming。必须对媒体流进行解复用，这意味着必须在SmoothStreaming清单的不同StreamIndex元素中定义视频，音频和文本。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [采样格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。

| 特征                | 支持的 | 评论         |
| ------------------- | :----: | :----------- |
| **货柜**            |        |              |
| FMP4                |   是   | 仅解复用的流 |
| **隐藏式字幕/字幕** |        |              |
| TTML                |   是   | 嵌入FMP4     |
| **内容保护**        |        |              |
| PlayReady SL2000    |   是   | 仅Android TV |

#### c). HLS

ExoPlayer支持具有多种容器格式的HLS。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [采样格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。我们强烈鼓励HLS内容制作产生高品质的HLS流，描述 [在这里](https://medium.com/google-exoplayer/hls-playback-in-exoplayer-a33959a47be7)。

| 特征                | 支持的 | 评论                                         |
| ------------------- | :----: | :------------------------------------------- |
| **货柜**            |        |                                              |
| MPEG-TS             |   是   |                                              |
| FMP4 / CMAF         |   是   |                                              |
| ADTS (AAC)          |   是   |                                              |
| MP3                 |   是   |                                              |
| **隐藏式字幕/字幕** |        |                                              |
| CEA-608             |   是   |                                              |
| WebVTT              |   是   |                                              |
| **元数据**          |        |                                              |
| ID3 metadata        |   是   |                                              |
| **内容保护**        |        |                                              |
| AES-128             |   是   |                                              |
| Sample AES-128      |   否   |                                              |
| Widevine            |   是   | API 19 +（“ cenc”方案）和25 +（“ cbcs”方案） |
| PlayReady SL2000    |   是   | 仅Android TV                                 |

#### d).Progressive container formats

ExoPlayer可以直接播放以下容器格式的流。还必须支持所包含的音频和视频样本格式（有关详细信息，请参阅 [样本格式](https://exoplayer.dev/supported-formats.html#sample-formats)部分）。

| 容器格式   | 支持的 | 评论                                                         |
| ---------- | :----: | :----------------------------------------------------------- |
| MP4        |   是   |                                                              |
| M4A        |   是   |                                                              |
| FMP4       |   是   |                                                              |
| WebM       |   是   |                                                              |
| Matroska   |   是   |                                                              |
| MP3        |   是   | 某些流只能使用恒定比特率搜索来搜索**                         |
| Ogg        |   是   | 包含Vorbis，Opus和FLAC                                       |
| WAV        |   是   |                                                              |
| MPEG-TS    |   是   |                                                              |
| MPEG-PS    |   是   |                                                              |
| FLV        |   是   | 不可搜索*                                                    |
| ADTS (AAC) |   是   | 仅可使用恒定比特率搜索来搜索**                               |
| FLAC       |   是   | 在[核心库中](https://github.com/google/ExoPlayer/tree/release-v2/library/core)使用[FLAC扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac)或FLAC提取器*** |
| AMR        |   是   | 仅可使用恒定比特率搜索来搜索**                               |

*不支持查找，因为容器不提供元数据（例如样本索引）以允许媒体播放器以有效方式执行查找。如果需要查找，我们建议使用更合适的容器格式。

**这些提取器具有`FLAG_ENABLE_CONSTANT_BITRATE_SEEKING`用于使用恒定比特率假设进行近似搜索的标志。默认情况下不启用此功能。最简单的方法来启用此功能支持它的是使用所有提取 `DefaultExtractorsFactory.setConstantBitrateSeekingEnabled`，描述 [在这里](https://exoplayer.dev/progressive.html#enabling-constant-bitrate-seeking)。

*** [FLAC扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac)提取器输出原始音频，框架可以在所有API级别上对其进行处理。所述[核心库](https://github.com/google/ExoPlayer/tree/release-v2/library/core) FLAC提取器输出FLAC音频帧等依赖于具有FLAC解码器（例如，`MediaCodec` 用于处理FLAC（从API级27需要解码器），或者 [FFmpeg的扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ffmpeg)使能与FLAC）。该`DefaultExtractorsFactory`如果应用程序使用内置使用扩展提取[FLAC扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac)。否则，它将使用[核心库](https://github.com/google/ExoPlayer/tree/release-v2/library/core)提取器。

### 1.3.4 采样格式(or 编码格式?）

默认情况下，ExoPlayer使用Android的平台解码器。因此，受支持的样本格式取决于基础平台而不是ExoPlayer。[此处](https://developer.android.com/guide/appendix/media-formats.html#core)记录了Android设备支持的示例格式 。请注意，个别设备可能支持所列格式以外的其他格式。

除了Android的平台解码器之外，ExoPlayer还可以利用软件解码器扩展。这些必须手动构建并包含在希望使用它们的项目中。我们目前为[AV1](https://github.com/google/ExoPlayer/tree/release-v2/extensions/av1)， [VP9](https://github.com/google/ExoPlayer/tree/release-v2/extensions/vp9)， [FLAC](https://github.com/google/ExoPlayer/tree/release-v2/extensions/flac)， [Opus](https://github.com/google/ExoPlayer/tree/release-v2/extensions/opus)和 [FFmpeg](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ffmpeg)提供软件解码器扩展 。

#### FFmpeg扩展

[FFmpeg扩展](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ffmpeg)支撑件进行解码的各种不同的音频样本格式。您可以选择在构建扩展时要包括的解码器，如扩展的[README.md中所述](https://github.com/google/ExoPlayer/tree/release-v2/extensions/ffmpeg/README.md)。下表提供了从音频样本格式到相应的FFmpeg解码器名称的映射。

| Sample format | Decoder name(s) |
| ------------: | --------------- |
|        Vorbis | vorbis          |
|          Opus | opus            |
|          FLAC | flac            |
|          ALAC | alac            |
|     PCM μ-law | pcm_mulaw       |
|     PCM A-law | pcm_alaw        |
| MP1, MP2, MP3 | mp3             |
|        AMR-NB | amrnb           |
|        AMR-WB | amrwb           |
|           AAC | aac             |
|          AC-3 | ac3             |
|        E-AC-3 | eac3            |
|   DTS, DTS-HD | dca             |
|        TrueHD | mlp truehd      |

### 1.3.5 独立字幕格式

ExoPlayer支持多种格式的独立字幕文件。字幕文件可以按照[Media source](https://exoplayer.dev/media-sources.html#side-loading-a-subtitle-file)说明进行侧面加载。

| 容器格式                     | 支持的 | MIME类型                     |
| ---------------------------- | :----: | :--------------------------- |
| WebVTT                       |   是   | MimeTypes.TEXT_VTT           |
| TTML / SMPTE-TT              |   是   | MimeTypes.APPLICATION_TTML   |
| 子纹                         |   是   | MimeTypes.APPLICATION_SUBRIP |
| SubStationAlpha（SSA / ASS） |   是   | MimeTypes.TEXT_SSA           |

### 1.3.6 HDR视频播放

ExoPlayer可处理各种容器中的高动态范围（HDR）视频提取，包括MP4中的Dolby Vision和Matroska / WebM中的HDR10 +。解码和显示HDR内容取决于Android平台和设备的支持。请参阅 [HDR视频播放](https://source.android.com/devices/tech/display/hdr.html) ，以了解有关检查HDR解码/显示功能以及跨Android版本的HDR支持限制的信息。

当播放需要支持特定编解码器配置文件的HDR流时，ExoPlayer的默认`MediaCodec`选择器将选择支持该配置文件的解码器（如果可用），即使在不支持该配置文件的相同MIME类型的另一解码器出现在较高位置时，编解码器列表。如果流超过了相同MIME类型的硬件解码器的功能，则可能导致选择软件解码器。

## 1.4 支持的设备 [**Supported devices**](https://exoplayer.dev/supported-devices.html)

核心ExoPlayer用例所需的最低Android版本是：

| 用途                                                    | Android版本号 | Android API level |
| ------------------------------------------------------- | :-----------: | :---------------: |
| Audio playback                                          |      4.1      |        16         |
| Video playback                                          |      4.1      |        16         |
| DASH (no DRM)                                           |      4.1      |        16         |
| DASH (Widevine CENC; “cenc” scheme)                     |      4.4      |        19         |
| DASH (Widevine CENC; “cbcs”, “cbc1” and “cens” schemes) |      7.1      |        25         |
| DASH (ClearKey)                                         |      5.0      |        21         |
| SmoothStreaming (no DRM)                                |      4.1      |        16         |
| SmoothStreaming (PlayReady SL2000)                      |   AndroidTV   |     AndroidTV     |
| HLS (no DRM)                                            |      4.1      |        16         |
| HLS (AES-128 encryption)                                |      4.1      |        16         |
| HLS (Widevine CENC; “cenc” scheme)                      |      4.4      |        19         |
| HLS (Widevine CENC; “cbcs” scheme)                      |      7.1      |        25         |

对于给定的用例，我们旨在在满足最低版本要求的所有Android设备上支持ExoPlayer。下面列出了特定于设备的已知兼容性问题，GitHub中将[在这里](https://github.com/google/ExoPlayer/labels/bug%3A device specific)跟踪兼容性问题。

- **FireOS（第4版及更低版本）** -尽管我们努力支持FireOS设备，但FireOS是Android的分支，因此我们无法保证会支持该产品。FireOS上遇到的特定于设备的问题通常是由FireOS为运行Android应用程序提供的支持不兼容引起的。此类问题应首先报告给亚马逊。我们知道影响FireOS版本4和更早版本的问题。我们认为FireOS版本5解决了这些问题。
- **Nexus Player（仅在使用HDMI至DVI电缆时）** -仅当设备使用某种类型的HDMI至DVI电缆连接到显示器时，才会影响Nexus Player，这会导致视频播放速度过快。对于终端用户，使用HDMI转DVI电缆不切实际，因为此类电缆无法传输音频。因此，可以安全地忽略此问题。我们建议使用实际的用户场景（例如，使用标准HDMI电缆连接到电视的设备）进行开发和测试。
- **模拟器** -一些Android模拟器无法正确实现Android媒体堆栈的组件，因此不支持ExoPlayer。这是模拟器的问题，而不是ExoPlayer的问题。Android的官方模拟器（Android Studio中的“虚拟设备”）支持ExoPlayer，前提是系统映像的API级别至少为23。API级别较早的系统映像不支持ExoPlayer。第三方仿真器提供的支持水平各不相同。在第三方仿真器上运行ExoPlayer的问题应报告给模拟器的开发人员，而不是报告给ExoPlayer团队。在可能的情况下，我们建议在物理设备而不是模拟器上测试媒体应用程序。

## 1.5 词汇表 [**Glossary**](https://exoplayer.dev/glossary.html)

略