

# SEGV in function Catalog::~Catalog() at Catalog.cc:294

Tested in Ubuntu 16.04, 64bit.

The testcase is [segv_pdfimages](segv_pdfimages).

I use the following command:

```
./pdfimages segv_pdfimages /dev/null
```

and get:

```
#many other tips omitted
Segmentation fault
```

I use **valgrind** to analysis the bug and get the below information (absolute path information omitted):

```
valgrind /home/wws/Music/function0430/target_progs/target_xpdf/install/bin/pdfimages /home/wws/Music/function0430/copy/Fuzz/fast/fuzz_out/fuzz_out_xpdf_pdfimages/crashes/id:000013,sig:11,src:000105,op:havoc,rep:8 /dev/null
==28688== Memcheck, a memory error detector
==28688== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
==28688== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info
==28688== Command: /home/wws/Music/function0430/target_progs/target_xpdf/install/bin/pdfimages /home/wws/Music/function0430/copy/Fuzz/fast/fuzz_out/fuzz_out_xpdf_pdfimages/crashes/id:000013,sig:11,src:000105,op:havoc,rep:8 /dev/null
==28688== 
--28688-- WARNING: Serious error when reading debug info
--28688-- When reading debug info from /lib/x86_64-linux-gnu/libz.so.1.2.8:
--28688-- Ignoring non-Dwarf2/3/4 block in .debug_info
--28688-- WARNING: Serious error when reading debug info
--28688-- When reading debug info from /lib/x86_64-linux-gnu/libz.so.1.2.8:
--28688-- Last block truncated in .debug_info; ignoring
--28688-- WARNING: Serious error when reading debug info
--28688-- When reading debug info from /lib/x86_64-linux-gnu/libz.so.1.2.8:
--28688-- parse_CU_Header: is neither DWARF2 nor DWARF3 nor DWARF4
Syntax Warning: May not be a PDF file (continuing anyway)
Syntax Error: Couldn't read xref table
Syntax Warning: PDF file is damaged - attempting to reconstruct xref table...
Syntax Error (21): Dictionary key must be a name object
Syntax Error (23): Dictionary key must be a name object
Syntax Error (27): Dictionary key must be a name object
Syntax Error (29): Dictionary key must be a name object
Syntax Error (34): Dictionary key must be a name object
Syntax Error (55): Illegal character <2f> in hex string
Syntax Error (56): Illegal character <54> in hex string
Syntax Error (57): Illegal character <79> in hex string
Syntax Error (58): Illegal character <70> in hex string
Syntax Error (59): Illegal character <2f> in hex string
Syntax Error (60): Illegal character <2f> in hex string
Syntax Error (61): Illegal character <2f> in hex string
Syntax Error (62): Illegal character <2f> in hex string
Syntax Error (63): Illegal character <2f> in hex string
Syntax Error (64): Illegal character <2f> in hex string
Syntax Error (65): Illegal character <2f> in hex string
Syntax Error (66): Illegal character <2f> in hex string
Syntax Error (67): Illegal character <2f> in hex string
Syntax Error (68): Illegal character <2f> in hex string
Syntax Error (69): Illegal character <2f> in hex string
Syntax Error (70): Illegal character <2f> in hex string
Syntax Error (71): Illegal character <2f> in hex string
Syntax Error (72): Illegal character <2f> in hex string
Syntax Error (73): Illegal character <2f> in hex string
Syntax Error (74): Illegal character <2f> in hex string
Syntax Error (75): Illegal character <2f> in hex string
Syntax Error (76): Illegal character <2f> in hex string
Syntax Error (77): Illegal character <2f> in hex string
Syntax Error (78): Illegal character <2f> in hex string
Syntax Error (79): Illegal character <2f> in hex string
Syntax Error (81): Illegal character <2f> in hex string
Syntax Error (82): Illegal character <50> in hex string
Syntax Error (86): Illegal character <52> in hex string
Syntax Error (87): Illegal character <5d> in hex string
Syntax Error (88): Illegal character <2f> in hex string
Syntax Error (90): Illegal character <6f> in hex string
Syntax Error (91): Illegal character <75> in hex string
Syntax Error (92): Illegal character <6e> in hex string
Syntax Error (93): Illegal character <74> in hex string
Syntax Error (95): Illegal character <5d> in hex string
Syntax Error (97): Illegal character '>'
Syntax Error (101): Dictionary key must be a name object
Syntax Error (102): Dictionary key must be a name object
Syntax Error (103): Dictionary key must be a name object
Syntax Error (105): Dictionary key must be a name object
Syntax Error (114): Dictionary key must be a name object
Syntax Error (116): Dictionary key must be a name object
Syntax Error (119): Dictionary key must be a name object
Syntax Error (123): Dictionary key must be a name object
Syntax Error (128): Dictionary key must be a name object
Syntax Error: Catalog object is wrong type (null)
Syntax Error: Couldn't read page catalog
==28688== Conditional jump or move depends on uninitialised value(s)
==28688==    at 0x477CAE: Catalog::~Catalog() (Catalog.cc:294)
==28688==    by 0x61FCC7: PDFDoc::setup2(GString*, GString*, int) (PDFDoc.cc:312)
==28688==    by 0x61FF71: PDFDoc::setup(GString*, GString*) (PDFDoc.cc:266)
==28688==    by 0x6205FB: PDFDoc::PDFDoc(char*, GString*, GString*, PDFCore*) (PDFDoc.cc:208)
==28688==    by 0x4394B8: main (pdfimages.cc:117)
==28688== 
==28688== 
==28688== HEAP SUMMARY:
==28688==     in use at exit: 72,704 bytes in 1 blocks
==28688==   total heap usage: 5,471 allocs, 5,470 frees, 743,250 bytes allocated
==28688== 
==28688== LEAK SUMMARY:
==28688==    definitely lost: 0 bytes in 0 blocks
==28688==    indirectly lost: 0 bytes in 0 blocks
==28688==      possibly lost: 0 bytes in 0 blocks
==28688==    still reachable: 72,704 bytes in 1 blocks
==28688==         suppressed: 0 bytes in 0 blocks
==28688== Rerun with --leak-check=full to see details of leaked memory
==28688== 
==28688== For counts of detected and suppressed errors, rerun with: -v
==28688== Use --track-origins=yes to see where uninitialised values come from
==28688== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)

```

An attacker can exploit this vulnerability by submitting a malicious input that exploits this bug which will result in a Denial of Service (DoS).
