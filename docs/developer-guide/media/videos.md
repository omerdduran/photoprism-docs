# Video File Support

## Codecs and Containers

For maximum browser compatibility, PhotoPrism can transcode video codecs and containers [supported by FFmpeg](https://www.ffmpeg.org/documentation.html) to [MPEG-4 AVC](https://en.wikipedia.org/wiki/MPEG-4).

Running the following command in a terminal displays a list of supported codecs:

```
ffmpeg -decoders
```

See our advanced setup guide to learn how to [configure hardware video transcoding](https://docs.photoprism.app/getting-started/advanced/transcoding/).

Please Note:

1. Not all [video and audio formats](https://caniuse.com/?search=video%20format) can be [played with every browser](https://docs.photoprism.app/getting-started/troubleshooting/browsers/). For example, [AAC](https://caniuse.com/aac "Advanced Audio Coding") - the default audio codec for [MPEG-4 AVC / H.264](https://caniuse.com/avc "Advanced Video Coding") - is supported natively in Chrome, Safari, and Edge, while it is only optionally supported by the OS in Firefox and Opera.
2. HEVC/H.265 video files can have a `.mp4` file extension too, which is often associated with AVC only. This is because MP4 is a *container* format, meaning that the actual video content may be compressed with H.264, H.265, or something else. The file extension doesn't really tell you anything other than that it's probably a video file.
3. In case [FFmpeg is disabled](https://docs.photoprism.app/user-guide/settings/advanced/#disable-ffmpeg) or not installed, videos cannot be indexed because still images cannot be created. You should also have [ExifTool enabled](https://docs.photoprism.app/getting-started/config-options/#feature-flags) to extract metadata such as duration, resolution, and codec.

## Hybrid Photo/Video Formats

For more information on hybrid photo/video file formats, e.g. Apple Live Photos and Samsung/Google Motion Photos, see [github.com/photoprism/photoprism/tree/develop/pkg/media](https://github.com/photoprism/photoprism/tree/develop/pkg/media) and [/developer-guide/media/live/](live.md).

## Standard Resolutions

The [`PHOTOPRISM_FFMPEG_SIZE`](../../getting-started/config-options.md#file-conversion) config option allows to limit the resolution of [transcoded videos](../../getting-started/advanced/transcoding.md). It accepts the following standard sizes, while other values are automatically adjusted to the next supported size:

| Size |       Usage       |
|------|-------------------|
|  720 | SD TV, Mobile     |
| 1280 | HD TV, SXGA       |
| 1920 | Full HD           |
| 2048 | DCI 2K, Tablets   |
| 2560 | Quad HD           |
| 3840 | 4K Ultra HD       |
| 4096 | DCI 4K, Retina 4K |
| 7680 | 8K Ultra HD 2     |

## Technical References and Tutorials

| Title                               | URL                                                                         |
|-------------------------------------|-----------------------------------------------------------------------------|
| Web Video Codec Guide               | https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Video_codecs     |
| Web Video Content-Type Headers      | https://developer.mozilla.org/en-US/docs/Web/Media/Formats/codecs_parameter |
| Media Container Formats             | https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Containers       |
| MP4 Signature Format                | https://www.file-recovery.com/mp4-signature-format.htm                      |
| List of file signatures (Wikipedia) | https://en.wikipedia.org/wiki/List_of_file_signatures                       |
| How to use the io.Reader interface  | https://yourbasic.org/golang/io-reader-interface-explained/                 |
| AV1 Codec ISO Media File Format     | https://aomediacodec.github.io/av1-isobmff                                  |

*[HEVC]: High Efficiency Video Coding / H.265
