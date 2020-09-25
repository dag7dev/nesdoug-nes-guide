# What you need

## What you need

This is what you will need to program an NES game…

1. An assembler \(or compiler\)  
 2. A tile editor  
 3. Photoshop or GIMP \(or similar\)  
 4. A text editor, like Notepad++  
 5. A good NES emulator  
 6. A tile arranging program  
 7. python 3 \(optional, but I use it\)  
 8. a music tracker

### 6502 Assembler

For my examples I will exclusively work with cc65. It is part of a suite of software, and is one of the best compiler/assembler programs available for 6502 \(i.e. NES\) programming. Although, the learning curve is a bit steep, I will help get you started.

[http://cc65.github.io/cc65/](http://cc65.github.io/cc65/)

\(Click on Windows snapshot\)

NOTE: the version that I use is V2.15 \(type

```text
cc65 --version
```

in command line to see your version\). Files from different versions of cc65 can sometimes give error messages \(especially much older versions of cc65 may not compile\).

### Tile Editor

You need a tile editor to create the graphics. I personally prefer YY-CHR. You can get it here…\(this is the updated and improved version\).

[https://w.atwiki.jp/yychr/sp/](https://w.atwiki.jp/yychr/sp/)

I prefer to work first in GIMP/Photoshop, convert to indexed \(4 color\), then copy/paste over to YY-CHR later. You should be working in the 2bpp NES format \(the default\).

Here is a link where you can download GIMP.

[https://www.gimp.org/downloads/](https://www.gimp.org/downloads/)

### Text Editor

Notepad++ is a tool for writing programming code. You could use any text editor, if you like. Notepad++ is available here…

[https://notepad-plus-plus.org/download/](https://notepad-plus-plus.org/download/)

The great thing about Notepad++ is you can set it to highlight your code, which makes it easier to read. And it has the line \# on the left. If you get error messages while compiling, it will tell you the line of the error. If you double-click on a word, it will highlight every instance of that word. It has a find-and-replace feature which I use a lot, also find-in-file to search an entire folder’s files for a word.

![NotepadPP2](https://nesdoug.files.wordpress.com/2018/09/notepadpp2.png?w=924)

Or, I’ve heard of people using VSCode to write their C code. It is a bit more advanced, and can do syntax error checking and code completion, which notepad++ can’t do.

### NES Emulator

And, next is an NES emulator. I use FCEUX 90% of the time because it has excellent debugging tools, PPU viewer, Nametable viewer, Hex Editor, etc. However, it is not the most accurate emulator. You may wish to test your game on multiple emulators, to assure that other people will be able to play your game without issues. \(I’ve used Nintendulator, Nestopia, and Mesen\). Actually, Mesen is probably better than FCEUX now \(it definitely has more debugging tools\). Get that too.

FCEUX is here…

[http://www.fceux.com/web/download.html](http://www.fceux.com/web/download.html)

Mesen is here…

[https://www.mesen.ca/\#Downloads](https://www.mesen.ca/#Downloads)

\* here’s where you can get a custom palette — FirebrandX has been working on making a better NES palette. What’s wrong with FCEUX’s default palette? Too bright and too saturated — not accurate to what it would look like on an actual NES. \(FCEUX / Config / Palette / Load Palette\).

[http://www.firebrandx.com/nespalette.html](http://www.firebrandx.com/nespalette.html)

I don’t see the exact palette I mentioned anymore, but I think it was this one…

[http://dl.dropboxusercontent.com/s/y3yeaqc87dnhqel/FBX-NES-Unsaturated.pal](http://dl.dropboxusercontent.com/s/y3yeaqc87dnhqel/FBX-NES-Unsaturated.pal)

You will probably want to change the pixel display to display every pixel. I’ve seen people say “the NES is 256×224 pixels”, but that is not true. Older TVs tended to cut off a few pixels from the top/bottom of the picture, but the NES generates a full 240 pixels high. One of my TVs displays nearly the entire 240 pixels. You should assume that some users will see the entire picture, so in FCEUX go to Config/Video/Drawing Area, and set the output to the full 0 to 239.

### Tile Arranger

Then, we need a tile arranging program. We can make a game without it, but it will definitely be helpful. Since we are working on an NES game, I highly recommend, **NES Screen Tool**. It shows the NES color limitations very well, and is good for making single screen games. It also gives you nametable addresses and attribute table addresses, which comes in handy. I use 2.3, and if you have an older version, it won’t open the .nss files in my source code. Get it here…

[https://shiru.untergrund.net/software.shtml](https://shiru.untergrund.net/software.shtml)

And, if you are making a scrolling game, I would also pick up **Tiled map editor**. I will go into more detail later, but you can make a data array out of the exported .csv files from Tiled.

[http://www.mapeditor.org/](http://www.mapeditor.org/)

Now that that’s done…how do these things work?

Photoshop – to prepare files to go to YY-CHR. First, resize to some reasonable NES size, here I’m using 128 x 128 pixels \(use nearest neighbor for resizing\). Then reduce to 4 colors, by Image/Mode/Indexed…Palette:Custom, reduce to 4 colors. \(I made a custom 4 color swatch set, that can be loaded here.\) You may have to touch up the image with the pencil tool. Cut and paste into YY-CHR.

![IndexColor](https://nesdoug.files.wordpress.com/2015/11/indexcolor.png?w=924)

YY-CHR – make sure it’s set on 2bpp \(NES\). You may have to use the color replacement tool, in YY-CHR, if it got the color indexing wrong…

![YYchr](https://nesdoug.files.wordpress.com/2015/11/yychr.png?w=924)

It doesn’t matter what palette settings you give it here. YY-CHR can show you various color options, but doesn’t save the palette. You will have to program the palette into your game.

You can load the chr file into NES Screen Tool, and create backgrounds, palettes, and sprites using just this tool, in a format that cc65 can understand and neslib can use. The newest version of NES Screen Tool can also import graphics \(BMP indexed 16 color\) as a tileset, or import it as a nametable \(auto generates tiles\). Kasumi has explained this process better than I could, here…

[http://nesmakers.com/viewtopic.php?t=189](http://nesmakers.com/viewtopic.php?t=189)

### Python

One more thing. I have been writing simple **python 3** scripts to process some of the data into C arrays. You don’t need to, but it might be helpful if you installed python 3, to use my tutorial files. I just use simple scripts for “automating the boring stuff”.

[https://www.python.org/downloads/](https://www.python.org/downloads/)

### Music Tracker

I don’t want to go into too much detail yet, but you will also need to get Famitracker for making music and/or sound effects for your game.

[http://famitracker.com/downloads.php](http://famitracker.com/downloads.php)

### Next Time…

cc65 – will take some explaining…next time.

