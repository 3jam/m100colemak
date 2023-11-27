Here are a few TRS-80 Model 100 ROM patches that swap its key map to Colemak.
The different patches are described below. I recommend using the
`colemak-caps` patch. All patches also include the `Y2K patch` found
[here](http://bitchin100.com/wiki/images/d/d1/R_M100.BR).  

These patches are based on the ROM in my system which had the marking
`LH535618`. There are a number of ROMs that were shipped with the M100, so
please confirm you have the correct ROM with the checksum below.  

More information about Model 100 ROMs can be found on the
[Bitchin100 wiki](https://bitchin100.com/wiki/index.php?title=Model_and_ROM_information).

To replace the ROM, I used Brian K. White's excellent
[FlexROM 100](https://github.com/bkw777/aDIPters) adapter.

Applying Patches
================

The bsdiff patches can be applied by running:
```
bspatch <original rom> <new rom> patch-name.bsdiff
```

The IPS patches can by applied using any number of IPS patch tools.
ROMhacking.net has a good online tool [here](https://www.romhacking.net/patch/).

MD5 Checksums
-------------
Here are checksums of the original ROM as well as the expected output of the
patched ROMs.
```
187d1495102acb7cde45eed10e52b9cf    Original M100 ROM (LH535618)
68a1d8ab45f07350b6a8dbe087bdc83f    Patched with colemak-o
42cf3f25da642893ae22396722ca4341    Patched with colemak
0f2b422678239d53ec583f667a62815e    Patched with colemak-caps
```

All patches change the following bytes:
=======================================

This is the absolute minimum patch to get Colemak working on the M100.  

```
00007BF0   AA 7A 78 63  76 62 6B 6D  69 61 72 73  74 64 68 6E  .zxcvbkmiarstdhn
00007C00   65 71 77 66  70 67 6A 6C  75 79 3B 5B  6F 27 2C 2E  eqwfpgjluy;[o',.
00007C10   2F 31 32 33  34 35 36 37  38 39 30 2D  3D 5A 58 43  /1234567890-=ZXC
00007C20   56 42 4B 4D  49 41 52 53  54 44 48 4E  45 51 57 46  VBKMIARSTDHNEQWF
00007C30   50 47 4A 4C  55 59 3A 5D  4F 22 3C 3E  3F 21 40 23  PGJLUY:]O"<>?!@#
...
00007CF0   00 00 00 D4  D2 D3 A6 A7  A8 6D 30 6E  31 65 32 69  .........m0n1e2i
00007D00   33 6C 34 75  35 79 36 01  06 14 02 20  7F 09 1B 8B  3l4u5y6.... ....
```

Caps Lock solutions
===================
Due to the way that the original Caps Lock routine works by only capitalizing
characters that have an index less than `1AH`, `o` is not capitalized, as we've
moved it's location in the array to `1BH`. There are a few solutions to this:  

colemak-o
---------
One is to accept that `;` and `[` will be shifted to `:` and `]` respectively,
and also include the `o` as well. We do this by increasing the number of valid
characters by 2. To do this, the byte at `722EH` is changed to `1CH`. This is a
very unintrusive way to get Caps Lock working on all letters without large
modifications to the ROM. The patch using this method is `colemak-o`. 

colemak
-------
Another solution is to accept that `o` will not be capitalized by the Caps Lock
key and reduce the number of valid characters by one to avoid shifting `;`. To
do this, the byte at `722EH` is changed to `19H`. The patch using this method is
`colemak`.  

colemak-caps
------------
A final, more extreme solution is to use Stephen Adolph's
[patch](http://bitchin100.com/wiki/index.php?title=M100/T102_Hardware_scrolling_patch)
that reduces the PIO data table from 240 to 80 bytes. This gives us room in the
ROM to write a slightly more complex Caps Lock routine. This patch leaves 138
free bytes for future patches from `75B6H` to `7640H`. I have been using this
patch daily for several months and haven't had any issues from the PIO table
size reduction. The patch using this method is `colemak-caps`.  

### The new Caps Lock routine:
```
        .org    0722CH  ; Location of original Caps Lock routine
        MOV     A,C     ; unchanged
        JMP     patch   ;                                       [C3 AB 75]
back:   MVI     E,2CH   ; unchanged
        RET             ; unchanged

        .org 075ABH	; Start of space made available by Stephen's patch
patch:  CPI     1BH     ; Check if we have an `o'               [FE 1B]
        JZ      back    ; Return to old Caps Lock routine       [CA 30 72]
        CPI     19H     ; Check if we have less than `;'        [FE 19]
        RNC             ; If A >= 19H do not capitalize         [D0]
        JMP     back    ; Return to old Caps Lock routine       [C3 30 75]
```
