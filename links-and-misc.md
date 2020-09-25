# Links and misc

Here’s some example codes from Shiru’s website…\(the link near the top that says “these small example programs” \).

[https://shiru.untergrund.net/articles/programming\_nes\_games\_in\_c.htm](https://shiru.untergrund.net/articles/programming_nes_games_in_c.htm)

I have a copy that works with the version I’ve been using for the past 2 years, cc65 version 2.15

[http://dl.dropboxusercontent.com/s/j80n6b9gfjntfip/cc65\_examples.zip](http://dl.dropboxusercontent.com/s/j80n6b9gfjntfip/cc65_examples.zip)

You can import a MIDI file to famitracker \(an older version\). I discussed it on this post.

[https://nesdoug.com/2016/01/24/25-importing-a-midi-to-famitracker/](https://nesdoug.com/2016/01/24/25-importing-a-midi-to-famitracker/)

You can download the famitracker here \(4.2 is the last version that imported MIDI\)

[http://famitracker.com/downloads.php](http://famitracker.com/downloads.php)

.

Other projects in C.

Mojon Twins

[https://github.com/mojontwins/MK1\_NES](https://github.com/mojontwins/MK1_NES)

.

cppchriscpp

[https://github.com/cppchriscpp/nes-starter-kit](https://github.com/cppchriscpp/nes-starter-kit)

.

.

.

I wrote 2 unofficial updates to the famitone music driver. NOTE. I fixed the bugs \(I think\).

famitone3.3, which adds all notes and a volume column \(sacrifices some efficiency in size and speed\).

[https://github.com/nesdoug/famitone3.3](https://github.com/nesdoug/famitone3.3)

And famitone4.1, which adds 1xx, 2xx, and 4xx effect support.

[https://github.com/nesdoug/famitone4.1](https://github.com/nesdoug/famitone4.1)

I have used these in some of my games. Jammin Honey and Flappy Jack, for example.

I added some other features and bug fixes, which I put in the original version also…

[https://github.com/nesdoug/famitone2d](https://github.com/nesdoug/famitone2d)

Changes:  
 -added song names to file output  
 -added command line switch -allin prevents removal of unused instruments  
 -added command line switch -Wno suppresses warnings about unsupported effects

Bugfixes:  
 -multiple D00 effects \(different channels\) incorrect pattern length  
 -Bxx below D00 effect \(different channels\) incorrect pattern length  
 -Bxx loop back causing wrong instrument inserted at loop point

example of new switches:  
 text2vol filename.txt -ca65 -allin -Wno

