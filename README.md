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
- `stream-to-youtube` takes a named pipe to read audio data from, a video file
  to loop, and a text file to render onto the video and streams the data to
  YouTube.
- `get-subscribers` pulls the latest subscriber count for a YouTube channel
  which is then used to be rendered onto the video.

Example
-------

An example of how to setup a continuous music stream to YouTube would look
something like this:

    ./shuffle-and-loop-music ./mp3-dir ./audio-pipe ./nowplaying.txt

In another terminal:

    ./stream-to-youtube -a ./audio-pipe -b ./background.mp4 -f ./SomeFreeFont.ttf -t ./nowplaying.txt -k ./youtube-stream-key.txt

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

### `stream-to-youtube`

The main program to stream to YouTube.  This program is responsible for:

- Reading in a looping background movie (`-b`)
- Reading in a continuous stream of music through a fifo (`-a`)
- Writing the current subcount (any text in `-t`, using font `-f`) to the output
  video
- Streaming the output video to YouTube (using `-k` with the stream key data)

```
Usage: stream-to-youtube [options] [output]

Options:
  -a <fifo>      required: pipe to read audio input from (generated by shuffle-and-loop-music)
  -b <file.mp4>  required: background video, should match resolution set in this program (1920x1080)
  -f <font.ttf>  required: font file to use for drawing text
  -k <key.txt>   optional: youtube stream key in a text file, optional if output is set to stdout for testing
  -t <text.txt>  required: text file to read text to render to the video - updated dynamically

Arguments
  - Specifying '-' as the last argument will tell this program to output to stdout instead of
    streaming to youtube.  This can then be piped to ffplay for testing like:

      stream-to-youtube ... - | ffplay -f flv -
```

More options can be customized inside the script itself.  Specifying `-` as the
last argument will tell this program to output to stdout instead of streaming to
YouTube.  This can then be piped to `ffplay` for testing like:

    stream-to-youtube ... - | ffplay -f flv -

License
-------

MIT License
