## RTSP-Client

This repository contains a basic RTSP Server using FFMpeg wrapper for iOS based on the [Hardware Video Encoding on iPhone — RTSP Server example][1].

### Disclaimer
This repository contains a sample code intended to demonstrate the capabilities of the ffmpeg as a camera recorder. It is not intended to be used as-is in applications as a library dependency, and will not be maintained as such. Bug fix contributions are welcome, but issues and feature requests will not be addressed.

### Example Contents
This sample code takes the following approach to this problem:

- Only video is written using the AVAssetWriter instance, or it would be impossible to distinguish video from audio in the mdat atom.
- Initially, I create two AVAssetWriter instances. The first frame is written to both, and then one instance is closed. Once the moov atom has been written to that file, I parse the file and assume that the parameters apply to both instances, since the initial conditions were the same.
- Once I have the parameters, I use a dispatch_source object to trigger reads from the file whenever new data is written. The body of the mdat chunk consists of H264 NALUs, each preceded by a length field. Although the length of the mdat chunk is not known, we can safely assume that it will continue to the end of the file (until we finish the output file and the moov is added).
- For RTP delivery of the data, we group the NALUs into frames by parsing the NALU headers. Since there are no AUDs marking the frame boundaries, this requires looking at several different elements of the NALU header.
- Timestamps arrive with the uncompressed frames from the camera and are stored in a FIFO. These timestamps are applied to the compressed frames in the same order. Fortunately, the AVAssetWriter live encoder does not require re-ordering of frames. Update this is no longer true, and I now have a version that supports re-ordered frames.
- When the file gets too large, a new instance of AVAssetWriter is used, so that the old temporary file can be deleted. Transition code must then wait for the old instance to be closed so that the remaining NALUs can be read from the mdat atom without reading past the end of that atom into the subsequent metadata. Finally, the new file is opened and timestamps are adjusted. The resulting compressed output is seamless.

A little experimentation suggests that we are able to read compressed frames from file about 500ms or so after they are captured, and these frames then arrive around 200ms after that at the client app.

## Credits
* [Hardware Video Encoding on iPhone][1]
* [FFmpeg][2]

### Pre-requisites
    
- FFmpeg 3.3
- Xcode 8.3.2
- gas-preprocessor
- yasm 1.2.0

## License

The code supplied here is covered under the MIT Open Source License.

[1]: http://www.gdcl.co.uk/2013/02/20/iOS-Video-Encoding.html
[2]: https://www.ffmpeg.org/
