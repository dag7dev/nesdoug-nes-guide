# Appendix, neslib library

## 

Shiru wrote the neslib code, for NES development. These are all my detailed notes on how everything works. I will be adding example code, a little later. I mostly use a slightly modified version of the neslib from

[http://shiru.untergrund.net/files/nes/chase.zip](http://shiru.untergrund.net/files/nes/chase.zip)

And, here again is the example code

[http://shiru.untergrund.net/files/src/cc65\_nes\_examples.zip](http://shiru.untergrund.net/files/src/cc65_nes_examples.zip)

And this link has a version of neslib that works with the most recent version of cc65 \(as of 2016\) version 2.15

[http://forums.nesdev.com/viewtopic.php?p=154078\#p154078](http://forums.nesdev.com/viewtopic.php?p=154078#p154078)

### Palette

**pal\_all\(const char \*data\);**

const unsigned char game\_palette\[\]={…} // define a 32 byte array of chars  
 pal\_all\(game\_palette\);

-pass a pointer to a 32 byte full palette  
 -it will copy 32 bytes from there to a buffer  
 -can be done any time, this only updates during v-blank

**pal\_bg\(bg\_palette\);** // 16 bytes only, background

**pal\_spr\(sprite\_palette\);** // 16 bytes only, sprites  
 -same as pal\_all, but 16 bytes

**pal\_col\(unsigned char index,unsigned char color\);**  
 -sets only 1 color in any palette, BG or Sprite  
 -can be done any time, this only updates during v-blank  
 -index = 0 – 31 \(0-15 bg, 16-31 sprite\)

\#define RED 0x16  
 pal\_col\(0, RED\); // would set the background color red  
 pal\_col\(0, 0x30\); // would set the background color white = 0x30

pal\_col\(\) might be useful for rotating colors \(SMB coins\), or blinking a sprite  
 NOTE: palette buffer is set at 0x1c0-0x1df in example code  
 PAL\_BUF =$01c0, defined somewhere in crt0.s  
 -this is in the hardware stack. If subroutine calls are more than 16 deep, it will start to overwrite the buffer, possibly causing wrong colors or game crashing

**pal\_clear\(void\);** // just sets all colors to black, can be done any time

**pal\_bright\(unsigned char bright\);** // brightens or darkens all the colors  
 – 0-8, 4 = normal, 3 2 1 darker, 5 6 7 lighter  
 – 0 is black, 4 is normal, 8 is white  
 pal\_bright\(4\); // normal

NOTE: pal\_bright\(\) must be called at least once during init \(and it is, in crt0.s\). It sets a pointer to colors that needs to be set for the palette update to work.

Shiru has a fading function in the Chase source code game.c

```text
void pal_fade_to(unsigned to)
{
  if(!to) music_stop();
  while(bright!=to)
  {
    delay(4);
    if(bright<to) ++bright; 
    else --bright;
    pal_bright(bright);
  }
  if(!bright)
  {
    ppu_off();
    set_vram_update(NULL);
    scroll(0,0);
  }
}
```

**pal\_spr\_bright\(unsigned char bright\);**  
 -sets sprite brightness only

**pal\_bg\_bright\(unsigned char bright\);**  -sets BG brightness , use 0-8, same as pal\_bright\(\)

**ppu\_wait\_nmi\(void\);**  
 -wait for next frame

**ppu\_wait\_frame\(void\);**  
 -it waits an extra frame every 5 frames, for NTSC TVs  
 -do not use this, I removed it  
 -potentially buggy with split screens

**ppu\_off\(void\);** // turns off screen

**ppu\_on\_all\(void\);** // turns sprites and BG back on

**ppu\_on\_bg\(void\);** // only turns BG on, doesn’t affect sprites  
 **ppu\_on\_spr\(void\);** // only turns sprites on, doesn’t affect bg

**ppu\_mask\(unsigned char mask\);** // sets the 2001 register manually, see nesdev wiki  
 -could be used to set color emphasis or grayscale modes

ppu\_mask\(0x1e\); // normal, screen on  
 ppu\_mask\(0x1f\); // grayscale mode, screen on  
 ppu\_mask\(0xfe\); // screen on, all color emphasis bits set, darkening the screen

**ppu\_system\(void\);** // returns 0 for PAL, !0 for NTSC

-during init, it does some timed code, and it figures out what kind of TV system is running. This is a way to access that information, if you want to have it programmed differently for each type of TV  
 -use like…  
 a = ppu\_system\(\);

### Sprites

**oam\_clear\(void\);** // clears the OAM buffer, making all sprites disappear

OAM\_BUF =$0200, defined somewhere in crt0.s

**oam\_size\(unsigned char size\);** // sets sprite size to 8×8 or 8×16 mode

oam\_size\(0\); // 8×8 mode  
 oam\_size\(1\); // 8×16 mode

NOTE: at the start of each loop, set sprid to 0

sprid = 0;

then every time you push a sprite to the OAM buffer, it returns the next index value \(sprid\)

**oam\_spr\(unsigned char x,unsigned char y,unsigned char chrnum,unsigned char attr,unsigned char sprid\);**  
 -returns sprid \(the current index to the OAM buffer\)  
 -sprid is the number of sprites in the buffer times 4 \(4 bytes per sprite\)

sprid = oam\_spr\(1,2,3,0,sprid\);  
 -this will put a sprite at X=1,Y=2, use tile \#3, palette \#0, and we’re using sprid to keep track of the index into the buffer

sprid = oam\_spr \(1,2,3,0\|OAM\_FLIP\_H,sprid\); // the same, but flip the sprite horizontally  
 sprid = oam\_spr \(1,2,3,0\|OAM\_FLIP\_V,sprid\); // the same, but flip the sprite vertically  
 sprid = oam\_spr \(1,2,3,0\|OAM\_FLIP\_H\|OAM\_FLIP\_V,sprid\); // the same, but flip the sprite horizontally and vertically  
 sprid = oam\_spr \(1,2,3,0\|OAM\_BEHIND,sprid\); // the sprite will be behind the background, but in front of the universal background color \(the very first bg palette entry\)

**oam\_meta\_spr\(unsigned char x,unsigned char y,unsigned char sprid,const unsigned char \*data\);**  
 -returns sprid \(the current index to the OAM buffer\)  
 -sprid is the number of sprites in the buffer times 4 \(4 bytes per sprite\)

sprid = oam\_meta\_spr\(1,2,sprid, metasprite1\)

const unsigned char metasprite1\[\] = …; // definition of the metasprite, array of chars

This would put the metasprite at a relative location of x=1,y=2

A metasprite is a collection of sprites  
 -you can’t flip it so easily  
 -you can make a metasprite with nes screen tool  
 -it’s an array of 4 bytes per tile =  
 -x offset, y offset, tile, attribute \(per tile palette/flip\)  
 -you have to pass a pointer to this data array  
 -the data set needs to terminate in 128 \(0x80\)  
 -during each loop \(frame\) you will be pushing sprites to the OAM buffer  
 -they will automatically go to the OAM during v-blank \(part of nmi code\)

**oam\_hide\_rest\(unsigned char sprid\);**  
 -pushes the rest of the sprites off screen  
 -do at the end of each loop

-necessary, if you don’t clear the sprites at the beginning of each loop  
 -if \# of sprites on screen is exactly 64, the sprid value would wrap around to 0, and this function would accidentally push all your sprites off screen \(passing 0 will push all sprites off screen\)  
 -if for some reason you pass a value not divisible by 4 \(like 3\), this function would crash the game in an infinite loop  
 -it might be safer, then, to just use oam\_clear\(\) at the start of each loop, and never call oam\_hide\_rest\(\)

### 

### Music

Note, in crt0.s, the music data is included after music\_data: and the init code initializes the music system.

**music\_play\(unsigned char song\);** // send it a song number, it sets a pointer to the start of the song, will play automatically, updated during v-blank  
 music\_play\(0\); // plays song \#0

**music\_stop\(void\);** // stops the song, must do music\_play\(\) to start again, which will start the beginning of the song

**music\_pause\(unsigned char pause\);** // pauses a song, and unpauses a song at the point you paused it

music\_pause\(1\); // pause  
 music\_pause\(0\); // unpause

**sfx\_play\(unsigned char sound,unsigned char channel\);** // sets a pointer to the start of a sound fx, which will auto-play

sfx\_play\(0, 0\); // plays sound effect \#0, priority \#0

channel 3 has priority over 2,,,,,, 3 &gt; 2 &gt; 1 &gt; 0. If 2 sound effects conflict, the higher priority will play.

**sample\_play\(unsigned char sample\);** // play a DMC sound effect

sample\_play\(0\); // play DMC sample \#0

### 

### Controllers

**pad\_poll\(unsigned char pad\);**  
 -reads a controller  
 -have to send it a 0 or 1, one for each controller  
 -do this once per frame

pad1 = pad\_poll\(0\); // reads contoller \#1, store in pad1  
 pad2 = pad\_poll\(1\); // reads contoller \#2, store in pad2

**pad\_trigger\(unsigned char pad\);** // only gets new button presses, not if held

a = pad\_trigger\(0\); // read controller \#1, return only if new press this frame  
 b = pad\_trigger\(1\); // read controller \#2, return only if new press this frame

-this actually calls pad\_poll\(\), but returns only new presses, not buttons held

**pad\_state\(unsigned char pad\);**  
 -get last poll without polling again  
 -do pad\_poll\(\) first, every frame  
 -this is so you have a consistent value all frame  
 -can do this multiple times per frame and will still get the same info

pad1 = pad\_state\(0\); // controller \#1, get last poll  
 pad2 = pad\_state\(1\); // controller \#2, get last poll

### **Scrolling**

It is expected that you have 2 int’s defined \(2 bytes each\), ScrollX and ScrollY.  
 You need to manually keep them from 0 to 0x01ff \(0x01df for y, there are only 240 scanlines, not 256\)  
 In example code 9, shiru does this

– -y;

if\(y&lt;0\) y=240\*2-1; // keep Y within the total height of two nametables

**scroll\(unsigned int x,unsigned int y\);**  
 -sets the x and y scroll. can do any time, the numbers don’t go to the 2005 registers till next v-blank  
 -the upper bit changes the base nametable, register 2000 \(during the next v-blank\)  
 -assuming you have mirroring set correctly, it will scroll into the next nametable.

scroll\(scroll\_X,scroll\_Y\);

\*note, I don’t use this scroll function, but my own similar ones, where you don’t need to keep Y between 0 and $1df.

**split\(unsigned int x,unsigned int y\);**  
 -waits for sprite zero hit, then changes the x scroll  
 -will only work if you have a sprite currently in the OAM at the zero position, and it’s somewhere on-screen with a non-transparent portion overlapping the non-transparent portion of a BG tile.

-i’m not sure why it asks for y, since it doesn’t change the y scroll  
 -it’s actually very hard to do a mid-screen y scroll change, so this is probably for the best  
 -warning: all CPU time between the function call and the actual split point will be wasted!  
 -don’t use ppu\_wait\_frame\(\) with this, you might have glitches

### **Tile banks**

-there are 2 sets of 256 tiles loaded to the ppu, ppu addresses 0-0x1fff  
 -sprites and bg can freely choose which tileset to use, or even both use the same set

**bank\_spr\(unsigned char n\);** // which set of tiles for sprites

bank\_spr\(0\); // use the first set of tiles  
 bank\_spr\(1\); // use the second set of tiles

**bank\_bg\(unsigned char n\);** // which set of tiles for background

bank\_bg\(0\); // use the first set of tiles  
 bank\_bg\(1\); // use the second set of tiles

**rand8\(void\);** // get a random number 0-255  
 a = rand8\(\); // a is char

**rand16\(void\);** // get a random number 0-65535  
 a = rand16\(\); // a is int

**set\_rand\(unsigned int seed\);** // send an int \(2 bytes\) to seed the rng

-note, crt0 init code auto sets the seed to 0xfdfd  
 -you might want to use another seeding method, if randomness is important, like checking FRAME\_CNT1 at the time of START pressed on title screen

### Writing to the screen

**set\_vram\_update\(unsigned char \*buf\);**  
 -sets a pointer to an array \(a VRAM update buffer, somewhere in the RAM\)  
 -when rendering is ON, this is how BG updates are made

usage…  
 set\_vram\_update\(Some\_ROM\_Array\); // sets a pointer to the data in ROM

also…  
 set\_vram\_update\(NULL\);  
 -to disable updates, call this function with NULL pointer

The vram buffer should be filled like this…

**Non-sequential:**  
 -non-sequential means it will set a PPU address, then write 1 byte  
 -MSB, LSB, 1 byte data, repeat  
 -sequence terminated in 0xff \(NT\_UPD\_EOF\)

MSB = high byte of PPU address  
 LSB = low byte of PPU address

**Sequential:**  
 -sequential means it will set a PPU address, then write more than 1 byte to the ppu  
 -left to right \(or\) top to bottom  
 -MSB\|NT\_UPD\_HORZ, LSB, \# of bytes, a list of the bytes, repeat  
 or  
 -MSB\|NT\_UPD\_VERT, LSB, \# of bytes, a list of the bytes, repeat  
 -NT\_UPD\_HORZ, means it will write left to right, wrapping around to the next line  
 -NT\_UPD\_VERT, means is will write top to bottom, but a new address needs to be set after it reaches the bottom of the screen, as it will never wrap to the next column over  
 -sequence terminated in 0xff \(NT\_UPD\_EOF\)

\#define NT\_UPD\_HORZ 0x40 = sequential  
 \#define NT\_UPD\_VERT 0x80 = sequential  
 \#define NT\_UPD\_EOF 0xff

Example of 4 sequential writes, left to right, starting at screen position x=1,y=2  
 tile \#’s are 5,6,7,8  
 {  
 MSB\(NTADR\_A\(1,2\)\)\|NT\_UPD\_HORZ,  
 LSB\(NTADR\_A\(1,2\)\),  
 4, // 4 writes  
 5,6,7,8, // tile \#’s  
 NT\_UPD\_EOF  
 };

Interestingly, it will continually write the same data, every v-blank, unless you send a NULL pointer like this…  
 set\_vram\_update\(NULL\);  
 …though, it may not make much difference.  
 The data set \(aka vram buffer\) must not be &gt; 256 bytes, including the ff at the end of the data, and should not push more than…I don’t know, maybe \* bytes of data to the ppu, since this happens during v-blank and not during rendering off, time is very very limited.

> \* Max v-ram changes per frame, with rendering on, before BAD THINGS start to happen…
>
> sequential max = 97 \(no palette change this frame\),  
>  74 \(w palette change this frame\)
>
> non-sequential max = 40 \(no palette change this frame\),  
>  31 \(w palette change this frame\)
>
> the buffer only needs to be…  
>  3 \* 40 + 1 = 121 bytes in size  
>  …as it can’t push more bytes than that, during v-blank.
>
> \(this hasn’t been tested on hardware, only FCEUX\)

// all following vram functions only work when display is disabled

**vram\_adr\(unsigned int adr\);**  
 -sets a PPU address  
 \(sets a start point in the background for writing tiles\)

vram\_adr\(NAMETABLE\_A\); // start at the top left of the screen  
 vram\_adr\(NTADR\_A\(x,y\)\);  
 vram\_adr\(NTADR\_A\(5,6\)\); // sets a start position x=5,y=6

**vram\_put\(unsigned char n\);** // puts 1 byte there  
 -use vram\_adr\(\); first

vram\_put\(6\); // push tile \# 6 to screen

**vram\_fill\(unsigned char n,unsigned int len\);** // repeat same tile \* LEN  
 -use vram\_adr\(\); first  
 -might have to use vram\_inc\(\); first \(see below\)

vram\_fill\(1, 0x200\); // tile \# 1 pushed 512 times

**vram\_inc\(unsigned char n\);** // mode of ppu  
 vram\_inc\(0\); // data gets pushed into vram left to right \(wraping to next line\)  
 vram\_inc\(1\); // data gets pushed into vram top to bottom \(only works for 1 column \(30 bytes\), then you have to set another address\).  
 -do this BEFORE writing to the screen, if you need to change directions

**vram\_read\(unsigned char \*dst,unsigned int size\);**  
 -reads a byte from vram  
 -use vram\_adr\(\); first  
 -dst is where in RAM you will be storing this data from the ppu, size is how many bytes

vram\_read\(0x300, 2\); // read 2 bytes from vram, write to RAM 0x300

NOTE, don’t read from the palette, just use the palette buffer at 0x1c0

**vram\_write\(unsigned char \*src,unsigned int size\);**  
 -write some bytes to the vram  
 -use vram\_adr\(\); first  
 -src is a pointer to the data you are writing to the ppu  
 -size is how many bytes to write

vram\_write\(0x300, 2\); // write 2 bytes to vram, from RAM 0x300  
 vram\_write\(TEXT,sizeof\(TEXT\)\); // TEXT\[\] is an array of bytes to write to vram.  
 \(For some reason this gave me an error, passing just an array name, had to cast to char \* pointer\)  
 vram\_write\(\(unsigned char\*\)TEXT,sizeof\(TEXT\)\);

**vram\_unrle\(const unsigned char \*data\);**  
 -pass it a pointer to the RLE data, and it will push it all to the PPU.  
 -this unpacks compressed data to the vram  
 -this is what you should actually use most…this is what NES screen tool outputs best.  
 vram\_unrle\(titleRLE\);

usage:  
 -first, disable rendering, ppu\_off\(\);  
 -set vram\_inc\(0\) and set a starting address, vram\_adr\(\)  
 -call vram\_unrle\(\);  
 -then turn rendering back on, ppu\_on\_all\(\)  
 -only load 1 nametable worth of data, per frame

NOTE:  
 -nmi is turned on in init, and never comes off

**memcpy\(void \*dst,void \*src,unsigned int len\);**  
 -moves data from one place to another…usually from ROM to RAM

memcpy\(update\_list,updateListData,sizeof\(updateListData\)\);

**memfill\(void \*dst,unsigned char value,unsigned int len\);**  
 -fill memory with a value

memfill\(0x200, 1, 0x100\);  
 -to fill 0x200-0x2ff with tile \#1…that is 0x100 bytes worth of filling

**delay\(unsigned char frames\);** // waits a \# of frames

delay\(5\); // wait 5 frames

**TECHNICAL NOTES, ON ASM BITS IN NESLIB.S:**  
 -vram \(besides the palette\) is only updated if VRAM\_UPDATE + NAME\_UPD\_ENABLE are set…  
 -ppu\_wait\_frame \(or\) ppu\_wait\_nmi, sets ‘UPDATE’  
 -set\_vram\_update, sets ‘ENABLE’  
 -set\_vram\_update\(0\); disables the vram ‘UPDATE’  
 -I guess you can’t set a pointer to the zero page address 0x0000, or it will never update.  
 -music only plays if FT\_SONG\_SPEED is set, play sets it, stop resets it, pause sets it to negative \(ORA \#$80\), unpause clears that bit

