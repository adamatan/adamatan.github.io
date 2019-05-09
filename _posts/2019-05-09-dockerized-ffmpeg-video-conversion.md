---
layout: post
title: Dockerized ffmpeg video conversion
date: 2019-05-09
tags: docker ffmpeg video container
---

Easily convert large `.MOV` files to a compressed VP9 format using the [`jrottenberg/ffmpeg`](https://hub.docker.com/r/jrottenberg/ffmpeg/) docker image.

# Code

```bash
# Replace the following:
#  * Mounted directory (mine is /tmp/ff)
#  * Source filename (mine is is MVI_7493.MOV)
#  * CRF (quality) value - I used 30
# First pass output.webm
docker run -it \
    --mount type=bind,source=/tmp/ff,target=/tmp/workdir \
    --rm \
    jrottenberg/ffmpeg \
    -y \ 
    -i /tmp/workdir/MVI_7493.MOV \
    -c:v libvpx-vp9 -b:v 0 -crf 30 \
    -pass 1 \
    -f webm \
    /dev/null

# Second pass
docker run -it \
    --mount type=bind,source=/tmp/ff,target=/tmp/workdir \
    --rm \
    jrottenberg/ffmpeg \
    -y \
    -i /tmp/workdir/MVI_7493.MOV \
    -c:v libvpx-vp9 -b:v 0 -crf 30 \
    -pass 2 \
    -f webm \
    output.webm
```

# File size
```bash
1.5G May  9 21:23 MVI_7493.MOV
2.2M May  9 23:09 ffmpeg2pass-0.log
121M May 10 02:18 output.webm
```
The conversion took about 3 hours. The original is 1.5GB, the temporary log file is 2.2M, and the output is 121M.

# Discussion

My [Canon 550d DSLR](https://www.dpreview.com/products/canon/slrs/canon_eos550d)  saves video files using the `h264` format. The video files are quite big - 4.5 minutes take about 1.5 GB, and I want to share a smaller version in a royalty-free format.

I am not a professional video editor, so I decided to avoid the complexity of `ffmpeg` installation on my Mac and choose a proper Docker container instead. [Julien Rottenberg's image](https://github.com/jrottenberg/ffmpeg) seemed like a great choice - popular, starred, and [its crazy Dockerfile](https://hub.docker.com/r/jrottenberg/ffmpeg/dockerfile) is exactly what I was trying to avoid doing myself.

I took some [digging into the Dockerfile](https://github.com/jrottenberg/ffmpeg/blob/master/docker-images/4.1/alpine/Dockerfile#L14) to understand the mount point (which is `/tmp/workdir`), and from that point it's pretty much running the ffmpeg commands. 

# Two pass
Quoting the manual:

> Two-pass is the recommended encoding method for libvpx-vp9 as some quality-enhancing encoder features are only available in 2-pass mode.

In Two-pass mode, the first pass creates a temporary log file, and the second pass uses the log file and the original video to generate the final output. The only difference between the first and second pass commands are the output file (`/dev/null` in the first pass) and the `-pass 1`, `-pass 2` arguments respectively.

# Links
* [Docekrhub, jrottenberg/ffmpeg](https://hub.docker.com/r/jrottenberg/ffmpeg/)
* [FFmpeg and VP9 Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/VP9#constrainedq)