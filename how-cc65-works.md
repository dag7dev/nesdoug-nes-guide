# How cc65 works

## How cc65 works

All NES assemblers are command line programs. What does that mean? It has no graphic user interface. You don’t type code into it. You have to write your code in a separate program \(Notepad++\) and save it. And, then open a command prompt, and run the assembler by typing into the command prompt.

Cc65 is much more complicated, it requires several directives to be typed in, with multiple steps, so we’re not going to use the command prompt. But DON’T WORRY! I’m going to simplify it for you…we’re going to write a batch file \(compile.bat\) to automate the process. Once it’s written, all you should have to do is double click on compile.bat, and if all goes well, it will make your NES file. \(Write the .bat file with Notepad++.\)

How cc65 compiles..first, you will write your source code in C \(with Notepad++\). cc65 will compile it into 6502 assembly code. Then ca65 will assemble it into an object file. Then ld65 will link your object files \(using a configuration file .cfg\) into a completed .nes file. All these steps will be written in the batch file, so in the end, you will be double clicking on a .bat file to do all these steps for you. Here is an example.

```text
@echo off
set name="hello"
set path=%path%;..\bin\
set CC65_HOME=..\

cc65 -Oirs %name%.c --add-source
ca65 crt0.s
ca65 %name%.s -g
ld65 -C nrom_32k_vert.cfg -o %name%.nes crt0.o %name%.o nes.lib -Ln labels.txt

del *.o
move /Y labels.txt BUILD\ 
move /Y %name%.s BUILD\ 
move /Y %name%.nes BUILD\
pause
BUILD\%name%.nes
```

Quick explanation…you would edit the name=”” to match your .c filename. Anywhere there is a %name% it will replace it with the actual filename. Then, it runs the cc65 \(compiler\) and ca65 \(assembler\) and ld65 \(linker\) programs. Then, it deletes object files and moves the output into a BUILD folder. The last line tries to run the final .nes file.

I tried to make this as pain free as possible, so I “\#include” all the source files into one .c file and “.include” or “.incbin” all .s files in crt0.s. The only thing you need to change in the .bat file is the “name” of the main .c file. And you just need to change the names of the included files in \(name\).c and crt0.s, if they change.

More about cc65…

The 6502 processor that runs the NES is an 8 bit system. It doesn’t have an easy way to access variables larger than 8 bit, so it is preferred to use ‘unsigned char’ for most variables. Addresses are 16 bit, but nearly everything else is processed 8 bit. And, the only math it knows is adding, substraction, and bit shift multiplication/division \(ie. x 2 = &lt;&lt; 1\).

What this means, is you may want to write your C code very differently, due to the system limitations.

1. Most variables should be defined as unsigned char \(8 bit, assumed to have a value 0-255\).  
 2. Everything is global \(or static local\)  
 3. I also try to pass no values\* to functions…

The main slowdown for C code is moving things back and forth from the C stack. Local variables and passed variables use the C stack, which can be up to 5x slower than a global variable. The alternative to a function argument is to store the values to temporary global variables, just before the function call. This is the kind of thing that could easily cause conflicts, so you might want to immediately put them into static local variables \(which are also fast\) at the top of the function.

\*return values are ok, they are passed via registers

4. Arrays, ideally, should have a max of 256 bytes.  
 5. Use pointers like arrays. Instead of \*\(ptr + 1\), do ptr\[1\].  
 6. use ++g instead of g++

cc65 doesn’t optimize very well. It uses “inc g” for ++g, but always uses “lda g, clc, adc \#1, sta g” for the second \(4x longer\).  
So if you want to do this…  
 `z = g++;` you should instead do…  
  
 `z = g;`  
`++g;`

When compiling, run cc65 with the -O directive to optimize the code. There are also i,r,s directives, which are sometimes combined as -Oirs, which can add another level of optimization. However, each of these can also introduce bugs \(I haven’t encountered any bugs, so I’m still using all optimizations\).

Here’s some more suggestions for cc65 coding…

[http://www.cc65.org/doc/coding.html](http://www.cc65.org/doc/coding.html)

Why are we so concerned with optimization? Because timing is very important and resources are so slim for an old 8-bit processor. And, regular styled C code will compile into very slow code that takes up too much of the limited memory space.

cc65 can access variables and arrays from asm modules by declaring a variable “extern unsigned char foo;” \(and if it’s a zeropage symbol, add the line \#pragma zpsym \(“foo”\);. When it compiles the C code, it will add an import definition for it. If you have a large binary file…it’s easiest to “incbin” it in the asm code like this…

`'.export _foo  
 _foo:  
 .incbin “foo.bin”`  
  
Then to access it from the C code, do this `extern unsigned char foo[];`.  
Note the underscore. For some reason, when cc65 compiles, it adds an underscore before every symbol. So, on the asm side you have to add an underscore to every exported label/variable.

You can call functions in cc65 written in ASM with a \_\_fastcall\_\_. This will store rightmost passed variable in the A,X registers, rather than the C stack. This doesn’t help if the function is in C, because it immediately passes it to the C stack to use it as a local variable.

It’s also possible to inline asm code into the C code.  
It would look like this…

`asm (“Z: bit $2002”);  
asm (“bpl Z”);`

Another note. I’m using the standard nes.lib file that comes with cc65. Other people have been using runtime.lib, which doesn’t work if you get one from a different version of cc65.

And, I’m using a slightly different version of neslib that Shiru originally wrote. There are many different version floating around. Such as this fork…

[https://github.com/clbr/neslib](https://github.com/clbr/neslib)

and you could cut and paste some of the alternate functions \(both .c and .s / .sinc code\) into the version used on my code, if you want to use them.

