# cdgmaker

Create custom CDG files for karaoke.

In August 2024, I had the idea to make custom karaoke files for songs I wanted
to perform at a local karaoke bar, but couldn’t. And starting in September 2024,
[that idea was real](https://winslowjosiah.com/blog/2024/09/29/i-made-a-cdg-karaoke-maker/).

This is the custom Python program I wrote in order to do it. It's not the most
featureful or easy-to-use thing in the world, but it's robust enough to make
[decent karaoke files](https://winslowjosiah.com/karaoke/). Plus, it's free!

## Usage

My program is still somewhat of a work in progress, but the way I have it
working right now is with configuration files in the [TOML](https://toml.io/)
format. To run my program, run it as a module, and pass the TOML config file as
its first argument.

```bash
py -m cdgmaker song.toml
```

In the same directory as that config file, a `.zip` file should be created, with
a `.cdg` file (for graphics) and an `.mp3` file (for music). These files should
be compatible with most karaoke players, including [Karafun Player](https://www.karafun.com/karaokeplayer/)
and (my personal favorite) [Karaoke Builder Player](https://www.karaokebuilder.com/kbplayer.php).

You can find some example songs for it [here](https://winslowjosiah.com/blog/2024/09/29/i-made-a-cdg-karaoke-maker/examples.zip).

If you specify the `-r` flag, a `.mp4` file and `.ass` subtitle file will also
be created (note: requires FFmpeg).

```bash
py -m cdgmaker song.toml -r
```

----

Note that the files will have an 8-second intro and outro, the style of which is
hardcoded and unchangeable at the moment. I'm still brainstorming ways to make
the format of the intro screen easily configurable.

> [!NOTE]
> Time values are in the form of centiseconds (1/100ths of a second). For
> example, a value of 500 corresponds to 5 seconds, and a value of 314
> corresponds to 3.14 seconds. This is the same convention Karafun Studio uses
> for its timing values.
>
> Color values are in the form of strings. These strings can be in any of the
> following formats:
>
> - 6-digit hex (e.g. `"#663399"`)
> - 3-digit hex (e.g. `"#639"`)
> - RGB functions (e.g. `"rgb(102, 51, 153)"`)
> - HSL functions (e.g. `"hsl(270, 50%, 40%)"`)
> - HSV functions (e.g. `"hsv(270, 67%, 60%)"`)
> - HTML color names (e.g. `"rebeccapurple"`)

### Top-level values

- **title**: Title of song. Can have newlines.
- **artist**: Artist of song. Can have newlines.
- **file**: Audio file of song. Format can be any format supported by ffmpeg.
- **outname** _(default `"output"`)_: Name of output files (without extension).
    E.g. if **outname** is `"song"`, the output will be a ZIP file called
    `song.zip`, which contains `song.cdg` and `song.mp3`.
- **clear_mode** _(default `"delayed"`)_: Any of the following values:
    - `"page"` (clear screen after each page)
    - `"delayed"` (wait to draw new lines as long as possible)
    - `"eager"` (draw new lines as soon as possible).
- **sync_offset** _(default 0)_: The offset from which to start the sync. Each
    sync point will be brought forward in time by this amount.
- **highlight_bandwidth** _(default 1)_: The relative number of packets used for
    highlighting text during each update. If this number is higher than
    **draw_bandwidth**, more time is spent on highlighting than drawing/erasing.
    Both should ideally be as low as possible.
- **draw_bandwidth** _(default 1)_: The relative number of packets used for
    drawing/erasing text during each update. If this number is higher than
    **highlight_bandwidth**, more time is spent on drawing/erasing than
    highlighting. Both should ideally be as low as possible.
- **background** _(default `"black"`)_: Background color.
- **border** _(default `"black"`)_: Border color. An empty string means that the
    border is not updated with a color.
- **font**: Font of lyric text. This is a path to a TTF font file.
- **font_size** _(default 18)_: Font size of lyric text.
- **stroke_width** _(default 0)_: Width of lyric text stroke.
- **stroke_type** _(default `"octagon"`)_: Any of the following values:
    - `"circle"`
    - `"square"`
    - `"octagon"`

### Singers (list)

A song can have up to 3 "singers", which basically specify the colors of a text
line. These are defined in a list called `singers` at the top level.

- **active_fill**: Fill color of highlighted lyric text.
- **active_stroke**: Stroke color of highlighted lyric text.
- **inactive_fill**: Fill color of non-highlighted lyric text.
- **inactive_stroke**: Stroke color of non-highlighted lyric text.

### Lyrics (list)

A song can have multiple sets of "lyrics", which are a series of words and
timings drawn in a specific place on the screen. These are defined in a list
called `lyrics` at the top level.

- **singer** _(default 1)_: The singer of these lyrics. The singer's color will
    be applied to every line, except for lines where the singer is specified.
- **lines_per_page**: The number of lines on each page for these lyrics.
- **row** _(default 1)_: The tile row where the first line of each page is
    drawn. Row 0 is the top row, and row 17 is the bottom row. Recommended to be
    between 1 and 16 inclusive.
- **line_tile_height**: The height of each line, in tile rows. Should be at
    least large enough to fit a line of text, but can be bigger if space between
    lines is desired.
- **text**: The text of these lyrics.

    Each word should be divided into syllables using the `/` character (e.g.
    `Wel/come to the Ho/tel Ca/li/for/nia`).

    A space character is interpreted as a word break (and thus a syllable
    division). Use the `_` character to signify a space without introducing a
    word break (e.g. `Je_veux ton a/mour et je veux ton re/vanche`).

    A singer can be specified for a line by starting the line with the singer
    number (from 1 to 3), followed by the `|` character (e.g. `1|Tell me why`;
    `2|Ain't no/thin' but a heart/ache`). There can be spaces around the `|`
    character.

    A line without any syllables is signified with the `~` character. Blank
    lines are ignored.

- **sync**: A list of syllable start times, one for each syllable.

> [!TIP]
> Timings are defined for the starts of syllables, but not the ends. Within a
> line, each syllable ends when the next syllable starts, and the last
> syllable of a line will end up to 0.45 seconds after it starts.
>
> To define a different endpoint for a syllable, add a blank syllable after it
> and set the timing for that blank syllable.

----

> [!NOTE]
> Multiple sets of lyrics can be used for simultaneous duets, where multiple
> singers are singing at the same time, but with different words or timings.
>
> However, if the song is an alternating duet (where multiple singers are
> singing, but not at the same time), or the singers are singing the same
> words at the same time when singing together, it is better to use one set of
> lyrics and specify different "singers" for each line.
>
> Here's an example, with the lyrics of [Summer Nights (from "Grease")](https://www.youtube.com/watch?v=A_J2bcNx3Gw).
> Singer 1 is the male part, singer 2 is the female part, and singer 3 is for
> both the male and female simultaneously.
>
>     :::text
>     1 | Sum/mer lov/in',
>     1 | had me a blast/
>     2 | Sum/mer lov/in',
>     2 | hap/pened so fast/
>     1 | I met a girl
>     1 | cra/zy for me/
>     2 | Met a boy,
>     2 | cute as can be/
>     3 | Sum/mer days
>     3 | drift/in' a/way
>     3 | To, uh oh,
>     3 | those sum/mer nights

### Instrumentals (list)

A song can have one or more "instrumental" segments, which are intended for
sections of the song without any actively sung lyrics. These are defined in a
list called `instrumentals` at the top level.

- **sync**: The time at which this instrumental should appear. If this
    instrumental comes after sung lyrics, **sync** should be on or before the
    last syllable sung on the last line before this instrumental.
- **wait** _(default true)_: If true, wait until the currently sung syllable is
    sung (and its line is erased, if it will be erased) before showing this
    instrumental. If false, do not wait for the currently sung syllable. (If
    this instrumental is immediately after a different instrumental, **wait**
    should be false.)
- **text** _(default `"INSTRUMENTAL"`)_: The text to display during the
    instrumental. Can have newlines.
- **text_align** _(default `"center"`)_: Any of the following values:
    - `"left"`
    - `"center"`
    - `"right"`
- **text_placement** _(default `"middle"`)_: Any of the following values:
    - `"top left"`
    - `"top middle"`
    - `"top right"`
    - `"middle left"`
    - `"middle"`
    - `"middle right"`
    - `"bottom left"`
    - `"bottom middle"`
    - `"bottom right"`
- **line_tile_height**: The height of each line, in tile rows. Should be at
    least large enough to fit a line of text, but can be bigger if space between
    lines is desired.
- **fill** _(default `"#bbb"`)_: Fill color of text.
- **stroke** _(default `""`)_: Stroke color of text. An empty string means that
    the text stroke is not drawn.
- **background** _(default `""`)_: Background color. An empty string means that
    the main background color is used instead.
- **image**: Image to display during the instrumental. This is a path to an
    image. If not specified, an image is not drawn.
- **x** _(default 0)_: The X position at which to draw the image, if any.
- **y** _(default 0)_: The Y position at which to draw the image, if any.
- **transition**: Any of a few different values, corresponding to the images in
    the `transitions` folder. As of now, they include:
    - `"circlein"`
    - `"circleout"`
    - `"fizzle"`
    - `"rectangle"`
    - `"spiral"`
    - `"wipein"`
    - `"wipeleft"`
    - `"wipeout"`
    - `"wiperight"`

    These signify a certain order in which to draw the tiles of the image, if
    any. If not specified, the tiles are drawn in a default order.
