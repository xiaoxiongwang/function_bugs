# Bad free in binutils-2.34/bfd/coffgen.c:1782

Tested in Ubuntu 16.04, 64bit.

I use the following command:

```shell
./nm free_not_malloc_address
```

and get (absolute path information omitted):

```
nm free_not_malloc_address
00000000 I .idata$4
00000000 I .idata$5
00000000 I .idata$6
00000000 I __imp_
         U __IMPORT_DESCRIPTOR_
Aborted
```

I use **valgrind** to analysis the bug and get the below information (absolute path information omitted):

```
valgrind nm free_not_malloc_address
==11525== Memcheck, a memory error detector
==11525== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
==11525== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info
==11525== Command: nm free_not_malloc_address
==11525== 
==11525== Invalid free() / delete / delete[] / realloc()
==11525==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==11525==    by 0x6B7A80: _bfd_coff_free_symbols (coffgen.c:1782)
==11525==    by 0x6B7A80: _bfd_coff_close_and_cleanup (coffgen.c:3180)
==11525==    by 0x458ED4: bfd_close_all_done (opncls.c:789)
==11525==    by 0x458ED4: bfd_close (opncls.c:759)
==11525==    by 0x40D70F: display_file (nm.c:1392)
==11525==    by 0x405829: main (nm.c:1860)
==11525==  Address 0x5420f70 is 1,120 bytes inside a block of size 2,523 alloc'd
==11525==    at 0x4C2FB55: calloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==11525==    by 0x4518C8: bfd_malloc (libbfd.c:275)
==11525==    by 0x4518C8: bfd_zmalloc (libbfd.c:360)
==11525==    by 0x660CE1: pe_ILF_build_a_bfd (peicode.h:834)
==11525==    by 0x660CE1: pe_ILF_object_p (peicode.h:1302)
==11525==    by 0x660CE1: pe_bfd_object_p (peicode.h:1428)
==11525==    by 0x447BF7: bfd_check_format_matches (format.c:328)
==11525==    by 0x40D62F: display_file (nm.c:1375)
==11525==    by 0x405829: main (nm.c:1860)
==11525== 
==11525== Invalid free() / delete / delete[] / realloc()
==11525==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==11525==    by 0x6B7B5C: _bfd_coff_free_symbols (coffgen.c:1789)
==11525==    by 0x6B7B5C: _bfd_coff_close_and_cleanup (coffgen.c:3180)
==11525==    by 0x458ED4: bfd_close_all_done (opncls.c:789)
==11525==    by 0x458ED4: bfd_close (opncls.c:759)
==11525==    by 0x40D70F: display_file (nm.c:1392)
==11525==    by 0x405829: main (nm.c:1860)
==11525==  Address 0x5421200 is 1,776 bytes inside a block of size 2,523 alloc'd
==11525==    at 0x4C2FB55: calloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==11525==    by 0x4518C8: bfd_malloc (libbfd.c:275)
==11525==    by 0x4518C8: bfd_zmalloc (libbfd.c:360)
==11525==    by 0x660CE1: pe_ILF_build_a_bfd (peicode.h:834)
==11525==    by 0x660CE1: pe_ILF_object_p (peicode.h:1302)
==11525==    by 0x660CE1: pe_bfd_object_p (peicode.h:1428)
==11525==    by 0x447BF7: bfd_check_format_matches (format.c:328)
==11525==    by 0x40D62F: display_file (nm.c:1375)
==11525==    by 0x405829: main (nm.c:1860)
==11525== 
00000000 I .idata$4
00000000 I .idata$5
00000000 I .idata$6
00000000 I __imp_
         U __IMPORT_DESCRIPTOR_
==11525== 
==11525== HEAP SUMMARY:
==11525==     in use at exit: 6 bytes in 1 blocks
==11525==   total heap usage: 124 allocs, 125 frees, 135,906 bytes allocated
==11525== 
==11525== LEAK SUMMARY:
==11525==    definitely lost: 0 bytes in 0 blocks
==11525==    indirectly lost: 0 bytes in 0 blocks
==11525==      possibly lost: 0 bytes in 0 blocks
==11525==    still reachable: 6 bytes in 1 blocks
==11525==         suppressed: 0 bytes in 0 blocks
==11525== Rerun with --leak-check=full to see details of leaked memory
==11525== 
==11525== For counts of detected and suppressed errors, rerun with: -v
==11525== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

I use ***\*AddressSanitizer\**** to build binutils and running nm with the following command:

```
./nm free_not_malloc_address
```

This is the ASAN information (absolute path information omitted):

```
nm free_not_malloc_address
=================================================================
==11561==ERROR: AddressSanitizer: attempting free on address which was not malloc()-ed: 0x61e00000f4e0 in thread T0
    #0 0x7f1ff1baa32a in __interceptor_free (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x9832a)
    #1 0x55b934 in _bfd_coff_free_symbols binutils-2.34/bfd/coffgen.c:1782
    #2 0x562664 in _bfd_coff_close_and_cleanup binutils-2.34/bfd/coffgen.c:3180
    #3 0x428445 in bfd_close_all_done binutils-2.34/bfd/opncls.c:789
    #4 0x40952b in display_file binutils-2.34/binutils/nm.c:1392
    #5 0x405250 in main binutils-2.34/binutils/nm.c:1860
    #6 0x7f1ff156482f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #7 0x405cf8 in _start (nm+0x405cf8)

0x61e00000f4e0 is located 1120 bytes inside of 2523-byte region [0x61e00000f080,0x61e00000fa5b)
allocated by thread T0 here:
    #0 0x7f1ff1baa662 in malloc (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x98662)
    #1 0x424733 in bfd_malloc binutils-2.34/bfd/libbfd.c:275
    #2 0x4248e0 in bfd_zmalloc binutils-2.34/bfd/libbfd.c:360
    #3 0x52f0bc in pe_ILF_build_a_bfd binutils-2.34/bfd/peicode.h:834
    #4 0x52f0bc in pe_ILF_object_p binutils-2.34/bfd/peicode.h:1302
    #5 0x52f0bc in pe_bfd_object_p binutils-2.34/bfd/peicode.h:1428
    #6 0x421b9b in bfd_check_format_matches binutils-2.34/bfd/format.c:328
    #7 0x4094e0 in display_file binutils-2.34/binutils/nm.c:1375
    #8 0x405250 in main binutils-2.34/binutils/nm.c:1860
    #9 0x7f1ff156482f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)

SUMMARY: AddressSanitizer: bad-free ??:0
```

