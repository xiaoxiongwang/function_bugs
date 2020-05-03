# Bad free in binutils-2.34/bfd/coffgen.c:1782

Tested in Ubuntu 16.04, 64bit.

I use the following command:

```shell
./objdump -d bad_free
```

and get (absolute path information omitted):

```
bad_free:     file format pei-i386


Disassembly of section .text:

00000000 <��>:
   0:	ff 25 00 00 00 00    	jmp    *0x0
   6:	90                   	nop
   7:	90                   	nop
*** Error in `./objdump': free(): invalid pointer: 0x000000000104ab90 ***
======= Backtrace: =========
/lib/x86_64-linux-gnu/libc.so.6(+0x777e5)[0x7fc81e8287e5]
/lib/x86_64-linux-gnu/libc.so.6(+0x8037a)[0x7fc81e83137a]
/lib/x86_64-linux-gnu/libc.so.6(cfree+0x4c)[0x7fc81e83553c]
./objdump[0x87d831]
./objdump[0x61e7b3]
./objdump[0x40b569]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf0)[0x7fc81e7d1830]
./objdump[0x40c2a9]
======= Memory map: ========
00400000-00b23000 r-xp 00000000 103:01 21657469                          ./objdump
00d22000-00d23000 r--p 00722000 103:01 21657469                          ./objdump
00d23000-00d2b000 rw-p 00723000 103:01 21657469                          ./objdump
00d2b000-00d34000 rw-p 00000000 00:00 0 
01046000-01067000 rw-p 00000000 00:00 0                                  [heap]
7fc818000000-7fc818021000 rw-p 00000000 00:00 0 
7fc818021000-7fc81c000000 ---p 00000000 00:00 0 
7fc81e10e000-7fc81e125000 r-xp 00000000 103:01 10504621                  /lib/x86_64-linux-gnu/libgcc_s.so.1
7fc81e125000-7fc81e324000 ---p 00017000 103:01 10504621                  /lib/x86_64-linux-gnu/libgcc_s.so.1
7fc81e324000-7fc81e325000 r--p 00016000 103:01 10504621                  /lib/x86_64-linux-gnu/libgcc_s.so.1
7fc81e325000-7fc81e326000 rw-p 00017000 103:01 10504621                  /lib/x86_64-linux-gnu/libgcc_s.so.1
7fc81e326000-7fc81e7b1000 r--p 00000000 103:01 14811834                  /usr/lib/locale/locale-archive
7fc81e7b1000-7fc81e971000 r-xp 00000000 103:01 10487655                  /lib/x86_64-linux-gnu/libc-2.23.so
7fc81e971000-7fc81eb71000 ---p 001c0000 103:01 10487655                  /lib/x86_64-linux-gnu/libc-2.23.so
7fc81eb71000-7fc81eb75000 r--p 001c0000 103:01 10487655                  /lib/x86_64-linux-gnu/libc-2.23.so
7fc81eb75000-7fc81eb77000 rw-p 001c4000 103:01 10487655                  /lib/x86_64-linux-gnu/libc-2.23.so
7fc81eb77000-7fc81eb7b000 rw-p 00000000 00:00 0 
7fc81eb7b000-7fc81eb7e000 r-xp 00000000 103:01 10485854                  /lib/x86_64-linux-gnu/libdl-2.23.so
7fc81eb7e000-7fc81ed7d000 ---p 00003000 103:01 10485854                  /lib/x86_64-linux-gnu/libdl-2.23.so
7fc81ed7d000-7fc81ed7e000 r--p 00002000 103:01 10485854                  /lib/x86_64-linux-gnu/libdl-2.23.so
7fc81ed7e000-7fc81ed7f000 rw-p 00003000 103:01 10485854                  /lib/x86_64-linux-gnu/libdl-2.23.so
7fc81ed7f000-7fc81eda5000 r-xp 00000000 103:01 10485855                  /lib/x86_64-linux-gnu/ld-2.23.so
7fc81ef71000-7fc81ef75000 rw-p 00000000 00:00 0 
7fc81ef9c000-7fc81ef9d000 rw-p 00000000 00:00 0 
7fc81ef9d000-7fc81efa4000 r--s 00000000 103:01 15079042                  /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
7fc81efa4000-7fc81efa5000 r--p 00025000 103:01 10485855                  /lib/x86_64-linux-gnu/ld-2.23.so
7fc81efa5000-7fc81efa6000 rw-p 00026000 103:01 10485855                  /lib/x86_64-linux-gnu/ld-2.23.so
7fc81efa6000-7fc81efa7000 rw-p 00000000 00:00 0 
7ffdb18b5000-7ffdb18d7000 rw-p 00000000 00:00 0                          [stack]
7ffdb1988000-7ffdb198b000 r--p 00000000 00:00 0                          [vvar]
7ffdb198b000-7ffdb198d000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
Aborted (core dumped)

```

I use **AddressSanitizer** to build binutils2.34 and running objdump with the following command:

```shell
./objdump -d bad_free
```

This is the ASAN information (absolute path information omitted):

```
bad_free:     file format pei-i386


crashes/crashes1/id:000000,sig:06,src:002244,op:flip1,pos:12:     file format pei-i386


Disassembly of section .text:

00000000 <��>:
   0:	ff 25 00 00 00 00    	jmp    *0x0
   6:	90                   	nop
   7:	90                   	nop
