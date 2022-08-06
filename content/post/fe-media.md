---
title: "Intro to Video Format"
date: "2021-12-13"
description: ""
# tags: []
categories: [
    "frontend",
]
series: ["frontend"]
katex: true
---



## Codec



![](/blog/post/images/codec.png "Figure 1: Process of video creation.")



Codec, short for compressor-decompressor, is an algorithm designed for compressing and decompressing video and audio data. Simply put, it's an encoding and decoding algorithm. 

Why encoding? 

- raw data can be very large and noisy
- encoding makes it easy to transmit and storage data

Why decoding?

- preview or playback

### Video Codec

- H.264/AVC (MPEG-4 part 10/Advanced Video Coding) is the most widely used video codec on the market
- VP8, free and open source codec owned by Google
- Ogg Theora, typically used with OGG container
- H.265, successor of H.264



### Lossy Audio 

- AAC is an ISO standard audio codec widely used in iTunes, iPhone, etc. Much better than MP3
- Ogg Vorbis, free and open source, better than MP3, typically used with OGG container
- MP3, the earliest and most popular audio format



### Lossless Audio

- FLAC, typically compress a file by 50%
- WAV/WMA, owned by Microsoft. Started in early 1990s



### Container

Now we have the encoded video and audio, are we ready for playing? No. We still need a container to include the encoded data and other metadata, such as video thumbnail and subtitles. Table below shows some common cotainer formats for storage and online video streaming and distribution.



| Container | Desc                                                         | Video Codec           | Audio Codec |
| --------- | ------------------------------------------------------------ | --------------------- | ----------- |
| MP4       | - MPEG 4 Part-14, ISO/IEC standard container format<br />- supports multiple codecs<br />- `.m4v` is for video while `.m4a` is for audio only | H.264, MPEG-2, MPEG-4 | AAC, MPEG-1 |
| OGG       |                                                              | Theora                | Vorbis      |
| WebM      | developed by Google based on the open-source container `Matroska (.mkv)` | VP8                   | Vorbis      |
| QuickTime | `.mov` or `.qt`, widely used in Apple's products             |                       |             |
| AVI       | - developed by Microsoft in early 1990s<br />- does not support streaming media |                       |             |



We can see that 

- some containers (MP4) support variety of codecs while some containers (WebM) support only one type of codec
- AVI does not support straming media, that is, playing and downloading at the same time



## Caniuse

Browers that support HTML5 audio and video 

- IE>=9
- FF >= 3.5
- safari >= 4
- opera >= 10.5，
- chrome
- safari for iOS
- webkit for android



Besides, different browsers support different container formats and codecs. In other words, there is no standard specification about video format and codecs for web development. Table below shows the container format between browsers.

| Browsers | Supported codecs and containers      |
| -------- | ------------------------------------ |
| Chrome   | theora/Vorbis/Ogg;   VP8/Vorbis/WebM |
| FF       | theora/Vorbis/Ogg;   VP8/Vorbis/WebM |
| IE       | H.264/AAC/MPEG 4;  VP8/Vorbis/WebM   |
| Opera    | theora/Vorbis/Ogg;   VP8/Vorbis/WebM |
| Safari   | H.264/AAC/MPEG 4                     |

The audio codecs supported between modern browsers.

|            | IE   | FF   | Chrome | Opera | Safari |
| ---------- | ---- | ---- | ------ | ----- | ------ |
| MP3        | 9    | NO   | 5      | NO    | 3.1    |
| Ogg Vorbis | NO   | 3.6  | 5      | 10.5  | NO     |
| WAV        | NO   | 3.6  | 8      | 10.5  | 3.1    |

From the above tables, we see that we need to prepare at least two formats to accommodate the most browsers.

Video 

- H.264/AAC/MPEG 4
- theora/Vorbis/Ogg
- VP8/Vorbis/WebM

Audio

- MP3
- OGG



## Feature Detection



`canPlayType(mediaType)` tells us how likely it is that the media can be played. We can dynamically detect whether the browser supports the audio element.



```js
var hasMedia = !!(document.createElement("audio").canPlayType;
```



If not, a simple way to degrade is to insert fallback content into the audio/video tags, as the code snippet below shows.



```html
<audio id="myAudio" controls>
    <source src="song.ogg" type="audio/ogg">
    <source src="song.mp3" type="audio/mpeg">
    Audio player not available.
</audio>
```



However, even though browsers support some kind of format, it does not mean that it can play it. So, to ensure the successfull palyback, we specify the codecs using the `type` attribute.



| Container         | type value                                      |
| ----------------- | ----------------------------------------------- |
| theora/vorbis/Ogg | type="video/ogg;codecs='theora,vorbis'"         |
| vorbis/Ogg        | type="audio/ogg;codecs='vorbis'"                |
| vp8/vorbis/webm   | type="video/webm;codecs='vp8,vorbis'"           |
| H.264/vorbis/mp4  | type="video/mp4;codecs='avc1.42E01E,mp4a.40.2'" |



## References

- [Intro to Video Codecs](http://xahlee.info/comp/streaming_video_notes.html)
- [视音频编解码技术零基础学习方法](https://blog.csdn.net/leixiaohua1020/article/details/18893769)
- [FFmpeg](http://www.ffmpeg.org/)

