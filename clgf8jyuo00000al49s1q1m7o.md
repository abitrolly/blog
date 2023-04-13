---
title: "AIFF 48kHz/24-bit"
datePublished: Thu Apr 13 2023 14:48:44 GMT+0000 (Coordinated Universal Time)
cuid: clgf8jyuo00000al49s1q1m7o
slug: aiff-48-24
tags: audio, filmmaking, aiff, post-production

---

AIFF is a file format, which is also a standard way to exchange audio when making movies. When video footage and sounds are recorded, the film [post-production](https://en.wikipedia.org/wiki/Post-production) starts, and AIFF provides a way to send uncompressed audio to editors.

Recording sound outside the studio usually involves several mics and a "digital audio recorder" device like [Zoom](https://en.wikipedia.org/wiki/Zoom_H4n_Handy_Recorder). My rather Zoom H1 Handy Recorder writes WAV and MP3, but no AIFF. During filming session I recorded WAV as 96kHz/24-bit, and now I need convert that to AIFF. Some records are done with stereo mic, and some with mono mic, so I also need to convert records with only left channel to mono.

The `ffmpeg` command I used to downsample all 96kHz WAV in a directory to AIFF.

```bash
for i in .WAV; do ffmpeg -i "$i" -c:a pcm_s24be -ar 48000 "${i%.}.aiff"; done
```

This still produces stereo files. I added `-af "pan=mono|FC=FL"`param to do [mono output](https://superuser.com/questions/601972/ffmpeg-isolate-one-audio-channel) and map Front Left channel to Front Center.

```bash
for i in .WAV; do ffmpeg -i "$i" -c:a pcm_s24be -ar 48000 -af "pan=mono|FC=FL" "${i%.}.aiff"; done
```

I also want to copy original metadata to AIFF.

```bash
$ ffprobe S1K8D1.WAV -hide_banner 
Input #0, wav, from 'S1K8D1.WAV':
  Metadata:
    encoded_by      : ZOOM Handy Recorder H1
    date            : 2022-04-05
    creation_time   : 15:43:50
    time_reference  : 5436480000
    coding_history  : A=PCM,F=96000,W=24,M=stereo,T=ZOOM Handy Recorder H1
  Duration: 00:00:27.53, bitrate: 4617 kb/s
  Stream #0:0: Audio: pcm_s24le ([1][0][0][0] / 0x0001), 96000 Hz, 2 channels, s32 (24 bit), 4608 kb/s
```

[For that](https://superuser.com/questions/1493313/covert-flac-to-aiff-while-saving-tags-metadata) I had to specify `-write_id3v2 1` param.

```bash
for i in .WAV; do ffmpeg -i "$i" -c:a pcm_s24be -ar 48000 -af "pan=mono|FC=FL" -write_id3v2 yes "${i%.}.aiff"; done
```

Some drawbacks.

1. I can not validate that there is no right channel before processing.
    
2. Metadata was not updated after recoding. Ideally reencoder should add another line to `coding_history` ([`BEXT CodingHistory`](https://www.si.edu/sites/default/files/unit/OCIO/si_dams_metadata_guidelines_mm16_2020.pdf)).