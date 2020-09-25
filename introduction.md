# Introduction

## Introduction

![Doug](https://nesdoug.files.wordpress.com/2018/09/selfie4.png?w=924)

Hello all. I’m Doug \(@nesdoug2, @dougeff\). Welcome to my REDESIGNED tutorial – How to program an NES game in C. Yes. You too can make an original Nintendo game that plays on a real NES console \(or emulator\).

I’ve worked very hard to make this as easy as possible for you – and hopefully, you shouldn’t have to learn 6502 assembly. I have rewritten all the code on this website, and posted it on github.

[https://github.com/nesdoug](https://github.com/nesdoug)

Why redo everything? To better utilize the neslib code and make it easier. If you were following the old code, I made a pdf archive of the pages. I highly recommend that you don’t use those. I don’t.

[https://nesdoug.com/2018/07/11/backup-pdf-archive/](https://nesdoug.com/2018/07/11/backup-pdf-archive/)

Probably nearly all original NES games were written in Assembly. \(Almost nobody was using C in 1985\). If you prefer that route, then you could start with the **Nerdy Nights** tutorial like I did.

[http://nerdy-nights.nes.science/](http://nerdy-nights.nes.science/)

But, ASM is harder to read than C, and slower to write. You could develop a game in half the time in C.

Another option, you could use **NESmaker** \(currently $36\), which has all the code for a game written, and all you need to do is design the graphics and the levels. Apparently hundreds of people are in progress of making games with this tool.

[http://austinmckinley.com/8bit/the-tools.html](http://austinmckinley.com/8bit/the-tools.html)

Why make an NES game today? Why not? You could make a much better game in Game Maker, or Unity, or something. Much bigger, with better audio, and full color. But, I like the retro consoles. And when someone sees you pop your game into a real NES console, and it plays…they’ll be like…”Wow! You made this?”. Yes. You can. I did, and I’m an idiot.

And the NES is in that perfect middle ground for making a game. Earlier, and you have big blocks that don’t look like anything, and one blob of squares is shooting squares at another blob of squares. And you go later \(SNES/Genesis\), and you basically have full color. The best consoles for retro gaming \(my opinion\) are NES, Sega Master System \(or Game Gear\), and Gameboy Color. \[those all have z80 cpu’s and require a different compiler… and a different tutorial\].

Let’s talk about the NES.  
Released in Japan \(Famicom\), 1983, released in US, 1985. Europe, 1986. Australia, 1987.  
Re-released in 1993 as a top-loader, without the troublesome lockout chip.  
Japan had alternative disk drive systems available. FDS.  
CPU, Ricoh 2A03, 1.79 MHz, is a 6502 \(missing decimal mode\) with audio circuitry

256×240 pixels, can display 25 colors out of 53-ish available.  
64 sprites\(8×8 or 8×16\), sprites limited to 8 per horizontal line.  
60 fps \(50 fps Europe\)

Here’s the memory map for the CPU

![NES\_MAP](https://nesdoug.files.wordpress.com/2018/09/nes_map.png?w=924)

The PPU \(produces a video image\) is a separate chip, that has its own memory. It is only accessible from the CPU through a slow 1 byte at a time transfer, using hardware registers.

Here’s the memory map for the PPU

![NES\_PPU\_MAP](https://nesdoug.files.wordpress.com/2018/09/nes_ppu_map.png?w=924)

Nametable is a technical word that basically means tilemap, or background screen.

There is also another 256 bytes \(0-FF\) memory dedicated to Sprites \(OAM\). This memory is especially volatile, and needs to be rewritten every frame.

It looks like there are 4 usable nametables \(background screens\), but actually there is only enough internal VRAM for 2. The cartridges are hardwired to either be in “horizontal mirroring” or “vertical mirroring”. More advanced mappers can switch between these options.

Vertical mirroring \(or horizontal arrangement\) is for sideways scrolling games, and the lower 2 nametables are just a mirror of the upper 2 nametables, like this…

![Mario](https://nesdoug.files.wordpress.com/2015/11/mario.png?w=924)

Horizontal mirroring \(or vertical arrangement\) is for vertically scrolling games, and the right 2 nametables are just a mirror or the left 2 nametables, like this…

![KidI](https://nesdoug.files.wordpress.com/2015/11/kidi.png?w=924)

For more detailed information, check out the **nesdev wiki**.

[http://wiki.nesdev.com/w/index.php/Nesdev\_Wiki](http://wiki.nesdev.com/w/index.php/Nesdev_Wiki)

Lastly, the game cartridges usually have 2 ROM chips inside. One PRG-ROM \(executable code\), and one CHR-ROM \(graphic tiles\). But some games, instead of a CHR-ROM chip, have a CHR-RAM chip. The graphics are located somewhere in the PRG-ROM, and the program has to transfer the bytes from there to the CHR-RAM chip.

My tutorials will exclusively deal with CHR-ROM style games. It’s easier.

You might want to read up on hexadecimal numbers. 8 bit numbers are much easier to read and understand in hex. I usually use $ to indicate hex, but sometimes I use 0x. $ is used in assembly languages, and 0x is used in C like languages. You don’t need to be a math expert, but it will help if you know what I’m talking about.

