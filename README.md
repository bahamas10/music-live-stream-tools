Music Live Stream Tools
=======================

Tools made for live streaming music specifically on YouTube, but could be
modified to work on other platforms.

These tools are meant to be run on a headless (no gui) setup on a Unix or
Unix-like machine.  I'm currently using them on SmartOS but they should work on
pretty much any distro of Linux, MacOS, BSD, or whatever.

Requirements
------------

- `bash`
- `ffmpeg`
- `perl` (only for `get-subscribers`)
- Run these tools out of this directory - they are bash scripts that source
  files using relative locations.

Synopsis
--------

This repo contains 3 tools that are all being used to run my [continuous piano
live stream](https://www.youtube.com/channel/UC580SYuIdAIWf8ngzASdKGQ/live) on
YouTube.

- `shuffle-and-loop-music` takes a directory full of mp3 files, shuffles them,
  and continually "plays" them (decodes them with `ffmpeg`) to a named pipe.
  This program will decodes as fast as it can and is blocked by the consumer of
  the pipe.
- `stream-video` takes a named pipe to read audio data from, a video file
  to loop, and an optional  text file to render onto the video and streams the
  data to any RTMP stream.
- `get-subscribers` pulls the latest subscriber count for a YouTube channel
  which is then used to be rendered onto the video.

Example
-------

An example of how to setup a continuous music stream to YouTube would look
something like this:

    ./shuffle-and-loop-music ./mp3-dir ./audio-pipe ./nowplaying.txt

In another terminal:

    ./stream-video -c config

And bam - you will now be streaming music continuously to YouTube.  You can add
mp3s to the directory and they will be picked up automatically without any
programs needing to be restarted.

Note that the `audio-pipe` does *not* need to exist before running these tools.

Tools
-----

### `get-subscriber-count`

Get YouTube subscriber count (approximate) for a given channel id or name.

    Usage: get-subscriber-count <channel> [file] [interval] [cmd [args ...]]

Examples

With a single argument, the output is written to stdout:

    $ ./get-subscriber-count bahamas10
    2.93K subscribers

With a second argument, the output is written (atomically) to the file:

    $ ./get-subscriber-count bahamas10 foo.txt
    $ cat foo.txt
    2.93K subscribers

With a third argument, the output will is written (atomically) to the
file infinitely at the given interval:

    $ ./get-subscriber-count bahamas10 foo.txt 5
    [2021-09-04T21:02:17-0400] 2.93K subscribers - file=foo.txt sleeping for 5
    [2021-09-04T21:02:22-0400] 2.93K subscribers - file=foo.txt sleeping for 5

With a fourth argument, the output is written (atomically) to the
file infinitely at the given interval, and the command "cmd" is run with
any arguments given:

    $ ./get-subscriber-count bahamas10 foo.txt 5 echo hello world
    hello world
    [2021-09-10T23:45:03-0400] 2.96K subscribers written to foo.txt sleeping for 5

---

### `shuffle-and-loop-music`

Shuffle a directory full of mp3s and stream them to a given fifo pipe
continuously.  Allows for new tracks to be added without needing to restart this
process.

    Usage: shuffle-and-loop-music <mp3-dir> <pipe> [now-playing] [cmd [args ...]]

Examples

Shuffle the mp3 directory and output the raw data to the given pipe, and
optionally update the now-playing file with the current file name playing and
run the command `my-program` with the arguments `a b c`:

    $ ./shuffle-and-loop-music ../music/ ../pipes/audio-pipe ../data/now-playing.txt my-program a b c
    [2021-09-04T20:50:08-0400] beginning loop
    [2021-09-04T20:50:08-0400] playing 02-transpose-19.mp3

And in another terminal:

    $ cat ../data/now-playing.txt
    02-transpose-19.mp3

---

### `stream-video`

The main program to stream to an RTMP stream (like YouTube or Twitch).  This
program is responsible for:

- Reading in a looping background movie or image.
- Reading in a continuous stream of music through a fifo.
- Writing text to the video as an overlay (optional).
- Streaming the output video to an RTMP stream (or stdout if `-` is given).

```
$ ./stream-video -h
Usage: stream-video [options] [output]

Options:
  -c <config>    required: config file to read
  -n             optional: dry-run, just print the command and exit

Arguments
  - Specifying '-' as the last argument will tell this program to output
    to stdout instead of streaming to youtube.  This can then be piped
    to ffplay for testing like:

      stream-video ... - | ffplay -f flv -
```

An example config file is included in this repo as `config.dist`:

``` bash
#!/usr/bin/env bash
#
# Config file for the main streaming program.  This is a bash file that is
# sourced.
#

# output image settings
resolution='1920x1080'
preset='ultrafast'
tune='stillimage'
bitrate='3500k'
maxrate='5000k'
bufsize='8000k'
fps=6

# input/output audio settings
audiochannels=2
audiobits='192k'
audiorate=44100
audiopipe='/path/to/audio-pipe-fifo'

# background image or video
bgtype='image'
bgfile='/path/to/image.jpg'

# text overlay options, set "textoverlay=1" to enable
textoverlay=1
fontcolor='#00aa00'
fontsize=42
textlocation='x=40:y=38'
fontfile='/path/to/font.ttf'
textfile='/path/to/message.txt'

# output RTMP stream settings
rtmpurl='rtmp://a.rtmp.youtube.com/live2/<key>'
```

License
-------

MIT License