=================================================================
==17144==ERROR: AddressSanitizer: attempting free on address which was not malloc()-ed: 0x61e00000f4e0 in thread T0
    #0 0x7fa6f620432a in __interceptor_free (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x9832a)
    #1 0x69c8be in _bfd_coff_free_symbols binutils-2.34/bfd/coffgen.c:1782
    #2 0x6a437d in _bfd_coff_close_and_cleanup binutils-2.34/bfd/coffgen.c:3180
    #3 0x4f8498 in bfd_close_all_done binutils-2.34/bfd/opncls.c:789
    #4 0x4186f6 in display_file objdump.c:5016
    #5 0x4199a4 in main objdump.c:5349
    #6 0x7fa6f5bbe82f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #7 0x403418 in _start (objdump+0x403418)

0x61e00000f4e0 is located 1120 bytes inside of 2511-byte region [0x61e00000f080,0x61e00000fa4f)
allocated by thread T0 here:
    #0 0x7fa6f6204662 in malloc (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x98662)
    #1 0x4f3bec in bfd_malloc binutils-2.34/bfd/libbfd.c:275
    #2 0x4f3dcd in bfd_zmalloc binutils-2.34/bfd/libbfd.c:360
    #3 0x6520e2 in pe_ILF_build_a_bfd binutils-2.34/bfd/peicode.h:834
    #4 0x653c32 in pe_ILF_object_p binutils-2.34/bfd/peicode.h:1302
    #5 0x6546d0 in pe_bfd_object_p binutils-2.34/bfd/peicode.h:1428
    #6 0x4f02bd in bfd_check_format_matches binutils-2.34/bfd/format.c:328
    #7 0x418349 in display_object_bfd objdump.c:4890
    #8 0x418661 in display_any_bfd objdump.c:4982
    #9 0x4186d6 in display_file objdump.c:5003
    #10 0x4199a4 in main objdump.c:5349
    #11 0x7fa6f5bbe82f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)

SUMMARY: AddressSanitizer: bad-free ??:0 __interceptor_free
==17144==ABORTING
```

I use **valgrind** to analysis the bug and get the below information (absolute path information omitted):

```
==3062== Memcheck, a memory error detector
==3062== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
==3062== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info
==3062== Command: ./objdump -d bad_free
==3062== 

bad_free:     file format pei-i386


Disassembly of section .text:

00000000 <��>:
   0:	ff 25 00 00 00 00    	jmp    *0x0
   6:	90                   	nop
   7:	90                   	nop
==3062== Invalid free() / delete / delete[] / realloc()
==3062==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==3062==    by 0x87D830: _bfd_coff_free_symbols (coffgen.c:1782)
==3062==    by 0x87D830: _bfd_coff_close_and_cleanup (coffgen.c:3180)
==3062==    by 0x61E7B2: bfd_close_all_done (opncls.c:789)
==3062==    by 0x40B568: display_file (objdump.c:5016)
==3062==    by 0x40B568: main (objdump.c:5349)
==3062==  Address 0x541f340 is 1,120 bytes inside a block of size 2,511 alloc'd
==3062==    at 0x4C2FB55: calloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==3062==    by 0x616258: bfd_malloc (libbfd.c:275)
==3062==    by 0x616258: bfd_zmalloc (libbfd.c:360)
==3062==    by 0x826A91: pe_ILF_build_a_bfd (peicode.h:834)
==3062==    by 0x826A91: pe_ILF_object_p (peicode.h:1302)
==3062==    by 0x826A91: pe_bfd_object_p (peicode.h:1428)
==3062==    by 0x60C587: bfd_check_format_matches (format.c:328)
==3062==    by 0x42247F: display_object_bfd (objdump.c:4890)
==3062==    by 0x42247F: display_any_bfd (objdump.c:4982)
==3062==    by 0x40B51D: display_file (objdump.c:5003)
==3062==    by 0x40B51D: main (objdump.c:5349)
==3062== 
==3062== Invalid free() / delete / delete[] / realloc()
==3062==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==3062==    by 0x87D90C: _bfd_coff_free_symbols (coffgen.c:1789)
==3062==    by 0x87D90C: _bfd_coff_close_and_cleanup (coffgen.c:3180)
==3062==    by 0x61E7B2: bfd_close_all_done (opncls.c:789)
==3062==    by 0x40B568: display_file (objdump.c:5016)
==3062==    by 0x40B568: main (objdump.c:5349)
==3062==  Address 0x541f5d0 is 1,776 bytes inside a block of size 2,511 alloc'd
==3062==    at 0x4C2FB55: calloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==3062==    by 0x616258: bfd_malloc (libbfd.c:275)
==3062==    by 0x616258: bfd_zmalloc (libbfd.c:360)
==3062==    by 0x826A91: pe_ILF_build_a_bfd (peicode.h:834)
==3062==    by 0x826A91: pe_ILF_object_p (peicode.h:1302)
==3062==    by 0x826A91: pe_bfd_object_p (peicode.h:1428)
==3062==    by 0x60C587: bfd_check_format_matches (format.c:328)
==3062==    by 0x42247F: display_object_bfd (objdump.c:4890)
==3062==    by 0x42247F: display_any_bfd (objdump.c:4982)
==3062==    by 0x40B51D: display_file (objdump.c:5003)
==3062==    by 0x40B51D: main (objdump.c:5349)
==3062== 
==3062== 
==3062== HEAP SUMMARY:
==3062==     in use at exit: 0 bytes in 0 blocks
==3062==   total heap usage: 70 allocs, 72 frees, 141,767 bytes allocated
==3062== 
==3062== All heap blocks were freed -- no leaks are possible
==3062== 
==3062== For counts of detected and suppressed errors, rerun with: -v
==3062== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

From the above analysis, we can see that the memory is allocated in function `bfd_malloc` of `bfd/libbfd.c` and there is a bad free in function `_bfd_coff_free_symbols` of `bfd/coffgen.c` which causes a crash.
