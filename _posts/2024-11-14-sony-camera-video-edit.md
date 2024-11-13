---
title: "Video 사이즈 줄이기"
excerpt: ""
date: 2024-11-14 06:00:00 +0900
categories:
  - Manuals
categories:
  - Tools
---

문제: 4K 이상 화질 동영상 용량이 너무 크다.  

최대한 화질을 유지하면서 용량을 줄이려고 한다.  

## 사용한 방법

해상도를 1920x1080으로 줄이고, h.265 코덱을 사용해보았다.

`ffmpeg -i input.mp4 -c:v libx265 -vf "scale=1920:1080" -preset slow -crf 20 -c:a copy -movflags use_metadata_tags -map_metadata 0 output.mp4`

- 참고 1: [StackOverflow: How can I reduce a video's size with ffmpeg](https://unix.stackexchange.com/questions/28803/how-can-i-reduce-a-videos-size-with-ffmpeg)
- 참고 2: [StackOverflow: Retrieving and saving media metadata using ffmpeg](https://stackoverflow.com/questions/9464617/retrieving-and-saving-media-metadata-using-ffmpeg)
- 참고 3: [FFmpeg Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/H.264)