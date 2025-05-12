# CD-i System Libraries Technical Reference Manual

> This document is a manual conversion from the cdisys.pdf, which was accessed from ICDIA. Notes by the creator of this document are listed in quote sections like this one.

## Overview

### Introduction

C language bindings for the UCM File Manager functions, CDFM File Manager functions and the Hardware Configuration Status Descriptor (CSD) functions are provided for C programmers through the C library `cdi.l`. These bindings give direct access to the low-level assembly language functions defined in the Green Book.

C language bindings for access to the OS-9 system calls `F$Alarm`, `F$Load`, `F$DatMod` and `F$SrqCMem` are provided in the C library `os9.l`.

A C program accessing these bindings must link with appropriate library. For example:

`cc prog.r -l=/dd/lib/cdi.l -l=/dd/lib/os9.l -f=prog`

**NOTE:** These two libraries are not dependent upon each other in any way.

### CDI.L: The CD-I C Bindings Library

The CD-I library `cdi.l`, consists of C bindings for both the UCM and CDFM File Managers. In all of these functions, a `path` parameter is used. This is a pathlist obtained from the standard `open()` call. For UCM functions, this path should be opened to the video device at the start of the program and used for all calls to the UCM functions. For the CDFM `getStat` and `setStat` functions, this is a pathlist to a CD-I file. For the CDFM "audio" functions, this is a pathlist to the audio device.

`cdi.l` also contains the C bindings to access the Configuration Status Descriptor (CSD). In order for an application program to be able to determine the name of each device and the functional configuration of a CD-I system, the CSD must be accessed. The CSD contains one for each device on the system. Each CSD entry is known as a Device Status Descriptor (DSD).

Each DSD is made up of 3 parts: a name, a type and a list of parameters which describe the functional capabilities of the device. For a full description of the device types and their associated parameter lists, refer to Appendix VII.2 of the Green Book.

Finally, `cdi.l` contains the low-level special effects functions. These low level special effects routines return actual FCT and LCT instructions which can be written directly into the control tables to affect the display. The low level routines place an extra layer of software between the application and the actual manipulation of the LCTs. In this way, the content provider does not directly need to know the format of all LCT instructions.

In most cases, a function which does not have a specific return value will return `0` if no error occurs and `-1` when an error does occur. If an error occurs, the global variable `errno` will be set. Functions which return pointers will return NULL on error.

#### UCM Functions

**Character Output Functions** are used in conjunction with `I$Write` and `I$Writeln` (`write()` and `writeln()`) calls to output text characters instead of the `dr_text()` and `dr_jtxt()` function calls. There are two classes of these functions: those to set up the character output drawmap and fonts and those to emulate terminal control sequences. The CRT emulation functions are provided for convenience. They only work, however,
for 8 bit and 7/15 bit character modes. The character output functions are:

- [`co_afnt()`](#co_afnt-activate-font) Activate font
- [`co_cod()`](#co_cod-set-character-output-drawmap) Set character output drawmap
- [`co_dfnt()`](#co_dfnt-deactivate-font) Deactivate font
- [`co_scmm()`](#co_scmm-set-character-code-mapping-method) Set character code mapping method
- [`crt_cdwn()`](#crt_cdwn-cursor-down) Cursor down
- [`crt_ceol()`](#crt_ceol-clear-to-end-of-line) Clear to end-of-line
- [`crt_ceos()`](#crt_ceos-clear-to-end-of-screen) Clear to end-of-screen
- [`crt_clft()`](#crt_clft-cursor-left) Cursor left
- [`crt_cr()`](#crt_cr-carriage-return) Carriage return
- [`crt_crgt()`](#crt_crgt-cursor-right) Cursor right
- [`crt_cscrn()`](#crt_cscrn-clear-screen) Clear screen
- [`crt_cup()`](#crt_cup-cursor-up) Cursor up
- [`crt_curxy()`](#crt_curxy-cursor-address) Cursor address
- [`crt_dchar()`](#crt_dchar-delete-character) Delete character
- [`crt_dlin()`](#crt_dlin-delete-line) Delete line
- [`crt_hidcur()`](#crt_hidcur-hide-cursor) Hide cursor
- [`crt_home()`](#crt_home-cursor-home) Cursor home
- [`crt_ichar()`](#crt_ichar-insert-character) Insert character
- [`crt_ilin()`](#crt_ilin-insert-line) Insert line
- [`crt_revoff()`](#crt_revoff-end-reverse-video) End reverse video
- [`crt_revon()`](#crt_revon-start-reverse-video) Start reverse video
- [`crt_shwcur()`](#crt_shwcur-show-cursor) Show cursor
- [`crt_ulon()`](#crt_ulon-start-underlining) Start underlining
- [`crt_uloff()`](#crt_uloff-end-underlining) End underlining
- [`crt_wrapoff()`](#crt_wrapoff-turn-auto-wrap-mode-off) Turn auto-wrap mode off
- [`crt_wrapon()`](#crt_wrapon-turn-auto-wrap-mode-on) Turn auto-wrap mode on

**Display Control Program Functions**: Applications control the Video Display through the use of Field Control Tables and Line Control Tables. These tables together make up the Display Control Program. The following functions allow the application to manipulate this program:

- [`dc_crfct()`](#dc_crfct-create-field-control-table) Create Field Control Table
- [`dc_crlct()`](#dc_crlct-create-line-control-table) Create Line Control Table
- [`dc_dlfct()`](#dc_dlfct-delete-field-control-table) Delete Field Control Table
- [`dc_dllct()`](#dc_dllct-delete-line-control-table) Delete Line Control Table
- [`dc_exec()`](#dc_exec-execute-display-control-program) Execute display control program
- [`dc_flnk()`](#dc_flnk-link-fct-to-lct) Link Field Control Table to Field Control Table
- [`dc_intl()`](#dc_intl-set-display-interlace-mode) Set display interlace mode
- [`dc_llnk()`](#dc_llnk-link-lct-to-lct) Link Line Control Table to Line Control Table
- [`dc_nop()`](#dc_nop-write-array-of-nop-instructions-to-lct) Write array of no operation instructions to LCT
- [`dc_prdlct()`](#dc_prdlct-physical-read-of-lct-columns) Physical read of Line Control Table columns
- [`dc_pwrlct()`](#dc_pwrlct-physical-write-to-lct-columns) Physical write to Line Control Table columns
- [`dc_rdfct()`](#dc_rdfct-read-field-control-table) Read Field Control Table
- [`dc_rdfi()`](#dc_rdfi-read-field-control-table-instruction) Read Field Control Table instruction
- [`dc_rdlct()`](#dc_rdlct-read-line-control-table-columns) Read Line Control Table columns
- [`dc_rdli()`](#dc_rdli-read-line-control-table-instruction) Read Line Control Table instruction
- [`dc_relea()`](#dc_relea-release-pending-signal-request) Release pending signal request
- [`dc_setcmp()`](#dc_setcmp-set-compatibility-mode) Set compatibility mode
- [`dc_ssig()`](#dc_ssig-send-signal-on-video-interrupt) Send signal on video interrupt
- [`dc_wrfct()`](#dc_wrfct-write-field-control-table) Write Field Control Table
- [`dc_wrfi()`](#dc_wrfi-write-field-control-table-instruction) Write Field Control Table instruction
- [`dc_wrlct()`](#dc_wrlct-write-line-control-table) Write Line Control Table
- [`dc_wrli()`](#dc_wrli-write-line-control-table-instruction) Write Line Control Table instruction

**Display Control Program Macros**: The macros provided here make it easier for an application to generate FCT and LCT instructions. The application will have to directly write these instructions into the FCT/LCT using the display control program functions. These macros are also used to generate the parameters for many of the high-level functions found in CD-I/RAVE. The macros consist of:

- [`cp_bkcol()`](#cp_bkcol-load-backdrop-color) Load backdrop color
- [`cp_cbnk()`](#cp_cbnk-set-clut-bank) Set CLUT bank
- [`cp_clut()`](#cp_clut-load-clut-color) Load CLUT color
- [`cp_crmatte()`](#cp_crmatte-create-an-array-of-matte-instructions-from-a-region) Create an array of matte instructions from a region
- [`cp_dadr()`](#cp_dadr-load-display-line-start-pointer) Load display line start pointer
- [`cp_dadr_col()`](#cp_dadr_col-create-an-array-of-display-start-address-instructions) Create an array of display start address instructions
- [`cp_dprm()`](#cp_dprm-load-display-parameters) Load display parameters
- [`cp_icf()`](#cp_icf-load-image-contribution-factor) Load image contribution factor
- [`cp_icm()`](#cp_icm-select-image-coding-method) Select image coding method
- [`cp_matte()`](#cp_matte-load-matte-register-0-7) Load matte register 0-7
- [`cp_mcol()`](#cp_mcol-load-mask-color) Load mask color
- [`cp_nop()`](#cp_nop-no-operation) No operation
- [`cp_offset_matte()`](#cp_offset_matte-add-x-offset-to-an-array-of-matte-instructions) Add X offset to an array of matte instructions
- [`cp_phld()`](#cp_phld-load-mosaic-pixel-hold-factor) Load mosaic pixel hold factor
- [`cp_po()`](#cp_po-load-plane-order) Load plane order
- [`cp_sig()`](#cp_sig-signal-when-scan-reaches-this-line) Signal when scan reaches this line
- [`cp_tci()`](#cp_tci-load-transparency-control-information) Load transparency control information
- [`cp_tcol()`](#cp_tcol-load-transparent-color) Load transparent color
- [`cp_yuv()`](#cp_yuv-load-dyuv-start-value) Load DYUV start value

**Drawing Parameter Functions** are used to change the parameters used by all drawing functions which include the drawing pattern, color, pen size, pen style and fonts used for graphics text drawing:

- [`dp_afnt()`](#dp_afnt-activate-font) Activate Font
- [`dp_clip()`](#dp_clip-set-clipping-region) Set clipping region
- [`dp_dfnt()`](#dp_dfnt-deactivate-font) Deactivate Font
- [`dp_gfnt()`](#dp_gfnt-get-font) Get Font
- [`dp_paln()`](#dp_paln-set-pattern-alignment) Set pattern alignment
- [`dp_pnsz()`](#dp_pnsz-set-pen-size) Set Pen size
- [`dp_pstyl()`](#dp_pstyl-set-pen-style) Set Pen Style
- [`dp_ptn()`](#dp_ptn-set-drawing-pattern) Set drawing pattern
- [`dp_rfnt()`](#dp_rfnt-release-font) Release Font
- [`dp_scmm()`](#dp_scmm-set-character-mapping-method) Set Character mapping method
- [`dp_scr()`](#dp_scr-set-color-register) Set color register
- [`dp_tcol()`](#dp_tcol-set-transparent-color) Set transparent color

**Drawmap Control Functions** are used to allocate and deallocate drawmaps and to exchange or copy data between them:

- [`dm_close()`](#dm_close-close-drawmap) Close drawmap
- [`dm_cncl()`](#dm_cncl-conceal-error-in-drawmap) Conceal error in drawmap
- [`dm_copy()`](#dm_copy-copy-drawmap-to-drawmap) Copy drawmap to drawmap
- [`dm_creat()`](#dm_creat-create-drawmap) Create drawmap
- [`dm_create()`](#dm_create-create-drawmap) Create drawmap
- [`dm_dup()`](#dm_dup-duplicate-drawmap) Duplicate drawmap
- [`dm_dupe()`](#dm_dupe-duplicate-drawmap) Duplicate drawmap
- [`dm_exch()`](#dm_exch-exchange-data-between-drawmaps) Exchange data between drawmaps
- [`dm_irwr()`](#dm_irwr-irregular-write-to-drawmap) Irregular Write to drawmap
- [`dm_org()`](#dm_org-set-drawmap-origin) Set drawmap origin
- [`dm_rdpix()`](#dm_rdpix-read-pixel) Read single pixel from drawmap
- [`dm_read()`](#dm_read-read-from-drawmap) Read from drawmap
- [`dm_tcopy()`](#dm_tcopy-copy-drawmap-to-drawmap-with-transparency) Copy drawmap to drawmap with transparency
- [`dm_texch()`](#dm_texch-exchange-between-drawmaps-with-transparency) Exchange data between drawmaps with transparency
- [`dm_write()`](#dm_write-write-to-drawmap) Write to drawmap
- [`dm_wrpix()`](#dm_wrpix-write-pixel) Write single pixel to drawmap

**Graphics Drawing Functions** execute the graphics primitives such as line, rectangle, circle and ellipse. The functions provided are:

- [`dr_bfil()`](#dr_bfil-fill-a-bounded-area) Fill a bounded area
- [`dr_carc()`](#dr_carc-draw-an-arc) Draw an arc
- [`dr_circ()`](#dr_circ-draw-a-circle) Draw a circle
- [`dr_copy()`](#dr_copy-copy-data-from-drawmap-to-drawmap) Copy data from drawmap to drawmap
- [`dr_cwdg()`](#dr_cwdg-draw-a-circular-wedge) Draw a circular wedge
- [`dr_dot()`](#dr_dot-draw-a-dot) Draw a dot
- [`dr_drgn()`](#dr_drgn-draw-a-region) Draw a region
- [`dr_earc()`](#dr_earc-draw-an-elliptical-arc) Draw an elliptical arc
- [`dr_elps()`](#dr_elps-draw-an-ellipse) Draw an ellipse
- [`dr_erect()`](#dr_erect-draw-an-elliptical-cornered-rectangle) Draw an elliptical cornered rectangle
- [`dr_ewdg()`](#dr_ewdg-draw-an-elliptical-wedge) Draw an elliptical wedge
- [`dr_ffil()`](#dr_ffil-flood-fill) Flood fill
- [`dr_jtxt()`](#dr_jtxt-output-justified-text) Output justified text
- [`dr_line()`](#dr_line-draw-a-line) Draw a line
- [`dr_pgon()`](#dr_pgon-draw-a-polygon) Draw a polygon
- [`dr_plin()`](#dr_plin-draw-a-polyline) Draw a polyline
- [`dr_rect()`](#dr_rect-draw-a-rectangle) Draw a rectangle
- [`dr_text()`](#dr_text-output-graphics-text) Output graphics text

Each of the drawing functions have an operation code parameter (`opcode`) which specifies whether clipping will be performed, how shapes will be filled and how pixels will be combined.

Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and below:

```plaintext
Bits 0-4  combinatorial writing mode:
          OP_ZERO     0x0000      Write Zeros
          OP_SAD      0x0001      Source And Destination
          OP_SANO     0x0002      Source And Not Destination
          OP_RPLC     0x0003      Replace
          OP_NSAO     0x0004      Not Source And Destination
          OP_NMOD     0x0005      Do Not Modify
          OP_SXO      0x0006      Source Xor Destination
          OP_SOD      0x0007      Source Or Destination
          OP_N_SOD    0x0008      Not (Source Or Destination)
          OP_N_SXD    0x0009      Not (Source Xor Destination)
          OP_ND       0x000A      Not Destination
          OP_SOND     0x000B      Source Or Not Destination
          OP_NS       0x000C      Not Source
          OP_NSOD     0x000D      Not Source Or Destination
          OP_N_SAD    0x000E      Not (Source And Destination)
          OP_ONES     0x000F      Write Ones
          OP_SPD      0x0010      Source Plus Destination
          OP_SMD      0x0011      Source Minus Destination
          OP_DMS      0x0012      Destination Minus Source

Bits 5-8  color register number for solids:
          OP_CR0      0x0000      Color Register Zero
          OP_CR1      0x0020      Color Register One
          OP_CR2      0x0040      Color Register Two
          OP_CR3      0x0060      Color Register Three
          OP_CR4      0x0080      Color Register Four
          OP_CR5      0x00A0      Color Register Five
          OP_CR6      0x00C0      Color Register Six
          OP_CR7      0x00E0      Color Register Seven
          OP_CR8      0x0100      Color Register Eight
          OP_CR9      0x0120      Color Register Nine
          OP_CR10     0x0140      Color Register Ten
          OP_CR11     0x0160      Color Register Eleven
          OP_CR12     0x0180      Color Register Twelve
          OP_CR13     0x01A0      Color Register Thirteen
          OP_CR14     0x01C0      Color Register Fourteen
          OP_CR15     0x01E0      Color Register Fifteen

Bit 9     transparency writing mode:
          OP_TMVIS    0x0000      Visible. i.e. Not Transparent
          OP_TMTRA    0x0200      Transparent

Bit 10    reserved

Bit 11    dash transparency:
          OP_DTVIS    0x0000      Visible. i.e. Not Transparent
          OP_DTTRA    0x0800      Transparent

Bit 12    dash styling:
          OP_OSSLD    0x0000      Solid Style
          OP_OSOSH    0x1000      Dashed Style

Bit 13    pattern mode:
          OP_PMSLD    0x0000      Solid Color
          OP_PMPAT    0x2000      Pattern On

Bit 14    fill mode:
          OP_FMOUT    0x0000      Outlined
          OP_FMFIL    0x4000      Filled

Bit 15    clipping flag:
          OP_CLPOFF   0x0000      Clipping off
          OP_CLPON    0x8000      Clipping on
```

**Graphic Cursor Functions** are provided to manipulate the position, pattern and color of the graphics cursor:

- [`gc_blnk()`](#gc_blnk-set-graphics-cursor-blink-rate) Set Graphics Cursor blink rate
- [`gc_col()`](#gc_col-set-graphics-cursor-color) Set Graphics Cursor color
- [`gc_hide()`](#gc_hide-hide-graphics-cursor) Hide Graphics Cursor
- [`gc_org()`](#gc_org-set-graphics-cursor-origin) Set Graphics Cursor origin
- [`gc_pos()`](#gc_pos-position-graphics-cursor) Position Graphics Cursor
- [`gc_ptn()`](#gc_ptn-set-graphics-cursor-pattern) Set Graphics Cursor pattern
- [`gc_show()`](#gc_show-show-graphics-cursor) Show Graphics Cursor

**Keyboard Functions** are used for keyboard status control and for getting information about the keyboard:

- [`kb_rdy()`](#kb_rdy-check-for-data-ready) Check for data ready
- [`kb_read()`](#kb_read-read-keyboard-event) Read keyboard event
- [`kb_rel()`](#kb_rel-release-keyboard-device) Release keyboard device
- [`kb_repeat()`](#kb_repeat-set-keyboard-latency-and-repeat-times) Set keyboard Latency and Repeat times
- [`kb_ssig()`](#kb_ssig-send-signal-on-data-ready) Send signal on data ready
- [`kb_stat()`](#kb_stat-get-keyboard-status) Get keyboard status

**Pointer Input Functions** are used to access pointing devices such as mice, joysticks or graphics tablets:

- [`pt_coord()`](#pt_coord-get-pointer-coordinates) Get pointer coordinates
- [`pt_org()`](#pt_org-set-pointer-origin) Set pointer origin
- [`pt_pos()`](#pt_pos-position-the-pointer) Position the pointer
- [`pt_rel()`](#pt_rel-release-pointer-device) Release pointer device
- [`pt_ssig()`](#pt_ssig-send-signal-on-pointer-change) Send signal on pointer change

**Region Functions**: Regions are used to divide the drawmap into two disjoint sets of pixels, those within the region and those outside the region. The main use of regions is to limit drawing to the area within a region, making a clipping region. Regions can be of any shape or size, they are created from the basic shapes of rectangle, polygon, circle, ellipse and wedges and can be made more complex by region intersection, union, exclusive-or or difference operations. The following functions are provided for the creation and manipulation of regions:

- [`rg_creat()`](#rg_creat-create-region) Create region
- [`rg_del()`](#rg_del-delete-region) Delete region
- [`rg_diff()`](#rg_diff-region-difference) Create region from difference of two regions
- [`rg_isect()`](#rg_isect-region-intersection) Create region from intersection of two regions
- [`rg_move()`](#rg_move-move-region) Move region
- [`rg_union()`](#rg_union-region-union) Create region from union of two regions
- [`rg_xor()`](#rg_xor-region-exclusive-or) Create region from exclusive or of two regions

**Video Inquiry Functions** are information requests used by an application to get information from the video driver:

- [`viq_cpos()`](#viq_cpos-return-relative-character-positions) Return relative character positions
- [`viq_dminfo()`](#viq_dminfo-return-drawmap-descriptor-information) Return drawmap descriptor information
- [`viq_fdta()`](#viq_fdta-return-font-data) Return font data
- [`viq_gdta()`](#viq_gdta-return-glyph-data) Return glyph data
- [`viq_jcps()`](#viq_jcps-return-character-positions-for-justified-text) Return character positions for justified text
- [`viq_pntr()`](#viq_pntr-test-if-point-is-in-region) Test if pointer is in region
- [`viq_rinfo()`](#viq_rinfo-return-region-descriptor-information) Return region descriptor information
- [`viq_rloc()`](#viq_rloc-inquire-region-location) Inquire region location
- [`viq_txtl()`](#viq_txtl-calculate-length-of-text) Calculate length of text

**The Subroutine Module Link Function** may be used to expand the functions of UCM. This subroutine may be linked with UCM through the use of the `ss_slink()` function.

#### CDFM Functions

**GetStat Functions**: The following getstat functions are supported:

- [`gs_eof()`](#gs_eof-test-for-end-of-file) Test for end-of-file
- [`gs_gcdfd()`](#gs_gcdfd-return-file-descriptor) Return file descriptor
- [`gs_opt()`](#gs_opt-get-options-of-path-descriptor) Get options of path descriptor
- [`gs_path()`](#gs_path-get-full-pathlist-of-open-file) Get full pathlist of open file
- [`gs_pos()`](#gs_pos-return-file-position) Return file position
- [`gs_size()`](#gs_size-return-file-size) Return file size

**SetStat Functions**: The following setstat functions are supported:

- [`ss_abort()`](#ss_abort-abort-operation-in-progress) Abort operation in progress
- [`ss_cchan()`](#ss_cchan-change-data-and-audio-channels) Change data and audio channels
- [`ss_cdda()`](#ss_cdda-play-cd-digital-audio) Play CD digital audio
- [`ss_cont()`](#ss_cont-continue-play-on-disc-drive) Continue play on disc drive
- [`ss_disable()`](#ss_disable-disable-hardware-control-buttons-on-player) Disable hardware control buttons on player
- [`ss_eject()`](#ss_eject-issue-door-open-command-to-disc-drive) Issue door open command to disc drive
- [`ss_enable()`](#ss_enable-enable-hardware-control-buttons-on-player) Enable hardware control buttons on player
- [`ss_mount()`](#ss_mount-mount-disc-by-disc-number) Mount disc by disc number
- [`ss_opt()`](#ss_opt-set-path-descriptor-options) Set path descriptor options
- [`ss_pause()`](#ss_pause-pause-the-disc-drive) Pause the disc drive
- [`ss_play()`](#ss_play-play-real-time-records) Play real time records
- [`ss_raw()`](#ss_raw-read-raw-sectors) Read raw sectors
- [`ss_readtoc()`](#ss_readtoc-read-table-of-contents) Read table of contents
- [`ss_seek()`](#ss_seek-move-file-position-pointer) Move file position pointer
- [`ss_stop()`](#ss_stop-stop-the-disc-drive) Stop the disc drive

**Audio Functions**: The following audio functions are supported:

- [`sc_atten()`](#sc_atten-set-volume-balance-and-position-of-sound) Set volume, balance and position of sound
- [`sd_loop()`](#sd_loop-set-soundmap-loopback-points) Set soundmap loopback points
- [`sd_mmix()`](#sd_mmix-mix-monaural-to-stereo) Mix two monaural soundmaps to a single stereo soundmap
- [`sd_smix()`](#sd_smix-mix-stereo-to-stereo) Mix two stereo soundmaps to a single monaural soundmap
- [`sm_close()`](#sm_close-close-soundmap) Close soundmap
- [`sm_cncl()`](#sm_cncl-conceal-error-in-soundmap) Conceal error in soundmap
- [`sm_creat()`](#sm_creat-create-soundmap) Create soundmap
- [`sm_info()`](#sm_info-get-soundmap-information) Get soundmap information
- [`sm_off()`](#sm_off-turn-off-audio-processor) Turn off audio processor
- [`sm_out()`](#sm_out-output-soundmap) Output soundmap
- [`sm_stat()`](#sm_stat-return-current-play-status) Return current play status

#### CSD Functions

To access the CSD, the following C functions are provided:

- [`csd_deldev()`](#csd_deldev-delete-device-from-csd-by-name) Delete device from CSD
- [`csd_devnam()`](#csd_devnam-get-device-name-by-type-and-number) Get Device name by type and number
- [`csd_devparam()`](#csd_devparam-get-parameter-string) Get Parameter String Associated with a device
- [`csd_setparam()`](#csd_setparam-set-parameter-string) Set parameter list for a device

### OS9.L: The OS-9 C Library

**Alarm Functions**: In order for an application program to be able to use the OS-9 `F$Alarm` system call, the following C Bindings have been provided:

**Colored Memory Functions**: In order to allocate colored memory or load a module into colored memory, the following C Bindings are provided:

## CD-I System Libraries: C Library Functions

This chapter describes the operation and usage of each library function. The functions appear in alphabetical order for quick access.

Each function description includes a synopsis and details on the usage of the function. The synopsis shows how the function and arguments would look if written as a C function definition, even if the actual function is a macro or is written in assembly language.

-----

### `alm_atdate()` Send signal at a specified Gregorian date/time

*OS-9 System Call C Binding*

```c
alm_atdate(sigcode, time, date)
int sigcode,            /* signal to send */
    time,               /* time given in 00hhmmss */
    date;               /* date given in yyyymmdd */
```

`alm_atdate()` sends the specified signal to the caller at the specified Gregorian date and time. This function will also send the signal if the system's date is changed to be equal or less than the alarm time through
the use of `F$STime`.

`alm_atdate()` returns the id of the alarm if successful; otherwise `-1` is returned and `errno` is set.

-----

### `alm_atjul()` Send signal at a specified Julian date/time

*OS-9 System Call C Binding*

```c
alm_atjul(sigcode, time, date)
int sigcode,            /* signal to send */
    time,               /* time given in 00hhmmss */
    date;               /* date given in yyyymmdd */
```

`alm_atjul()` sends the specified signal to the caller at the specified Julian date and time. This function will also send the signal if the system's date is changed to be equal or less than the alarm time through the use of `F$STime`.

`alm_atjul()` returns the id of the alarm if successful; otherwise `-1` is returned and `errno` is set.

-----

### `alm_cycle()` Send a signal at specified time intervals

*OS-9 System Call C Binding*

```c
alm_cycle(sigcode, timeinterva1)
int sigcode,            /* signal to send */
    timeinterval;       /* time interval */
```

`alm_cycle()` periodically sends the specified signal to the caller at the specified interval.

`timeinterval` may be specified in ticks or 1/256 of a second units. If the high order bit is set, the low 31 bits are interpreted as 1/256 second units. If `timeinterval` is zero, `alm_cycle()` will return an `E$IllPrm` error code.

**NOTE**: All time intervals are rounded up to the nearest 0.01 second.

`alm_cycle()` returns the id of the alarm if successful; otherwise `-1` is returned and `errno` is set.

-----

### `alm_delete()` Remove a pending alarm request

*OS-9 System Call C Binding*

```c
alm_delete(alarmid)
int alarmid;            /* alarm id */
```

`alm_delete()` removes the specified alarm request. If zero is passed as the `alarmid`, all pending alarm requests are removed.

`alm_delete()` returns the id of the alarm if successful; otherwise `-1` is returned and `errno` is set.

-----

### `alm_set()` Send a signal after a time interval

*OS-9 System Call C Binding*

```c
alm_set(sigcode, timeinterval)
int sigcode,            /* signal to send */
    timeinterval;       /* time interval */
```

`alm_set()` sends the specified signal to the caller after the specified interval.

`timeinterval` may be specified in ticks or 1/256 of a second units. If the high order bit is set, the low 31 bits are interpreted as 1/256 second units. If `timeinterval` is zero, `alm_set()` will return an `E$IllPrm` error code.

**NOTE**: All time intervals are rounded up to the nearest 0.01 second.

`alm_set()` returns the id of the alarm if successful; otherwise `-1` is returned and `errno` is set.

-----

### `co_afnt()` Activate Font

*UCM File Manager C Binding*

```c
co_afnt(path, fntid, afn)
int path,               /* path to video device */
    fntid,              /* font id */
    afn;                /* active font number */
```

`co_afnt()` activates the font specified by `fntid` for use by `I$Write` and `I$Writeln`.

`afn` specifies the active font number to be associated with this font. Up to four fonts (0-3) may be active at any given time.

This is similar to the `dp_afnt()` function for graphics text output. The font must have previously been loaded by use of the `dp_gfnt()` function.

`-1` is returned on error and `errno` is set.

-----

### `co_cod()` Set Character Output Drawmap

*UCM File Manager C Binding*

```c
co_cod(path, dmid)
int path,               /* path to video device*/
    dmid;               /* drawmap id */
```

`co_cod()` sets the drawmap specified by dmid as the drawmap in which all standard character output will take place.

`-1` is returned on error and `errno` is set.

-----

### `co_dfnt()` Deactivate Font

*UCM File Manager C Binding*

```c
co_dfnt(path, afn)
int path,               /* path to video device */
    afn;                /* active font number */
```

`co_dfnt()` deactivates the standard output font specified by `afn`.

`-1` is returned on error and `errno` is set.

-----

### `co_scmm()` Set Character Code Mapping Method

*UCM File Manager C Binding*

```c
co_scmm(path, mapmethod)
int path,               /* path to video device*/
    mapmethod:          /* mapping method for character codes */
```

`co_scmm()` sets the manner in. which `I$Write` and `I$Writeln` will interpret the character codes they receive. `mapmethod` can take one of three values:

- `0` eight bit
- `1` seven/fifteen bit
- `2` sixteen bit

-----

### `cp_bkcol()` Load backdrop color

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_bkco1(intensity, color)
int intensity,          /* Intensity */
    color;              /* Backdrop Color */
```

This function returns an instruction which may be written into an LCT or a FCT.

`intensity` can take one of the two values:

BK_HIGH         High Intensity
BK_LOW          Low Intensity

`color` can take one of the eight following values:

- `BK_BLACK`        Backdrop color is BLACK
- `BK_BLUE`         Backdrop color is BLUE
- `BK_GREEN`        Backdrop color is GREEN
- `BK_CYAN`         Backdrop color is CYAN
- `BK_RED`          Backdrop color is RED
- `BK_MAGENTA`      Backdrop color is MAGENTA
- `BK_YELLOW`       Backdrop color is YELLOW
- `BK_WHITE`        Backdrop color is WHITE

Refer to chapter V of the Green Book for a description of the parameter(s) to this functions.

**Note**: This function is implemented as a macro.

-----

### `cp_cbnk()` Set CLUT bank

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_cbnk(bank)
int bank;               /* Bank of the CLUT */
```

This function returns an instruction which may be written into an LCT or a FCT.

`bank` can take values between 0 and 3.
Refer to chapter V of the Green Book for a description of the parameter(s) to this functions.

> - `0` Colors 0-63 of plane A
> - `1` Colors 64-127 plane A
> - `2` Colors 0-63 of plane B
> - `3` Colors 64-127 plane B

**Note**: this function is implemented as a macro.

-----

### `cp_clut()` Load CLUT color

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_clut(clut, r, g, b)
int clut,               /* CLUT number */
    r, g, b;            /* Red, Blue and Green components */
```

This function returns an instruction which may be written into an LCT or a FCT.

`clut` can take values between 0 and 63. `r`, `g` and `b` can take values between 0 and 255.

Refer to chapter V of the Green Book for a description of the parameter(s) to this functions.

**Note**: this function is implemented as a macro.

-----

### `cp_crmatte()` Create an Array of Matte Instructions from a Region

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_crmatte(rgdef, array, width, height,  matreg, incmd, outcmd)
RegionDesc *rgdef;      /* pointer to region definition */
int *array,             /* pointer to buffer for instructions */
    width,              /* width of buffer for instructions */
    height,             /* height of buffer for instructions */
    matreg,             /* first matte register to use */
    incmd,              /* command for transitions into matte */
    outcmd;             /* command for transitions out of matte */
```

This function fills an array with instructions which may be written into columns of an LCT.

`incmd` and `outcmd` should be computed with the cp_matte() function.
Only the `OP`, `MF`, and `ICF` fields of these two parameters are taken into account when filling an array.

The vertical coordinate of the upper left comer of the regions's boundary rectangle will correspond to the first line of the array.

When a matte is placed on the screen, the vertical position of the matte on the screen is determined by the first line from which the array is written in the LCT. The horizontal position of the matte on the screen is
determined by the X positions contained in the matte instructions in the array. These X positions can be modifed in the array by the function `cp_offset_matte()`.

The height specified must be large enough to contain one (1) extra line for the matte end instruction. The instruction slots which are not filled with matte instructions will be filled with no operation instructions.

An error is returned when the array is not wide or not long enough to hold the matte definition. The matte may still be used in this case, but it will be incomplete.

A zero is returned on a successful call. If an error occurs, `-1` is returned and `errno` is set.

-----

### `cp_dadr()` Load display line start pointer

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_dadr(address)
int address;            /* Display Line Start Pointer */
```

This function returns an instruction which may be written into an LCT or a FCT.

Refer to chapter V of the Green Book for a description of the parameter(s) to this functions.

**Note**: this function is implemented as a macro.

-----

### `cp_dadr_col()` Create an Array of Display Start Address Instructions

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_dadr_co1(dmdsc, dmp, array, x, y, numlines, linerepeat, linehold)

DrawmapDesc *dmdsc;     /* pointer to drawmap desc */
int dmp,                /* dp plane (used for multi-plane dmaps) */
    *array,             /* pointer to buffer for instructions */
    x, y,               /* X, Y coordinate for first instruction*/
    numlines,           /* number of instructions to generate*/
    linerepeat,         /* line repeat value (for vertical zoom)*/
    linehold;           /* line hold value (for vertical granulation) */
```

This function returns an array of instructions which may be written into a column of an LCT.

When an application wants to display a part of a drawmap, it will use this function to create an array of Load Display Line Start Address instructions.

A drawmap may be larger and longer than the physical screen and the CD-I hardware allows you to display any part of this drawmap. The coordinate `x` and `y` determine the pixel in the drawmap from which the display will be done.

The `linerepeat` and `linehold` parameters are used to perform vertical mosaics.

A zero is returned on a successful call. If an error occurs, `-1` is returned and `errno` is set.

-----

### `cp_dprm()` Load display parameters

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_dprm(rms, prf, bp)
int rms,                /* Run length or Mosaic Select */
    prf,                /* Pixel Repeat Factor */
    bp;                 /* Bit per Pixel */

```

This function returns an instruction which may be written into an LCT or a FCT.

`rms` can take one of the three values:

- `RMS_NORMAL`      Runlength and mosiac disabled
- `RMS_RL`          Runlength enabled
- `RMS_MOSAIC`      Mosaic enabled

`prf` can take one of the four values:

- `PRF_X2`          Pixel repeat factor = times 2
- `PRF_X4`          Pixel repeat factor = times 4
- `PRF_X8`          Pixel repeat factor = times 8
- `PRF_X16`         Pixel repeat factor = times 16

`bp` can take one of the three values:

- `BP_NORMAL`       8 bits/pixel, normal resolution
- `BP_DOUBLE`       4 bits/pixel, double or high resolution
- `BP_HIGH`         8 bits/pixel, high resolution

**Note**: this function is implemented as a macro.

-----

### `cp_icf()` Load image contribution factor

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_icf(plane, icf)
int plane,              /* Plane */
    icf;                /* Image Contribution Factor */
```

This function returns an instruction which may be written into an LCT or a FCT.

`plane` can take one of the two values:

- `PA`              Plane A
- `PB`              Plane B

`icf` can take values between

- `ICF_MIN`         Minimum image contribution factor
- `ICF_MAX`         Maximum image contribution factor

**Note**: this function is implemented as a macro.

-----

### `cp_icm()` Select image coding method

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_icm(cm0, cml, nm, ev, cs)
int cm0,                /* Coding method for path 0 */
    cm1,                /* Coding method for path 1 */
    nm,                 /* Number of mattes */
    ev,                 /* External video */
    cs;                 /* CLUT bank selector */
```

This function returns an instruction which may be written into an LCT or a FCT.

`cm0` and `cml` can take one of the following values:

- `ICM_OFF`         Plane disabled
- `ICM_CLUT4`       Color Look Up Table with 4 bits per pixel
- `ICM_CLUT7`       Color Look Up Table with 7 bits per pixel
- `ICM_CLUTS`       Color Look Up Table with 8 bits per pixel
- `ICM_CLUT77`      CLUT7 or 7 Mode (Dual CLUT)
- `ICM_DYUV`        Natural Image DYUV
- `ICM_RGB`         RGB555

The following table describes what combination of values for `cm0` and `cml` are valid.

| &#x25BE;CM1&nbsp;CM0&#x25B8; | ICM_OFF | ICM_CLUT4 | ICM_CLUT7 | ICM_CLUT8 | ICM_CLUT77 | ICM_DYUV | ICM_RGB |
| :------:                     | :-----: | :-------: | :-------: | :-------: | :--------: | :------: | :-----: |
| **ICM_OFF**                  | ok      | ok        | ok        | ok        | ok         | ok       |         |
| **ICM_CLUT4**                | ok      | ok        | ok        |           |            | ok       |         |
| **ICM_CLUT7**                | ok      | ok        | ok        |           |            | ok       |         |
| **ICM_CLUT8**                |         |           |           |           |            |          |         |
| **ICM_CLUT77**               |         |           |           |           |            |          |         |
| **ICM_DYUV**                 | ok      | ok        | ok        | ok        | ok         | ok       |         |
| **ICM_RGB**                  | ok      |           |           |           |            |          |         |

`nm` can take one of the two values:

- `NM_1`            One Matte Register
- `NM_2`            Two Matte Registers

`ev` can take one of the two values:

- `EV_ON`           External Video Enabled
- `EV_OFF`          External Video Disabled

`cs` is the CLUT selector for dual 7-bit CLUTs (`ICM_CLUT77`). It may have have either of these values: `CS_A` (Bank 0 and 1) or `CS_B` (Bank 2 and 3).

A zero is returned on a successful call. If an error occurs, `-1` is returned and `errno` is set.

-----

### `cp_matte()` Load matte register 0-7

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_matte(reg, op, mf, icf, x)
int reg,                /* Matte Register */
    op,                 /* Operation Code */
    mf,                 /* Matte Flag */
    icf,                /* Image Contribution Factor */
    x                   /* X Position */
```

This function returns an instruction which may be written into an LCT or a FCT.

`reg` can take values between 0 and 7.

`op` can take one of the following values:

- `MO_END`          Disregard commands in higher registers
- `MO_ICF_0`        Change ICF for plane A
- `MO_ICF_1`        Change ICF for plane B
- `MO_RES`          Reset matte flag to FALSE
- `MO_SET`          Set matte flag to true
- `MO_RES_ICF0`     Reset matte flag and change ICF for plane A
- `MO_SET_ICF0`     Set matte flag and change ICF for plane A
- `MO_RES_ICF1`     Reset matte flag and change ICF for plane B
- `MO_SET_ICF1`     Set matte flag and change ICF for plane B

`mf` can take one of the two values:

- `MF_MF0`          Matte flag to be changed = 0
- `MF_MF1`          Matte flag to be changed = 1

`icf` can take values between

- `ICF_MIN`         Minimum image contribution factor
- `ICF_MAX`         Maximum image contribution factor

The `x` parameter must be expressed in double resolution.

**Note**: this function is implemented as a macro.

-----

### `cp_mcol()` Load mask color

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_mcol(plane, r, g, b)
int plane,              /*Plane*/
    r, g, b;            /* Red, Blue and Green components */
```

This function returns an instruction which may be written into an LCT or a FCT.

`plane` can take one of the two values:

- `PA`              Plane A
- `PB`              Plane B

`r`, `g` and `b` can take values between 0 and 255.

**Note**: this function is implemented as a macro.

-----

### `cp_nop()` No operation

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_nop()
```

This function returns an instruction which may be written into an LCT or a FCT.

**Note**: this function is implemented as a macro.

-----

### `cp_offset_matte()` Add X Offset to an Array of Matte Instructions

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_offset_matte(array, width, height, x)
int *array,             /* pointer to buffer of matte instructions */
    width,              /* buffer width */
    height,             /* buffer height */
    x;                  /* X offset (may be + or -) */
```

This function updates array wich contains matte instructions.

Given an array of matte instructions, this function modifies the x position parameter of the matte instructions contained in the array. An application will use this function when it wants to place a matte on a different horizontal position.

A zero is returned on a successful call. If an error occurs, `-1` is returned and `errno` is set.

-----

### `cp_phld()` Load mosaic pixel hold factor

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_phld(plane, en, factor)
int plane,              /* Plane */
    en,                 /* Pixel Hold Enable */
    factor;             /* Pixel Hold Factor*/
```

This function returns an instruction which may be written into an LCT or a FCT.

`plane` can take one of the two values:

- `PA`              Plane A
- `PB`              Plane B

`en` can take one of the two values:

- `PH_ON`           Pixel Hold Enabled
- `PH_OFF`          Pixel Hold Disabled

**Note**: this function is implemented as a macro.

-----

### `cp_po()` Load plane order

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_po(order)
int order;              /* Order of the two planes */
```

This function returns an instruction which may be written into an LCT or a FCT.

`order` can take one of the two values:

- `PR_AB`           Plane A in front of plane B
- `PR_BA`           Plane B in front of plane A

**Note**: this function is implemented as a macro.

-----

### `cp_sig()` Signal when scan reaches this line

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_sig()
```

This function returns an instruction which may be written into an LCT or a FCT.

**Note**: this function is implemented as a macro.

-----

### `cp_tci()` Load transparency control information

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_tci(mix, ta, tb)
int mix,                /* Mixing Flag */
    ta,                 /* Transparency mechanism for plane A */
    tb;                 /* Transparency mechanism for plane B */
```

This function returns an instruction which may be written into an LCT or a FCT.

`mix` can take one of the two values:

- `MIX_ON`          Mixing Enabled
- `MIX_OFF`         Mixing Disabled

`ta` and `tb` can take one of the following values:

- `TR_ON`               Always (plane disabled)
- `TR_CKEY_T`           Color key = TRUE
- `TR_TBIT_1`           Transparent bit = 1
- `TR_MAT0_T`           Matte Flag 0 = TRUE
- `TR_MAT1_T`           Matte Flag 1 = TRUE
- `TR_M0CK_T`           Matte Flag 0 = TRUE or Color key = TRUE
- `TR_M1CK_T`           Matte Flag 1 = TRUE or Color key = TRUE
- `TR_OFF`              Never (No transparent area)
- `TR_CKEY_F`           Color key = FALSE
- `TR_TBIT_F`           Transparent bit = 0
- `TR_MAT0_F`           Matte Flag 0 = FALSE
- `TR_MAT1_F`           Matte Flag 1 = FALSE
- `TR_M0CK_F`           Matte Flag 0 = FALSE or Color key = FALSE
- `TR_M1CK_F`           Matte Flag 1 = FALSE or Color key = FALSE

**Note**: this function is implemented as a macro.

-----

### `cp_tcol()` Load Transparent color

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_tcol(plane, r, g, b)
int plane,              /* Plane */
    r, g, b;            /* Red, Blue and Green components */
```

This function returns an instruction which may be written into an LCT or a FCT.

`plane` can take one of the two values:

- `PA`              Plane A
- `PB`              Plane B

`r`, `g` and `b` can take values between 0 and 255.

**Note**: this function is implemented as a macro.

-----

### `cp_yuv()` Load DYUV start value

*UCM File Manager C Binding*

```c
#include <ucm.h>

cp_yuv(plane, y, u, v)
int plane,              /* Plane */
    y,                  /* Absolute Y value */
    u,                  /* Absolute U value */
    v;                  /* Absolute V value */
```

This function returns an instruction which may be written into an LCT or a FCT.

`plane` can take one of the two values:

- `PA`              Plane A
- `PB`              Plane B

`y`, `u` and `v` can take values between 0 and 255.

**Note**: this function is implemented as a macro.

-----

### `crt_cdwn()` Cursor Down

*UCM File Manager C Binding*

```c
crt_cdwn(path)
int path;               /* Path to video device */
```

`crt_cdwn()` moves the cursor down one line in the same column. If the cursor is on the bottom line, the screen will scroll up one line.

`-1` is returned on error and `errno` is set.

-----

### `crt_ceol()` Clear to End of Line

*UCM File Manager C Binding*

```c
crt_ceol(path)
int path;               /* Path to video device */
```

`crt_ceol()` causes all characters from the cursor position to the end of the line to be cleared to spaces.

`-1` is returned on error and `errno` is set.

-----

### `crt_ceos()` Clear to End of Screen

*UCM File Manager C Binding*

```c
crt_ceos(path)
int path;               /* Path to video device */
```

`crt_ceos()` causes all characters from the cursor postion to the end of the screen to be cleared to spaces.

`-1` is returned o:n error and `errno` is set.

-----

### `crt_clft()` Cursor left

*UCM File Manager C Binding*

```c
crt_clft(path)
int path;               /* Path to video device */
```

`crt_clft()` moves the cursor one column to the left. If the cursor is in the first column of a line, it moves to the last column of the previous line. If the cursor is in the first column of the first line, this function has no effect.

`-1` is returned on error and `errno` is set.

-----

### `crt_cr()` Carriage Return

*UCM File Manager C Binding*

```c
crt_cr(path)
int path;               /* Path to video device */
```

`crt_cr()` moves the cursor to the first position on the current line.

`-1` is returned on error and `errno` is set.

-----

### `crt_crgt()` Cursor right

*UCM File Manager C Binding*

```c
crt_crgt(path)
int path;               /* Path to video device */
```

`crt_crgt()` moves the cursor one column to the right. If the cursor is in the last column and autowrap is on, it moves to the first position on the next line. If the cursor is in the last column of the bottom line and autowrap is on, the screen is scrolled up one line and the cursor is placed in the first position of the new bottom line.

`-1` is returned on error and `errno` is set.

> This function is also referred to as `crt_crght()` in the documentation, which one is correct is to be determined

-----

### `crt_cscrn()` Clear Screen

*UCM File Manager C Binding*

```c
crt_cscrn(path)
int path;               /* Path to video device */
```

`crt_cscrn()` causes all character positions to be cleared to spaces

`-1` is returned on error and `errno` is set.

-----

### `crt_cup()` Cursor Up

*UCM File Manager C Binding*

```c
crt_cup(path)
int path;               /* Path to video device */
```

`crt_cup()` moves the cursor up one line in the same column. If the cursor is on the top line, it is not moved.

`-1` is returned on error and `errno` is set.

-----

### `crt_curxy()` Cursor Address

*UCM File Manager C Binding*

```c
crt_curxy(path)
int path,               /* Path to video device */
    x, y;               /* Cursor coordinates */
```

`crt_curxy()` positions the cursor at the `x`, `y` coordinates given. The coordinates given must have an offset of `0x20`.

`-1` is returned on error and `errno` is set.

#### Example

- `crt_curxy(path, 0x20, 0x20)` moves the cursor to the home position

-----

### `crt_dchar()` Delete Character

*UCM File Manager C Binding*

```c
crt_dchar(path)
int path;               /* Path to video device */
```

`crt_dchar()` causes all characters to the right of the cursor to be moved one column to the left. The character under the cursor is destroyed and the last character in the line is cleared to a space.

`-1` is returned on error and `errno` is set.

-----

### `crt_dlin()` Delete Line

*UCM File Manager C Binding*

```c
crt_dlin(path)
int path;               /* Path to video device */
```

`crt_dlin()` deletes the line on which the cursor is currently placed. All following lines are scrolled up one line and the bottom line is cleared to spaces. The cursor remains in place.

`-1` is returned on error and `errno` is set.

-----

### `crt_hidcur()` Hide Cursor

*UCM File Manager C Binding*

```c
crt_hldcur(path)
int path;               /* Path to video device */
```

`crt_hidcur()` causes the text cursor to disappear from the screen.

`-1` is returned on error and `errno` is set.

-----

### `crt_home()` Cursor Home

*UCM File Manager C Binding*

```c
crt_home(path)
int path;               /* Path to video device */
```

`crt_home()` is used to move the cursor to the first column of the top line.

`-1` is returned on error and `errno` is set.

-----

### `crt_ichar()` Insert Character

*UCM File Manager C Binding*

```c
crt_ichar(path)
int path;               /* Path to video device */
```

`crt_ichar()` causes the character under the cursor and all characters to
the right to be moved one column to the right. The character under the
cursor is cleared to a space.

`-1` is returned on error and `errno` is set.

-----

### `crt_ilin()` Insert Line

*UCM File Manager C Binding*

```c
crt_ilin(path)
int path;               /* Path to video device */
```

`crt_ilin()` causes the line on which the cursor is placed and all lines following to be scrolled down one line. The line with the cursor is then cleared to spaces.

`-1` is returned on error and `errno` is set.

-----

### `crt_revoff()` End Reverse Video

*UCM File Manager C Binding*

```c
crt_revoff(path)
int path;               /* Path to video device */
```

`crt_revoff()` turns off reverse video.

`-1` is returned on error and `errno` is set.

-----

### `crt_revon()` Start Reverse Video

*UCM File Manager C Binding*

```c
crt_revon(path)
int path;               /* Path to video device */
```

`crt_revon()` turns on reverse video.

`-1` is returned on error and `errno` is set.

-----

### `crt_shwcur()` Show Cursor

*UCM File Manager C Binding*

```c
crt_shwcur(path)
int path;               /* Path to video device */
```

`crt_shwcur()`  causes the text cursor to appear on the screen.

`-1` is returned on error and `errno` is set.

-----

### `crt_uloff()` End Underlining

*UCM File Manager C Binding*

```c
crt_uloff(path)
int path;               /* Path to video device */
```

`crt_uloff()` turns off underlining.

`-1` is returned on error and `errno` is set.

-----

### `crt_ulon()` Start Underlining

*UCM File Manager C Binding*

```c
crt_ulon(path)
int path;               /* Path to video device */
```

`crt_ulon()` turns on underlining.

`-1` is returned on error and `errno` is set.

-----

### `crt_wrapoff()` Turn Auto-Wrap mode Off

*UCM File Manager C Binding*

```c
crt_wrapoff(path)
int path;               /* Path to video device */
```

`crt_wrapoff()` causes auto-wrap mode to be turned off.

`-1` is returned on error and `errno` is set.

-----

### `crt_wrapon()` Turn Auto-Wrap mode On

*UCM File Manager C Binding*

```c
crt_wrapon(path)
int path;               /* Path to video device */
```

`crt_wrapon()` causes auto-wrap mode to be turned on.

`-1` is returned on error and `errno` is set.

-----

### `csd_deldev()` Delete Device from CSD by Name

*Configuration Status Descriptor C Binding*

```c
csd_deldev(name)
char *name;             /* Name of device to remove */
```

`csd_deldev()` removes the Device Status Descriptor (DSD) entry with the specified device name.

If unsuccessful, `csd_deldev()` returns `-1`and the appropriate error code is placed in the global variable `errno`.

**NOTE**: No error is returned when the specified device is not found.

-----

### `csd_devnam()` Get Device Name by type and number

*Configuration Status Descriptor C Binding*

```c
#include <csd.h>

char *csd_devnam(type, num)

int type,               /* Device type code */
    num;                /* Ordinal number of device in CSD */
```

`csd_devnam()` returns a dynamic pointer to the requested device name from the CSD. The data space used for the name string may be returned to the system by using `free()`.

`type` is an integer representing the general class of functional devices to which this device belongs. The possible values for type are defined in the `DEFS` file `csd.h` and below:

```c
/* Base Case Devices */
    #define DT_CDC      1   /* CD-Control Unit */
    #define DT_AUDIO    2   /* Audio Processor */
    #define DT_VIDEO    3   /* Video Output Processor */
    #define DT_NVRAM    4   /* Non-volatile Random Access memory */
    #define DT_PTR      5   /* Pointing Device */
    #define DT_PIPEDEV  9   /* Pipe Device */

/* Base Case Peripherals */
    #define DT_CDPLAY   6   /* CD-Player */
    #define DT_AUDSET   7   /* Audio Set */
    #define DT_DISPLY   8   /* Display Monitor */

/* Peripheral Extensions */
    #define DT_KEYBRD  10   /* Keyboard */
```

`num` specifies the ordinal number of the device to return. For example, if `num = 1`, the name of the first device of the specified type is returned. If `num = 2`, the name of the second device of the specified type is returned.

If unsuccessful, `csd_devnam()` returns NULL and the appropriate error
code is placed in the global variable `errno`.

-----

### `csd_devparam()` Get Parameter String

*Configuration Status Descriptor C Binding*

```c
char *csd_devparam(name)
int *name;              /* Pointer to name of device */
```

`csd_devparam()` returns a dynamic pointer to the parameter string of the specified device, name. The data space used for the parameter string may be returned to the system by using `free()`.

If unsuccessful, `csd_devparam()` returns NULL and the appropriate error code is placed in the global variable `errno`.

-----

### `csd_setparam()` Set Parameter String

*Configuration Status Descriptor C Binding*

```c
csd_setparam(name, param)
char *name,             /* Pointer to name of device */
     *param;            /* Pointer to parameter string */
```

`csd_setparam()` assigns a new parameter string to an existing device.

If unsuccessful, `csd_setparam()` returns `-1`and the appropriate error code is placed in the global variable `errno`.

-----

### `dc_crfct()` Create Field Control Table

*UCM File Manager C Binding*

```c
dc_crfct(path, planeno, fctlen, res)

int path,               /* Path to video device */
    planeno,            /* Plane number */
    fctlen,             /* # of instructions in FCT */
    res;                /* FCT resolution */
```

`dc_crfct()` creates a Field Control Table (FCT) and returns an FCT identifier. The FCT contains the instructions which the display control hardware for the specified plane (`planeno`) will execute during the vertical retrace period. `fctlen` specifies the number of instructions for the FCT.

If `res` is 0, the FCT will be created for a low resolution display.

If `res` is set to one, the driver will create an FCT for a high resolution display. This effectively creates two FCT's. If interface mode is set, one FCT will be executed on even frames and the second will be executed on odd frames.

You must create one FCT for each plane to display. For each of these planes, one or more LCT' s may be created.

`-1` is returned on error and `errno` is set.

-----

### `dc_crlct()` Create Line Control Table

*UCM File Manager C Binding*

```c
dc_crlct(path, planeno, lctlen, res)

int path,               /* Path to video device */
    planeno,            /* Plane number */
    lctlen,             /* Number of lines in LCT */
    res;                /* FCT resolution */
```

`dc_crlct()` allocates a Line Control Table, LCT. The LCT contains instructions which the display control hardware for the specified plane (`planeno`) will execute during horizontal retrace periods.

The `lctlen` specifies the number of lines in the LCT in high resolution coordinates.

`res` specifies the resolution of the LCT: low resolution = 0, high resolution = 1. If a low resolution LCT is specified, even and odd lines will be interleaved. If a high resolution LCT is specified, even and odd lines will be separated.

You must create one FCT for each plane to display. For each of these planes, one or more LCT's may be created.

If the call is successful the LCT identifier will be returned. `-1` is returned
on error and `errno` is set.

-----

### `dc_dlfct()` Delete Field Control Table

*UCM File Manager C Binding*

```c
dc_dlfct(path, fctid)

int path,               /* Path to video device */
    fctid;              /* FCT id */
```

`dc_dlfct()` deletes the specified FCT and frees up any memory used by it.

`-1` is returned on error and `errno` is set.

-----

### `dc_dllct()` Delete Line Control Table

*UCM File Manager C Binding*

```c
dc_dllct(path, fctid)

int path,               /* Path to video device */
    lctid;              /* LCT id */
```

`dc_dllct()` deletes the specified LCT and frees up any memory used by it.

`-1` is returned on error and `errno` is set.

-----

### `dc_exec()` Execute Display Control Program

*UCM File Manager C Binding*

```c
dc_exec(path, fctid1, fctid2)

int path,               /* Path to video device */
    fctid1,             /* ID of first FCT */
    fctid2;             /* ID of second FCT */
```

`dc_exec()` causes the display control hardware to execute the display control program associated with the given FCT's. `fctid1` corresponds to plane A and `fctid2` corresponds with plane B. If either id is 0, the default FCT for that plane is executed.

`-1` is returned on error and `errno` is set.

-----

### `dc_flnk()` Link FCT to LCT

*UCM File Manager C Binding*

```c
dc_flnk(path, fctid, lctid, lctline)
int path,               /* Path to video device */
    fctid,              /* FCT id */
    lctid,              /* LCT id */
    lctline;            /* LCT line number */
```

`dc_flnk()` links the FCT specified by fetid to the lCT specified by `lctid` at the line nwnber given by `lctline`. `lctline` is specified as a high resolution logical line number.

If the FCT is high resolution and the LCT is low resolution, both FCT's for the plane will point to the same LCT line. If the LCT is high resolution, the even field FCT will point to the given line and the odd field FCT will point to the next line.

`-1` is returned on error and `errno` is set.

-----

### `dc_intl()` Set Display Interlace Mode

*UCM File Manager C Binding*

```c
dc_intl(path, mode)
int path,               /* Path to video device */
    mode;               /* Display interface mode */
```

`dc_intl()` sets the display interlace mode. mode is defined as follows:

- `0`               No interlace
- `1`               Repeat field interlace

`-1` is returned on error and `errno` is set.

-----

### `dc_llnk()` Link LCT to LCT

*UCM File Manager C Binding*

```c
dc_llnk(path, lctid1, line1, lctid2, line2)
int path,               /* Path to video device */
    lctid1,             /* ID of first LCT */
    line1,              /* LCT line to contain link instruction */
    lctid2,             /* ID of second LCT */
    line2;              /* LCT line to link to */
```

`dc_llnk()` links the LCT specified by `lctid1` to the LCT specified by `lctid2`.

By linking the two LCTs, the display control hardware is told to begin retrieving instructions from the second LCT. `line1` specifies the line number in the first LCT to contain the link instruction. `line2` specifies the
line number in the second LCT to which the link is made.

It should be noted that the link instruction will be placed in the last instruction in the specified line, `line1`. If there is already an instruction in that line, it will be overwritten.

-----

### `dc_nop()` Write array of NOP instructions to LCT

*UCM File Manager C Binding*

```c
dc_nop(path, lctid, startline, startcol, numlines, numcols)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    startline,          /* LCT line to begin write */
    startcol,           /* LCT column to begin write */
    numlines,           /* Number of lines to write */
    numcols;            /* Number of columns to write */
```

`dc_nop()` writes an array of NOP instructions to the LCT specified by `lctid`. The array will be written starting at the line specified by `startline` and `startcol`. The number of columns per line to be written is specified by `numcols`. The number of lines to be written is specified by `numlines`.

The function returns `-1`on error and `errno` is set.

-----

### `dc_prdlct()` Physical Read of LCT Columns

*UCM File Manager C Binding*

```c
dc_prdlct(path, lctid, lineno, colno, numlines, numcols, buff)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    lineno,             /* LCT line number to begin read */
    colno,              /* LCT column number to begin read */
    numlines,           /* Physical number of lines to read */
    numcols,            /* Physical number of columns to read */
    *buff;              /* Pointer to buffer to hold data read */
```

`dc_prdlct()` reads a two dimensional array of instructions from the LCT into the buffer pointed to by `buff`. The first element of the array to be read is specified by `lineno` and `colno`. These values reference the physical LCT regardless of resolution. The number of columns per line to be read is specified by `numcols`. The number of lines to be read is specified by `numlines`.

`-1` is returned on error and `errno` is set.

-----

### `dc_pwrlct()` Physical Write to LCT Columns

*UCM File Manager C Binding*

```c
dc_pwrlct(path, lctid, lineno, colno, numlines, numcols, buff)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    lineno,             /* LCT line number to begin write */
    colno,              /* LCT column number to begin write */
    numlines,           /* Physical number of lines to write */
    numcols,            /* Physical number of columns to write */
    *buff;              /* Pointer to buffer to hold data write */
```

`dc_pwrlct()` writes data from the buffer pointed to by `buff` to a two dimensional array in the LCT specified by `lctid`. The first element of the array to be written is specified by `lineno` and `colno`. These values reference the physical LCT regardless of resolution. The number of columns per line to be written is specified by `numcols`. The number of lines to be written is specified by `numlines`.

-----

### `dc_rdfct()` Read Field Control Table

*UCM File Manager C Binding*

```c
dc_rdfct(path, fctid, startins, count, buff)
int path,               /* Path to video device */
    fctid,              /* FCT id */
    startins,           /* FCT instruction at which to begin */
    count,              /* Number of instruction to be read */
    *buff;              /* Pointer to buffer to hold data read */
```

`dc_rdfct()` reads count instructions from the specified FCT starting at instruction `startins`. The data will be copied into the buffer pointed to by `buff`.

`-1` is returned on error and `errno` is set.

-----

### `dc_rdfi()` Read Field Control Table Instruction

*UCM File Manager C Binding*

```c
dc_rdfi(path, fctid, number)
int path,               /* Path to video device */
    fctid,              /* FCT id */
    number;             /* FCT instruction number to read */
```

`dc_rdfi()` reads the single instruction specified by number from the FCT and returns it to the application.

`-1` is returned on error and `errno` is set.

-----

### `dc_rdlct()` Read Line Control Table Columns

*UCM File Manager C Binding*

```c
dc_rdlct(path, lctid, lineno, colno, numlines, numcols, buff)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    lineno,             /* LCT line number to begin read */
    colno,              /* LCT column number to begin read */
    numlines,           /* Physical number of lines to read */
    numcols,            /* Physical number of columns to read */
    *buff;              /* Pointer to buffer to hold data read */
```

`dc_rdlct()` reads a two dimensional array of instructions from the LCT into the buffer pointed to by `buff`. The first element of the array to be read is specified by `lineno` and `colno`. The number of columns per line to be read is specified by `numcols`. The number of lines to be read is specified by `numlines`.

`-1` is returned on error and `errno` is set.

-----

### `dc_rdli()` Read Line Control Table Instruction

*UCM File Manager C Binding*

```c
dc_rdli(path, lctid, lineno, insnum)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    lineno,             /* LCT line number to read */
    insnum;             /* LCT column number to read */
```

`dc_rdli()` returns the instruction specified by `lineno` and `insnum` in the LCT specified by `lctid`.

`-1` is returned on error and `errno` is set.

-----

### `dc_relea()` Release Pending Signal Request

*UCM File Manager C Binding*

```c
dc_relea(path)
int path;               /* Path to video device */
```

`dc_relea()` informs the driver not to send a pending signal that was previously set up using `dc_ssig()`.

-----

### `dc_setcmp()` Set Compatibility Mode

*UCM File Manager C Binding*

```c
dc_setcmp(path, mode)
int path,               /* Path to video device */
    mode;               /* Display compatibility mode */
```

`dc_setcmp()` sets the display compatibility mode. Compatibility mode is provided by the video hardware to allow 525 line images to be decoded on 625 line hardware and vice-versa. The actual function of the call is described in full in the UCM Specification in the Green Book.

This function returns the x and y values which indicate how the video hardware will adjust the starting point of the image on the display. For example, on a PAL display, the image will be moved down 20 lines to center it vertically on the display.

These values are returned in a packed longword. The high order 16 bits contain the x offset and the low 16 bits contain the y offset. These values can be used to pass to the pointer driver to adjust the pointer origin appropriately.

The `mode` is defined as follows:

- `0` Image and Display are the same resolution
- `1` Image and Display are different resolutions

`-1` is returned on error and `errno` is set.

-----

### `dc_ssig()` Send Signal on Video Interrupt

*UCM File Manager C Binding*

```c
dc_ssig(path, sigcode, count)
int path,               /* Path to video device */
    sigcode,            /* Signal to send on video interrupt */
    count;              /* number of video interrupts */
```

dc_ssig() requests the video driver to send the signal specified by `sigcode`, after `count` video interrupts. If `count` is 0, the signal will be sent on the next video interrupt.

`-1` is returned on error and `errno` is set.

-----

### `dc_wrfct()` Write Field Control Table

*UCM File Manager C Binding*

```c
dc_wrfct(path, fctid, startins, count, buff)
int path,               /* Path to video device */
    fctid,              /* FCT id */
    startins,           /* FCT instruction at which to begin */
    count,              /* Number of instructions to write */
    *buff;              /* Pointer to buffer containing instructions */
```

`dc_wrfct()` writes count instructions from the buffer pointed to by `buff` into an FCT starting at the instruction specified by `startins`.

`-1` is returned on error and `errno` is set.

-----

### `dc_wrfi()` Write Field Control Table Instruction

*UCM File Manager C Binding*

```c
dc_wrfi(path, fctid, number, instruct)
int path,               /* Path to video device */
    fctid,              /* FCT id */
    number,             /* FCT instruction number to write */
    instruct;           /* Instruction to write */
```

`dc_wrf1()` writes a single instruction to the specified FCT. The instruction number to be written is specified by `number` and the actual data to write is passed in `instruct`.

`-1` is returned on error and `errno` is set.

-----

### `dc_wrlct()` Write Line Control Table

*UCM File Manager C Binding*

```c
dc_rdlct(path, lctid, lineno, colno, numlines, numcols, buff)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    lineno,             /* LCT line number to begin wrote */
    colno,              /* LCT column number to begin write */
    numlines,           /* Physical number of lines to write */
    numcols,            /* Physical number of columns to write */
    *buff;              /* Pointer to buffer of data to write */
```

`dc_wrlct()` writes data from the buffer pointed to by buff to a two dimensional array in the LCT specified by `lctid`. The first element of the array to be written is specified by `lineno` and `colno`. The number of columns per line to be written is specified by `numcols`. The number of lines to be written is specified by `numlines`.

`-1` is returned on error and `errno` is set.

-----

### `dc_wrli()` Write Line Control Table Instruction

*UCM File Manager C Binding*

```c
dc_rdli(path, lctid, lineno, insnum)
int path,               /* Path to video device */
    lctid,              /* LCT id */
    lineno,             /* LCT line number to write */
    insnum,             /* LCT column number to write */
    instruct;           /* Instruction to write */
```

`dc_wrli()` writes the instruction passed as `instruct` to the specified location in the LCT specifed by `lctid`. The instruction is written at the position specified by `lineno` and `insnum`.

`-1` is returned on error and `errno` is set.

-----

### `dm_close()` Close Drawmap

*UCM File Manager C Binding*

```c
dm_close(path, dmid)
int path,               /* Path to video device */
    dmid;               /* Drawmap id */
```

`dm_close()` deallocates the memory associated with the drawmap specified by `dmid`.

`-1` is returned on error and `errno` is set.

-----

### `dm_cncl()` Conceal Error in Drawmap

*UCM File Manager C Binding*

```c
dm_cncl(path, dmid, data)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
char *data;             /* Pointer to error bit string */
```

`dm_cncl()` performs the video driver's error concealment function for the drawmap specified by `dmid`. The data pointed to by data is a bit string containing one bit for each line of the drawmap. If a bit is set, the corresponding line in the drawmap is in error. The erroneous line is then replaced in the drawmap with the previous line. If the first line is in error, it is replaced by the first non-erroneous line. If all lines are in error, this function will return an `E$IllPrm` error.

`-1` is returned on error and `errno` is set.

-----

### `dm_copy()` Copy Drawmap to Drawmap

*UCM File Manager C Binding*

```c
dm_copy(path, srcdmid, dstdmid, dstx, dsty, srcx, srcy, width, height)
int path,               /* Path to video device */
    srcdmid,            /* ID of source drawmap */
    dstdmid,            /* ID of destinastion drawmap */
    dstx, dsty,         /* Coords for copy in destination drawmap */
    srcx, srcy,         /* Coords to copy in source drawmap */
    width,              /* Width of region to copy*/
    height;             /* Height of region to copy*/
```

`dm_copy()` copies a rectangular portion of the drawmap specified by `srcdmid`, to the drawmap specified by `dstdmid`.

The coordinates `srcx`, `srcy` specify the upper left corner of the rectangle in the source drawmap (i.e the area to be copied). The coordinates `dstx`, `dsty` specify the upper left corner of the copied rectangle in the destination drawmap.

The size of the rectangle is given by `width` and `height`.

The two drawmaps must have the same data type.

`-1` is returned on error and `errno` is set.

-----

### `dm_creat()` Create Drawmap

*UCM File Manager C Binding*

```c
int dm_creat(path, plane, type, width, height, length, qlength)
int path,               /* Path to video device */
    plane,              /* Plane number */
    type,               /* New drawmap type */
    width,              /* Width of drawmap */
    height,             /* Height of drawmap */
    length,             /* Length of drawmap */
    qlength;            /* Length of QHY drawmap */
```

`dm_creat()` creates a drawmap of the type specified by type. type is a 16 bit word. The bits have the following meaning:

```plaintext
bits 0-3    Drawmap type:
             0-2                 Reserved
               3  D_CLUT4        CLUT4 mode
               4  D_CLUT7        CLUT7 mode
               5  D_CLUT8        CLUT8 mode
               6  D_RL3          Run-Length 3 mode
               7  D_RL7          Run-Length 7 mode
               8  D_DYUV         Delta-YUV mode
               9  D_RGB          RGB555 mode
              10                 Reserved
              11  D_QHY          DYUV+QHY mode
           12-15                 Reserved

bits 4-5    Resolution:
               0  RES_NORMAL     Normal Resolution
               1  RES_DOUBLE     Double Resolution
               2                 Reserved
               3  RES_HIGH       High Resolution

bits 6-14   Reserved

bit 15      LPT Format:
               0                  Even/odd lines interleaved
               1 LN_SEPARATE      Even/odd lines separated
```

If the specified `type` is Run Length (`RL3` or `RL7`), the `width` and `height` parameters are ignored when calculating the size of the drawmap.

If `type` is `QHY`, the `size` and `length` parameters are used to create a standard `DYUV` drawmap in one plane. The parameter `qlength` specifies the
length of a `QHY` drawmap block in the other plane. Otherwise, `qlength` is
ignored.

If `type` is `RGB555`, the driver calculates the size and then allocates two blocks of memory of that size.

The plane number is specified by `plane` in order to allocate the drawmap in the proper memory.

The size of the drawmap data area depends upon the two-dimensional size specified by `width` and `height` or on the `length` parameter. The driver calculates the amount of memory required by the specified size (`width * height`). This amount is compared with `length`; the greater of the two values is used for the size of the data area.

If the application creates a drawmap with even and odd lines separated, the driver initializes the Line Pointer Table (LPT) entry for the first line to byte 0 and the LPT entry for the second line at byte `size / 2`.

`dm_creat()` returns the drawmap id of the new drawmap. If `dm_creat()` gets an error, it will return `-1` and `errno` will be set.

-----

### `dm_create()` Create Drawmap

*UCM File Manager C Binding*

```c
DrawmapDesc *dm_create(path, plane, type, width, height, length, qlength)
int path,               /* Path to video device */
    plane,              /* Plane number */
    type,               /* New drawmap type */
    width,              /* Width of drawmap */
    height,             /* Height of drawmap */
    length,             /* Length of drawmap */
    qlength;            /* Length of QHY drawmap */
```

`dm_create()` creates a drawmap of the type specified by type. type is a 16 bit word. The bits have the following meaning:

```plaintext
bits 0-3    Drawmap type:
             0-2                 Reserved
               3  D_CLUT4        CLUT4 mode
               4  D_CLUT7        CLUT7 mode
               5  D_CLUT8        CLUT8 mode
               6  D_RL3          Run-Length 3 mode
               7  D_RL7          Run-Length 7 mode
               8  D_DYUV         Delta-YUV mode
               9  D_RGB          RGB555 mode
              10                 Reserved
              11  D_QHY          DYUV+QHY mode
           12-15                 Reserved

bits 4-5    Resolution:
               0  RES_NORMAL     Normal Resolution
               1  RES_OOUBLE     Double Resolution
               2                 Reserved
               3  RES_HIGH       High Resolution

bits 6-14   Reserved

bit 15      LPT Format:
               0                  Even/odd lines interleaved
               1 LN_SEPARATE      Even/odd lines separated
```

If the specified `type` is Run Length (`RL3` or `RL7`), the `width` and `height` parameters are ignored when calculating the size of the drawmap.

If `type` is `QHY`, the `size` and `length` parameters are used to create a standard `DYUV` drawmap in one plane. The parameter `qlength` specifies the
length of a `QHY` drawmap block in the other plane. Otherwise, `qlength` is
ignored.

If `type` is `RGB555`, the driver calculates the size and then allocates two blocks of memory of that size.

The plane number is specified by `plane` in order to allocate the drawmap in the proper memory.

The size of the drawmap data area depends upon the two-dimensional size specified by `width` and `height` or on the `length` parameter. The driver calculates the amount of memory required by the specified size (`width * height`). This amount is compared with `length`; the greater of the two values is used for the size of the data area.

If the application creates a drawmap with even and odd lines separated, the driver initializes the Line Pointer Table (LPT) entry for the first line to byte 0 and the LPT entry for the second line at byte `size / 2`.

`dm_create()` returns a pointer to the drawmap descriptor. This descriptor is returned for informational purposes only. The application should not change any values in the drawmap descriptor. There are drawing parameter functions to change any fields which need to be altered.

If `dm_create()` gets an error, it will return a NULL pointer and `errno`
will be set.

-----

### `dm_dup()` Duplicate Drawmap

*UCM File Manager C Binding*

```c
dm_dup(path, dmid)
int path,               /* Path to video device */
    dmid;               /* Drawmap id */
```

`dm_dup()` creates a new drawmap descriptor and copies the contents of the descriptor of the specified drawmap into the new descriptor. This call returns the drawmap id of the new descriptor.

This call does not create a new drawmap.

-----

### `dm_dupe()` Duplicate Drawmap

*UCM File Manager C Binding*

```c
DrawmapDesc *dm_dupe(path, dmid)
int path,               /* Path to video device */
    dmid;               /* Drawmap id */
```

`dm_dupe()` creates a new drawmap descriptor and copies the contents of the descriptor of the specified drawmap into the new descriptor. This call returns the drawmap id of the new descriptor.

This call does not create a new drawmap.

-----

### `dm_exch()` Exchange Data Between Drawmaps

*UCM File Manager C Binding*

```c
dm_exch(path, srcdmid, dstdmid, dstx, dsty, srcx, srcy, width, height)
int path,               /* Path to video device */
    srcdmid,            /* ID of source drawmap */
    dstdmid,            /* ID of destination drawmap */
    dstx, dsty,         /* Coords for exchange in destination drawmap */
    srcx, srcy,         /* Coords for exchange in source drawmap */
    width,              /* Width of region to copy*/
    height;             /* Height of region to copy */
```

`dm_exch()` exchanges a rectangular portion of the drawmap specified by `srcdmid`, with a rectanguar portion of the drawmap specified by `dstdmid`.

The coordinates `srcx`, `srcy` specify the upper left comer of the rectangle in the source drawmap. The coordinates `dstx`, `dsty` specify the upper left corner of the rectangle in the destination drawmap.

The size of both rectangles is specified by `width` and `height`.

The two drawmaps must have the same data type.

`-1` is returned on error and `errno` is set with the appropriate error code.

-----

### `dm_irwr()` Irregular Write to Drawmap

*UCM File Manager C Binding*

```c
dm_irwr(path, dmid, ystart, lincnt, data, transarr)
int  path,              /* Path to video device */
     dmid,              /* Drawmap id */
     ystart,            /* Vertical coordinate to start write */
     lincnt;            /* Number of lines to write */
char *data;             /* Pointer to data to write */
int  *transarr;         /* Pointer to transfer data array */
```

`dm_irwr()` is used for irregular updates of a drawmap. The drawmap to be updated is specified by `dmid`. The update begins at the vertical position specified by `ystart` and continues for `lincnt` lines. `transarr` is a transfer specifier array describing how the update is to proceed. The
values in the array are as follows:

```plaintext
Bits  0-15:  Number of Pixels to transfer
     16-31:  Horizontal Coordinate
```

This allows a different beginning horizontal coordinate to be specified for each line to be updated.

The data to be written is pointed to by `data`.

-----

### `dm_org()` Set Drawmap Origin

*UCM File Manager C Binding*

```c
dm_org(path, dmid, xcoord, ycoord)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    xcoord,             /* New x origin */
    ycoord;             /* New y origin */
```

`dm_org()` changes the origin position of the drawmap specified by `dmid` in the coordinate space. When the drawmap is created, the default origin is the upper left comer of the drawmap. The new origin is specified as an offset from the upper left comer of the drawmap by `xcoord`, `ycoord`.

This function returns zero. If an error occurs, `-1` is returned with the appropriate error code set in `errno`.

-----

### `dm_rdpix()` Read Pixel

*UCM File Manager C Binding*

```c
dm_rdpix(path, dmid, x, y)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    x, y;               /* Coordinates to be read */
```

`dm_rdpix()` returns the pixel data read from the coordinates `x`, `y` in the drawmap specified by `dmid`.

If an error occurs, `-1` is returned with the appropriate error code set in `errno`.

-----

### `dm_read()` Read from Drawmap

*UCM File Manager C Binding*

```c
dm_read(path, dmid, x, y, width, height, pixarr)
int  path,              /* Path to video device */
     dmid,              /* Drawmap id */
     x, y,              /* Upper left coordinate of rectangle */
     width,             /* Width of rectangle */
     height;            /* Height of rectangle */
char *pixarr;           /* Pointer to pixel array */
```

`dm_read()` reads a rectangular array of pixels from the drawmap specified by `dmid` to a user buffer pointed to by `pixarr`. The data is read starting at the specified coordinate `x`, `y`. The size of the rectangle to be read is given by `width` and `height`. UCM assumes that each line in the output array is an integral number of longwords in length.

**NOTE**: If the area to be read extends beyond the borders of the drawmap, an error will be returned.

On error, `-1` is returned and `errno` is set.

-----

### `dm_tcopy()` Copy Drawmap to Drawmap with Transparency

*UCM File Manager C Binding*

```c
dm_tcopy(path, srcdmid, dstdmid, dstx, dsty, srcx, srcy, width, height, tcolor)
int path,               /* Path to video device */
    srcdmid,            /* ID of source drawmap */
    dstdmid,            /* ID of destinastion drawmap */
    dstx, dsty,         /* Coords for copy in destination drawmap */
    srcx, srcy,         /* Coords to copy in source drawmap */
    width,              /* Width of region to copy*/
    height,             /* Height of region to copy*/
    tcolor;             /* Transparency color */
```

`dm_tcopy()` copies a :ectangular portion of the drawmap specified by `srcdmid`, to a rectangular portion of the drawmap specified by `dstdmid`.

As it copies the drawmap, `dm_tcopy()` checks each source pixel for equivalence to the transparency color, `tcolor`. If the values match the pixel is not copied to the destination drawmap.

The coordinates `srcx`, `srcy` specify the upper left corner of the rectangle in the source drawmap (i.e the area to be copied). The coordinates `dstx`, `dsty` specify the upper left corner of the rectantangle in the destination drawmap.

The size of the rectangle is given by `width` and `height`.

The two drawmaps must have the same data type.

`-1` is returned on error and `errno` is set with the appropriate error code.

**NOTE**: This call is only usable on CLUT and RGB type drawmaps.

-----

### `dm_texch()` Exchange Between Drawmaps with Transparency

*UCM File Manager C Binding*

```c
dm_texch(path, srcdmid, dstdmid, dstx, dsty, srcx, srcy, width, height, tcolor)
int path,               /* Path to video device */
    srcdmid,            /* ID of source drawmap */
    dstdmid,            /* ID of destination drawmap */
    dstx, dsty,         /* Coords for exchange in destination drawmap */
    srcx, srcy,         /* Coords for exchange in source drawmap */
    width,              /* Width of region to copy*/
    height,             /* Height of region to copy*/
    tcolor;             /* Transparency color */
```

`dm_texch()` exchanges a rectangular portion of the drawmap specified by `srcdmid`, with a rectanguar portion of the drawmap specified by `dstdmid`.

As `dm_texch()` exchanges the drawmaps, it checks each source pixel for equivalence to the transparency color, given by `tcolor`. If the values
match the pixel is not exchanged with the destination drawmap pixel.

The coordinates `srcx`, `srcy` specify the upper left comer of the rectangle in the source drawmap. The coordinates `dstx`, `dsty` specify the upper left comer of the rectangle in the destination drawmap.

The size of both rectangles is specified by `width` and `height`.

The two drawmaps must have the same data type.

`-1` is returned on error and `errno` is set with the appropriate error code.

**NOTE**: This call is only usable on CLUT and RGB type drawmaps.

-----

### `dm_write()` Write to Drawmap

*UCM File Manager C Binding*

```c
dm_write(path, dmid, x, y, width, height, pixarr)
int  path,              /* Path to video device */
     dmid,              /* Drawmap id */
     x, y,              /* Upper left coordinate of rectangle */
     width,             /* Width of rectangle */
     height;            /* Height of rectangle */
char *pixarr;           /* Pointer to pixel array */
```

`dm_write()` writes a rectangular array of pixels (`pixarr`) to the drawmap specified by `dmid`. The writing begins at the coordinates `x`, `y`. The size of the rectangle is specified by `width` and `height`. UCM assumes that the
rows in the pixel array are an integral number of longwords in length.

**NOTE**: One row of the pixel array corresponds to one line in the drawmap.

UCM also assumes that all the bits for a pixel are in a contiguous string
of bits in the array.

`-1` is returned on error and `errno` is set.

-----

### `dm_wrpix()` Write Pixel

*UCM File Manager C Binding*

```c
dm_wrplx(path, dmid, x, y, pixdata)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    x, y,               /* Coordinates to be read */
    pixdata;            /* Data to be written */
```

`dm_wrpix()` writes the pixel data `pixdata` at the specified coordinates `x`, `y` in the drawmap `dmid`.

`-1` is returned on error and `errno` is set.

-----

### `dp_afnt()` Activate Font

*UCM File Manager C Binding*

```c
dp_afnt(path, dmid, fntident, afn)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    fntident,           /* Font id */
    afn;                /* Active font number */
```

`dp_afnt()` activates the font specified by `fntident` to be used for graphics text on the drawmap specified by `dmid`. The font will be associated with the active font number specified by `afn`. There may be up to four active fonts (0-3) at any given time.

`-1` is returned on error and `errno` is set.

-----

### `dp_clip()` Set Clipping Region

*UCM File Manager C Binding*

```c
dp_clip(path, dmid, regid)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    regid;              /* Region id */
```

`dp_clip()` specifies the clipping region for drawing in the drawmap specified by `dmid`. The clipping region is specified by `regid`. No drawing will be allowed outside this region.

If `regid` is zero, the clipping region is reset to the default clipping region
of the drawmap.

`-1` is returned on error and `errno` is set.

-----

### `dp_dfnt()` Deactivate Font

*UCM File Manager C Binding*

```c
dp_dfnt(path, dmid, fntident, afn)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    afn;                /* Active font number */
```

`dp_dfnt()` deactives the font specified by `afn` for the drawmap specified by `dmid`.

`-1` is returned on error and `errno` is set.

-----

### `dp_gfnt()` Get Font

*UCM File Manager C Binding*

```c
int dp_gfnt(path, fntname)
int  path,              /* Path to video device */
char *fntname;          /* pointer to font module name */
```

`dp_gfnt()` gets the specified font module ready for use by an application wishing to do text output. If the module is already in memory, `dp_gfnt()` will link to it. If not, it will be loaded from disk storage.

**NOTE**: The font module must be in the application's execution directory for it to be loaded by `F$Load`.

The font identifier is returned upon a successful link/load. `-1` is returned on error and `errno` is set.

-----

### `dp_paln()` Set Pattern Alignment

*UCM File Manager C Binding*

```c
dp_paln(path, dmid, x, y)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    x, y;               /* Pattern alignment coordinates */
```

`dp_paln()` aligns the pixel specified by the coordinates `x`, `y` in the drawing pattern with the pixel at coordinate 0,0 in the drawmap specified by `dmid`.

`-1` is returned on error and `errno` is set.

-----

### `dp_pnsz()` Set Pen Size

*UCM File Manager C Binding*

```c
dp_pnsz(path, dmid, penwidth, penheight)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    penwidth,           /* Width of lines when drawn */
    penheight;          /* Height of lines when drawn */
```

`dp_pnsz()` sets the and width of the lines that are used to draw all lines and unfilled shapes in the drawmap `dmid`. `penwidth` and `penheight` are specified in pixel units. The default width and height is one pixel.

`-1` is returned on error and `errno` is set.

-----

### `dp_pstyl()` Set Pen Style

*UCM File Manager C Binding*

```c
dp_pstyl(path, dmid, bitstring, pixcnt)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    bitstring,          /* Size/pattern of dash */
    pixcnt;             /* Pixel counter */
```

`dp_pstyl()` is used to set the dash styling for lines drawn in the drawmap specified by `dmid`. `bitstring` is a bit string used to indicate the size of dash. `pixcnt` represents the number of `pixels` each bit in the string represents. For example:

```plaintext
bitstring:      1101110110111011
pixcnt:         2
line drawn as:  11110011111100111100111111001111
```

This penstyle is used when the `OP_DSDSH` bit is set in the drawing opcode.

`-1` is returned on error and `errno` is set.

-----

### `dp_ptn()` Set Drawing Pattern

*UCM File Manager C Binding*

```c
#include <ucm.h>

dp_ptn(path, dmid, type, pattern)
int  path,              /* Path to video device */
     dmid,              /* Drawmap id */
     type;              /* Pattern type */
char *pattern:          /* Pointer to bitmap pattern */
```

`dp_ptn()` sets the drawing pattern for the drawmap specified by `dmid`. The pattern is a 16x16 pixel bitmap. It is used for most of the drawing operations.

The `type` parameter describes the type of pattern data to be used. The
pattern type definitions can be found in the header file, `ucm.h` and below:

```plaintext
type: 0  PTN_1BIT
      1  PTN_2BIT
      2  PTN_4BIT
      3  PTN_CLUT4
      4  PTN_CLUT7
      5  PTN_CLUT8
      9  PTN_RGB
```

If `type` equals 0, 1 or 2, the pixel data is expanded using a set of color registers to obtain the actual data that will be drawn in the drawmap. The pattern's pixel value selects the color register from which the drawing data is obtained.

If `type` equals 3, 4, 5 or 9, actual pixel data is pointed to by `pattern`.

`-1` is returned on error and `errno` is set.

-----

### `dp_rfnt()` Release Font

*UCM File Manager C Binding*

```c
dp_rfnt(path, fntident)
int path,               /* Path to video device */
    fntident;           /* Font id */
```

`dp_rfnt()` deallocates the memory used by the specified font module `fntident` and unlinks the font module.

`-1` is returned on error and `errno` is set.

-----

### `dp_scmm()` Set Character Mapping Method

*UCM File Manager C Binding*

```c
dp_scmm(path, dmid, mapmethod)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    mapmethod;          /* Mapping method for char codes */
```

`dp_scmm()` determines how character codes will be interpreted by `dr_text()` and `dr_jtxt()` for the drawmap specified by `dmid`. The possible methods specified by `mapmethod` are as follows:

- `0` eight bit
- `1` seven/fifteen bit
- `2` sixteen bit

-----

### `dp_scr()` Set Color Register

*UCM File Manager C Binding*

```c
dp_scr(path, dmid, regno, colordata)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    regno               /* Color register number */
    colordata;          /* Color data */
```

`dp_scr()` sets the color data to be used for pixel expansion. The register specified by `regno` will be set to the given data in `colordata`. There are sixteen registers available. The format of the color data given depends upon the type of the drawmap specified by `dmid`. For example, for CLUT drawmaps, the color register would contain the CLUT number. For RGB drawmaps, the color register would contain the actual RGB value.

`-1` is returned on error and `errno` is set.

-----

### `dp_tcol()` Set Transparent Color

*UCM File Manager C Binding*

```c
dp_tcol (path, dmid, tcolor)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    tcolor;             /* Transparency color */
```

`dp_tcol()` sets the color that will be used in the transparency check by the drawing functions. The format of the color data is dependant upon the type of the drawmap specified by `dmid`.

`-1` is returned on error and `errno` is set.

-----

### `dr_bfil()` Fill a Bounded Area

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_bfi1(path, dmid, opcode, x, y, bcolor)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    x, y,               /* Any coords in area to be filled */
    bcolor;             /* Color bordering area to be filled */
```

`dr_bfil()` fills an area in the drawmap specified by `dmid`. The area is defined as containing the specified `x`, `y` coordinates and being bound by pixels that match the color specified by `bcolor`. The area is filled with either a pattern specified by `dp_ptn()` or a solid color specified by bits 5-8 of `opcode`. Bit 13 of `opcode` (see below) specifies which type of fill to use.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_carc()` Draw an Arc

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_carc(path, dmid, opcode, sx, sy, ex, ey, cx, cy, radius)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Start arc coordinates */
    ex, ey,             /* End arc coordinates */
    cx, cy,             /* Center of arc coordinates */
    radius;             /* Radius of arc */
```

`dr_carc()` draws an arc with the specified `radius` from the specified center coordinates `cx`, `cy`. The arc's beginning and ending coordinates are determined by `sx`, `sy` and `ex`, `ey` respectively. The beginning point is on the line defined by `sx`, `sy` and `cx`, `cy`; the ending point on the line defined by `ex`, `ey` and `cx`, `cy`. The pixel at the end coordinate is not drawn.

**NOTE**: All arcs are drawn clockwise.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_circ()` Draw a Circle

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_circ(path, dmid, opcode, x, y, radius)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    x, y,               /* Center coordinates of circle */
    radius;             /* Radius of circle */
```

`dr_circ()` draws a circle in the drawmap specified by `dmid`. The circle's center point is the given coordinates `x`, `y` and the radius is radius.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_copy()` Copy Data from Drawmap to Drawmap

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_copy(path, opcode, srcdm, dstdm, dstx, dsty, srcx, srcy, width, height)
int path,               /* Path to video device */
    opcode,             /* Drawing operation codes */
    srcdm,              /* Source drawmap id */
    dstdm,              /* Destination drawmap id */
    dstx, dsty,         /* Destination coords for copy */
    srcx, srcy,         /* Source coords to begin copy */
    width,              /* Width of area to copy */
    height;             /* Height of area to copy */
```

`dr_copy()` copies a rectangular portion of the source drawmap specified by `srcdm`. The destination drawmap, specified by `dstdm`, may be another drawmap or the same drawmap.

The size of the rectangle copied is defined by `width` and `height`. The origin of the rectangle to be copied is specified by `srcx`, `srcy` in the source drawmap. The origin of the placement in the destination drawmap is specified by `dstx`, `dsty`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_cwdg()` Draw a Circular Wedge

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_cwdg(path, dmid, opcode, sx, sy, ex, ey, cx, cy, radius)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Start arc coordinates */
    ex, ey,             /* End arc coordinates */
    cx, cy,             /* Center of arc coordinates */
    radius;             /* Radius of arc */
```

`dr_cwdg()` draws an circular wedge in the drawmap specified by `dmid`. The wedge is defined by the specified `radius` from the center coordinates `cx`, `cy` The arc's beginning and ending coordinates are determined by `sx`, `sy` and `ex`, `ey` respectively. The beginning point is on the line defined by `sx`, `sy` and `cx`, `cy`; the ending point on the line defined by `ex`, `ey` and `cx`, `cy`.

**NOTE**: Wedges are always drawn in a clockwise direction.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_dot()` Draw a Dot

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_dot(path, dmid, opcode, x, y)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    x, y;               /* Coordinates for drawing */
```

`dr_dot()` draws a dot at the specified coordinates `x`, `y` in the drawmap `dmid` using the current pen size and drawing pattern.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_drgn()` Draw a Region

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_drgn(path, dmid, opcode, regid)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    regid;              /* Region id */
```

`dr_drgn()` draws a previously defined region identified by `regid`. The region is drawn in the drawmap specified by `dmid`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_earc()` Draw an Elliptical Arc

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_earc(path, dmid, opcode, sx, sy, ex, ey, cx, cy, xrad, yrad)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Start arc coordinates */
    ex, ey,             /* End arc coordinates */
    cx, cy,             /* Center of arc coordinates */
    xrad,               /* Horizontal radius of arc */
    yrad;               /* Vertical radius of arc */
```

`dr_earc()` draws an elliptical arc with a center point of `cx`, `cy` and the radii along the horizontal and vertical axes given by `xrad`, `yrad`. The arc's beginning and ending coordinates are determined by `sx`, `sy` and `ex`, `ey` respectively. The beginning point is on the line defined by `sx`, `sy` and `cx`, `cy`; the ending point on the line defined by `ex`, `ey` and `cx`, `cy`. The pixel at the end coordinate is not drawn.

**NOTE**: All arcs are drawn clockwise.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_elps()` Draw an Ellipse

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_elps(path, dmid, opcode, x, y, xrad, yrad)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    x, y,               /* Center coordinates of circle */
    xrad,               /* Horizontal radius of arc */
    yrad;               /* Vertical radius of arc */
```

`dr_elps()` draws an ellipse in the drawmap specified by `dmid`. The center point of the elipse is specified by `x`, `y`. The size and shape of the ellipse is specified by the radii on the horizontal and vertical axes given by `xrad`, `yrad`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_erect()` Draw an Elliptical Cornered Rectangle

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_erect(path, dmid, opcode, sx, sy, ex, ey, rx, ry)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Upper left corner coordinates */
    ex, ey,             /* Lower right corner coordinates */
    rx, ry;             /* Horizontal/vertical radii */
```

`dr_erect()` draws a rectangle which has its corners replaced by ninety degree arcs of an ellipse whose radii are specified by `rx`, `ry`. The rectangle must be minimally twice as wide as the horizontal radius, `rx`, and twice as high as the vertical radius, `ry`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

### `dr_ewdg()` Draw an Elliptical Wedge

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_ewdg(path, dmid, opcode, sx, sy, ex, ey, cx, cy, xrad, yrad)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Start coordinates of arc */
    ex, ey,             /* End coordinates of arc */
    cx, cy,             /* Center coordinates of arc */
    xrad, yrad;         /* Horizontal/vertical radii of arc */
```

`dr_ewdg()` draws an elliptical wedge in the drawmap specified by `dmid`. The ellipse is defined by the center point coordinates `cx`, `cy` and the radii along the horizontal and vertical axes given by `xrad`, `yrad`. The wedge's beginning and ending coordinates are determined by `sx`, `sy` and `ex`, `ey` respectively. The beginning point is on the line defined by `sx`, `sy` and `cx`, `cy`; the ending point on the line defined by `ex`, `ey` and `cx`, `cy`.

**NOTE**: All wedges are drawn in a clockwise direction.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_ffil()` Flood Fill

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_ffil(path, dmid, opcode, x, y)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    x, y;               /* Any coordinates in area to be filled */
```

`dr_ffil()` changes the pixel at the specified `x`, `y` coordinate and all adjacent pixels of the same color using either the pattern specified by `dp_ptn()` or a solid color. Bit 13 of `opcode` specifies which type of fill to use. Bits 5-8 of opcode determine the fill color for a solid fill.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_jtxt()` Output Justified Text

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_jtxt(path, dmid, opcode, x, y, len, str, maxchrs)
int  path,              /* Path to video device */
     dmid,              /* Drawmap id */
     opcode,            /* Drawing operation codes */
     x, y,              /* Coordinates at which to begin write */
     len;               /* Justification length */
char *str;              /* Pointer to string to write */
int  maxchrs;           /* maximum number of chars to write */
```

`dr_jtxt()` writes the text pointed to by `str` in the current font. The text baseline will start at the specified `x`, `y` coordinate. All control characters are ignored. The text will be justified to the length specified by `len`.

If the text will not fit in the specified length, it will be truncated. The text that is printed will be justified. The text will be printed with a maximum of `maxchrs` characters or until a NULL byte is reached.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_line()` Draw a Line

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_line(path, dmid, opcode, sx, sy, ex, ey)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Starting coordinates for line */
    ex, ey;             /* Ending coordinates for line */
```

`dr_line()` draws a line from the coordinates `sx`, `sy` to `ex`, `ey` in the drawmap specified by `dmid`. The pixel at the end coordinate is not drawn.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_pgon()` Draw a Polygon

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_pgon(path, dmid, opcode, novert, vertarr)
int   path,             /* Path to video device */
      dmid,             /* Drawmap id */
      opcode,           /* Drawing operation codes */
      novert,           /* Number of vertices to use */
short *vertarr;         /* Pointer to array of vertices */
```

`dr_pgon()` draws a polygon in the drawmap `dmid` from the coordinates given in the vertices array pointed to by `vertarr`. Each two elements of this array specify an x, y coordinate pair for a segment of the polygon. The number of coordinate pairs to use are specified by `novert`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_plin()` Draw a Polyline

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_plin(path, dmid, opcode, novert, vertarr)
int   path,             /* Path to video device */
      dmid,             /* Drawmap id */
      opcode,           /* Drawing operation codes */
      novert,           /* Number of vertices to use */
short *vertarr;         /* Pointer to array of vertices */
```

`dr_plin()` draws a polyline in the drawmap `dmid` from the coordinates given in the vertices array pointed to by `vertarr`. Each two elements of this array specify an x, y coordinate pair for a segment of the polyline. The number of coordinate pairs to use are specified by `novert`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_rect()` Draw a Rectangle

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_rect(path, dmid, opcode, sx, sy, ex, ey)
int path,               /* Path to video device */
    dmid,               /* Drawmap id */
    opcode,             /* Drawing operation codes */
    sx, sy,             /* Upper left corner coordinates */
    ex, ey;             /* Lower right corner coordinates */
```

`dr_rect()` draws a rectangle in the drawmap specified by `dmid`. Its opposite corners are specified by `sx`, `sy` and `ex`, `ey`.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `dr_text()` Output Graphics Text

*UCM File Manager C Binding*

```c
#include <ucm.h>

dr_jtxt(path, dmid, opcode, x, y, len, str, maxchrs)
int  path,              /* Path to video device */
     dmid,              /* Drawmap id */
     opcode,            /* Drawing operation codes */
     x, y;              /* Coordinates at which to begin write */
char *str;              /* Pointer to string to write */
int  maxchrs;           /* maximum number of chars to write */
```

`dr_text()` writes the text pointed to by `str` in the current font. The text baseline will start at the specified `x`, `y` coordinate. All control characters are ignored. The text will be printed with a maximum of `maxchrs` characters or up until a NULL byte is reached.

`opcode` sets the operational specifications for the drawing functions (i.e. whether clipping will be performed, how shapes will be filled and how pixels will be combined). Definitions for setting `opcode` are defined in the `DEFS` file `ucm.h` and in the drawing functions overview in the first chapter.

`-1` is returned on error and `errno` is set.

-----

### `gc_blnk()` Set Graphics Cursor Blink Rate

*UCM File Manager C Binding*

```c
gc_blnk(path , blnkconst)
int path,               /* Path to video device */
    blnkconst;          /* Blink rate */
```

`gc_blnk()` sets the blink rate for the graphics cursor.

The blink rate is determined by the "blink on" time period (BON) and the "blink off" time period (BOFF). For 60Hz displays, the time periods are specified in 200 ms units. For 50 Hz displays, the time periods are given in 240 ms units.

During the BOFF period, the cursor may be displayed in the complementary color or not displayed at all (transparent).

The individual bit fields for `blnkconst` are defined as:

```plaintext
bits   0-2  BOFF period (zero = never off)
         3  reserved
       4-6  BON period (zero not permitted)
      7-14  reserved
        15  complement blink field
            (0=transparent; 1=complementary color)
     16-31  reserved
```

`-1` is returned on error and `errno` is set.

-----

### `gc_col()` Set Graphics Cursor Color

*UCM File Manager C Binding*

```c
gc_co1(path, color)
int path,               /* Path to video device */
    color;              /* Color for cursor display */
```

`gc_col()` sets the color used for displaying the graphics cursor. The individual bits of the color parameter are defined as follows:

```plaintext
bits   0-7  Blue
      8-15  Green
     16-23  Red
     24-31  Intensity
```

**NOTE**: In the base case CD-I configuration, only the high order bit of each of these bytes is checked. Consequently, there are eight colors at low intensity (bit 31 = 0) and eight colors at high intensity (bit 31 = 1 ).

`-1` is returned on error and `errno` is set.

-----

### `gc_hide()` Hide Graphics Cursor

*UCM File Manager C Binding*

```c
gc_hide(path)
int path;               /* Path to video device */
```

`gc_hide()` causes the graphics cursor to disappear from the screen.

`-1` is returned on error and `errno` is set.

-----

### `gc_org()` Set Graphics Cursor Origin

*UCM File Manager C Binding*

```c
gc_org(path, x, y)
int path,               /* Path to video device */
    x, y;               /* New origin of the graphic cursor */
```

`gc_org()` sets the graphics cursor origin. The new origin is specified by the coordinates `x`, `y`. These coordinates are relative to the default origin at the upper left corner of the full screen display.

`-1` is returned on error and `errno` is set.

-----

### `gc_pos()` Position Graphics Cursor

*UCM File Manager C Binding*

```c
gc_pos(path, x, y)
int path,               /* Path to video device */
    x, y;               /* Coordinates of the cursor hit-point */
```

`gc_pos()` changes the position of the graphics cursor on the display. The `x`, `y` coordinates specify where the hit-point of the cursor will be positioned.

`-1` is returned on error and `errno` is set.

-----

### `gc_ptn()` Set Graphics Cursor Pattern

*UCM File Manager C Binding*

```c
gc_ptn(path, hitx, hity, width, height, res, pattern)
int  path,              /* Path to video device */
     hitx, hity,        /* Cursor hit-point coords in pattern */
     width,             /* Width of bitmap pattern */
     height,            /* Height of bitmap pattern */
     res;               /* Resolution of cursor display */
char *pattern:          /* Pointer to bitmap data*/
```

`gc_ptn()` sets the pattern used for the graphics cursor. The cursor shape is set by the bitmap pointed to by `pattern`. The hitpoint of the cursor is defined by the coordinates `hitx` and `hity` within the bitmap pattern. `width` and `height` specify the size of the bitmap used.

The pixels set to one in the bitmap will be displayed in the graphics cursor color set by the `gc_col()` function.

`res` specifies the resolution in which to display the cursor:

- `0` Normal
- `1` Double
- `2` High

**NOTE**: This resolution is independent of the resolution of any
drawmap.

`-1` is returned on error and `errno` is set.

-----

### `gc_show()` Show Graphics Cursor

*UCM File Manager C Binding*

```c
gc_show(path)
int path;               /* Path to video device */
```

`gc_show()` causes the graphics cursor to be displayed on the screen.

`-1` is returned on error and `errno` is set.

-----

### `gs_eof()` Test for End-Of-File

*CDFM File Manager C Binding*

```c
int gs_eof(path)
int path;               /* Path number of file to test */
```

`gs_eof()` is to determine if the current file pointer is pointing at the end of the file indicated by `path`. If so, 1 is returned; if not at the end of file, this call returns 0.

-----

### `gs_gcdfd()` Return File Descriptor

*CDFM File Manager C Binding*

```c
gs_gcdfd(path, buffer, count)
int  path;              /* Path number of open file */
char *buffer;           /* Pointer to buffer to hold file descriptor data*/
int count;              /* Number of bytes to read from file descriptor */
```

`gs_gcdfd()` reads count bytes of the file descriptor of the file indicated by the path number `path`. The file descriptor data is returned in the buffer pointed to by `buffer`. The CDFM file descriptor is defined in the header file `cdi.h`.

This function returns 0, if successful. If the path is invalid, this call returns `-1`with the appropriate error code in the variable `errno`.

-----

### `gs_opt()` Get options of path descriptor

*CDFM File Manager C Binding*

```c
gs_opt(path, buffer)
int  path;              /* Path number of open file */
char *buffer;           /* Pointer to buffer to options data*/
```

This call copies the options section of the path descriptor of the specified `path` number into the buffer pointed to by `buffer`. The buffer must be large enough to read the options section in its entirety. The section is 128 bytes.

The options section of the path descriptor contains information which may be of use to the application. Certain fields in the path descriptor may be changed by the application. To do so, first read the option section (i.e. use this call), change the individual fields in the buffer and write the new options section using the `ss_opt()` function.

If the path is invalid, this call returns `-1`with the appropriate error code in the variable `errno`.

-----

### `gs_path()` Get Full Pathlist of Open File

*CDFM File Manager C Binding*

```c
gs_path(path, buffer)
int  path;              /* Path number of open file */
char *buffer;           /* Pointer to buffer to hold pathlist */
```

`gs_path()` returns a full pathlist of the open file specified by the path number `path`. The pathlist is returned in the buffer pointed to by `buffer`.

If the path is invalid, this call returns `-1`with the appropriate error code in the variable `errno`.

-----

### `gs_pos()` Return File Position

*CDFM File Manager C Binding*

```c
int gs_pos(path)
int path;               /* Path number of file to be read */
```

`gs_pos()` returns the position of the next byte to be read in the file specified by the path number `path`.

-----

### `gs_size()` Return File Size

*CDFM File Manager C Binding*

```c
int gs_size(path)
int path;               /* Path number of file to be read */
```

This function returns the total size (in bytes) of the file specified by the
path number `path`.

**CAVEAT**: The file size is read from the file descriptor. This figure is determined by multiplying the number of sectors used by this file by 2048 and subtracting any unused portion of the last sector of the file. This does not take into account that form 2 sectors actually contain 2324 bytes of data and CDDA tracks also have sectors of a different size.

-----

### `kb_rdy()` Check for Data Ready

*UCM File Manager C Binding*

```c
int kb_rdy(path)
int path;               /* Path to keyboard device */
```

`kb_rdy()` returns the number of characters that are in the keyboard buffer. If no data is available or the path is invalid, a `-1` is returned and the appropriate error code is placed in `errno`.

-----

### `kb_read()` Read Keyboard Event

*UCM File Manager C Binding*

```c
kb_read(path, mode, code, type, spkeys)
int   path,             /* Path to keyboard device */
      mode;             /* Sleep or ready mode */
short *code;            /* Pointer to character code */
char  *type;            /* Pointer to data type */
```

`kb_read()` returns keyboard event information. The event information is queued and returned individually.

`mode` is a bitmapped int which specifies the function of this call:

- bit 0 set: wait for character (if none is available)
- bit 1 set: remove event from queue

The character code is pointed to by `code`.

`type` points to a byte specifying one of the following events:

- bit 0 set: the key has just been pressed
- bit 1 set: the key is being auto-repeated
- bit 2 set: the key has just been released

If no data is available at the time kb_read() is invoked and the mode parameter is set to zero, the caller is put to sleep until data is ready.

> The parameter `spkeys` is only listed in the function declaration, nowhere else in the function description.

-----

### `kb_rel()` Release Keyboard Device

*UCM File Manager C Binding*

```c
kb_rel(path)
int path;               /* Path to keyboard device */
```

`kb_rel()` releases the keyboard from a `kb_ssig()` request that was made.

-----

### `kb_repeat()` Set Keyboard Latency and Repeat Times

*UCM File Manager C Binding*

```c
kb_repeat(path, latency, repeat)
int path,               /* Path to keyboard device */
    latency,            /* Latency time */
    repeat;             /* Repeat time */
```

`kb_repeat()` sets the keyboard auto repeat latency time and the keyboard repeat time. These times are specified in ticks. If the high order bit is set, the times are in 1/256 second units.

The auto repeat latency time specifies how long to wait after the initial key depression before repeating the key. The keyboard repeat time specifies the amount of time to wait before repeating a key, when a key is depressed.

If the latency time is 0, auto repeats will not be generated. If the repeat time is 0, only one auto repeat event will be generated.

-----

### `kb_ssig()` Send Signal on Data Ready

*UCM File Manager C Binding*

```c
kb_ssig(path, signal)
int path,               /* Path to keyboard device */
    signal;             /* User defined signal code */
```

`kb_ssig()` sets up a user defined signal code to be sent when data is ready from the keyboard.

-----

### `kb_stat()` Get Keyboard Status

*UCM File Manager C Binding*

```c
kb_stat (path. code. spkeys)
int   path;             /* Path to keyboard device */
short *code,            /* Pointer to character code returned */
      *spkeys;          /* Pointer to special key status */
```

`kb_stat()` returns the current state of the keyboard including the state of the special keys.

`code` will contain the value of the character, if any key is currently depressed. If more than one key is depressed, `code` will contain the value of the last key pressed.

`spkeys` returns the state of the special keys. the special keys status is indicated in the following manner:

- bit 0: Shift key
- bit 1: Caps key
- bit 2: Supershift key
- bit 3: Control key

If a bit is set, the corresponding key is depressed.

> Supershift is also referred to as the ALT key.

-----

### `make_module()` Create a Module

*OS-9 System Call Binding*

```c
#include <memory.h>
#include <module.h>

mod_exec *make_module(name, size, attr, perm, type, memtype)
char  *name:            /* Pointer to name of module */
int   size;             /* Size of module */
short attr,             /* Attributes/revision of module */
      perm,             /* Module permissions*/
      type;             /* Type/language of module*/
int   memtype;          /* Memory type in which to make module*/
```

`make_module()` creates a executable memory module with the specified name and attributes in the specified memory. `mod_exec` is defined in `module.h`.

`size` specifies the desired memory size of the module in byte units. The size value does not include the module header and CRC bytes. `size` is the amount of memory available for actual use. The memory in the data module is initially cleared to zeroes.

`attr` is the attribute/revision word, perm is the access permission word and `type` specifies the type/language of the module.

`memtype` indicates the specific type of memory in which to load the
module.

There are three types of memory that may be specified:

- `SYSRAM` System RAM memory
- `VIDEO1` Video memory for plane A
- `VIDEO2` Video memory for plane B

`make_module()` returns a pointer to the beginning of the module header. If the data module can not be created, a value of `-1` is returned and the appropriate error code is placed in the global variable `errno`.

-----

### `modcload()` Load module into colored memory

*OS-9 System Call Binding*

```c
#include <memory.h>
#include <module.h>

mod_exec *modcload(path, mode, memtype)
int path,               /* Path to module*/
    mode,               /* Access mode with which to open the file * /
    memtype;            /* Type code of specific memory type*/
```

`modcload()` will search the module directory for a module with the same name as that pointed to by `path` and link to it. If the module is not in the module directory, `path` is considered a pathlist and all modules in the specified file are loaded. A link is made to the first module loaded from the file and a pointer to it is returned.

`mode` is the mode with which to open the file to load. If any access mode is acceptable, zero can be specified for mode.

`memtype` indicates the specific type of memory in which to load the module.

There are three types of memory that may be specified:

- `SYSRAM` System RAM memory
- `VIDEO1` Video memory for plane A
- `VIDEO2` Video memory for plane B

If the load is successful, `modcload()` returns a pointer to the module. If the load fails, `-1` is returned and the appropriate error code is placed in the global variable `errno`.

-----

### `pt_coord()` Get Pointer Coordinates

*UCM File Manager C Binding*

```c
pt_coord(path, btnstate, x, y)
int path,               /* Path to Pointing device */
    *btnstate,          /* Pointer to button state data */
    *x, *y:             /* Pointer to current coordinates of pointer */

```

`pt_coord()` returns the button state, `btnstate`, and the current coordinates `x`, `y` of the pointer with respect to the origin of the display.

`btnstat` uses the following bit value meanings:

```plaintext
Bit 0:     1 = Button 1 depressed
    1:     1 = Button 2 depressed
    2-14:  Reserved
    15:    1 = The pointer is active
           This is especially important for graphics tablet and touch screen devices.
```

`-1` is returned on error and `errno` is set.

> The documentation refers to `path` as the video device path, however in order to support two controllers, the `DT_PTR` path must be specified.
> `x` and `y` are specified in the range 0-1023

-----

### `pt_org()` Set Pointer Origin

*UCM File Manager C Binding*

```c
pt_org(path, x, y)
int path,               /* Path to Pointing device */
    x, y;               /* New pointer origin coordinates */
```

`pt_org()` sets the pointer origin to the given `x`, `y` coordinates. The new origin is set relative to the upper left corner of the full screen display.

`-1` is returned on error and `errno` is set.

> The documentation refers to `path` as the video device path, however in order to support two controllers, the `DT_PTR` path must be specified.
> `x` and `y` are specified in the range 0-1023

-----

### `pt_pos()` Position the Pointer

*UCM File Manager C Binding*

```c
pt_pos(path, x, y)
int path,               /* Path to Pointing device */
    x, y;               /* New coordinates for pointer */
```

`pt_pos()` positions the pointing device to the specified coordinates `x`, `y`.

`-1` is returned on error and `errno` is set.

> The documentation refers to `path` as the video device path, however in order to support two controllers, the `DT_PTR` path must be specified.
> `x` and `y` are specified in the range 0-1023

-----

### `pt_rel()` Release Pointer Device

*UCM File Manager C Binding*

```c
pt_rel(path)
int path;               /* Path to Pointing device */
```

`pt_rel()` releases the pointer from a `pt_ssig()` request.

`-1` is returned on error and `errno` is set.

> The documentation refers to `path` as the video device path, however in order to support two controllers, the `DT_PTR` path must be specified.

-----

### `pt_ssig()` Send Signal on Pointer Change

*UCM File Manager C Binding*

```c
pt_ssig(path, sigcode)
int path,               /* Path to Pointing device */
    sigcode;            /* User defined signal to be sent */
```

`pt_ssig()` sets up a signal to be sent to the process when the pointer is moved or one of its triggers changes state.

`-1` is returned on error and `errno` is set.

> The documentation refers to `path` as the video device path, however in order to support two controllers, the `DT_PTR` path must be specified.
> Note that no signal is sent when the controller stops moving.

-----

### `rg_creat()` Create Region

*UCM File Manager C Binding*

```c
#include <ucm.h>

rg_creat(path, regtype, {type dependant})
int path,               /* Path to video device */
    regtype;            /* Region type code */
```

`rg_creat()` is used to create a region of the specified type (`regtype`) and return the region identifier for that region. The region type codes are defined in `ucm.h` and given below:

0) `D_RECT`  Rectangular region
1) `D_ERECT` Elliptical cornered rectangular region
2) `D_POLY`  Polygon region
3) `D_CIRC`  Circular region
4) `D_CWDG`  Circular wedge region
5) `D_ELPS`  Elliptical region
6) `D_EWDG`  Elliptical wedge region
7) `D_RGN`   Pre-defined region
8) `D_BFIL`  Region defined by border fill area
9) `D_FFIL`  Region defined by flood fill area

`regtype` determines the type dependant parameters that rg_creat() expects. The following are the parameters for each region type.

In all cases the region identifier is returned, if the region is  uccessfully created. `-1` is returned on error and `errno` is set.

#### D_RECT: Rectangular region

```c
rg_creat(path, D_RECT, sx, sy, ex, ey)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    sx, sy, ex, ey;     /* Opposite corners of the region */
```

#### D_ERECT: Elliptical cornered rectangular region

```c
rg_creat(path, D_ERECT, sx, sy, ex, ey)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    sx, sy, ex, ey,     /* Opposite corners of the region */
    xrad, yrad;         /* Radii of the elliptical corners */
```

#### D_POLY: Polygon region

```c
rg_creat(path, D_POLY, sx, sy, ex, ey)
int   path,             /* Path to video device */
      regtype,          /* Region type code */
      novertices;       /* Number of vertices in polygon */
short *vertarr;         /* Array of vertices (x, y pairs) */
```

#### D_CIRC: Circular region

```c
rg_creat(path, D_CIRC, x, y, radius)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    x, y,               /* Center of circle */
    radius;             /* Radius */
```

#### D_CWDG: Circular wedge region

```c
rg_creat(path, D_CWDG, sx, sy, ex, ey, cx, cy, rad)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    sx, sy, ex, ey,     /* Start, end points of wedge arc */
    cx, cy,             /* Center of circle for arc */
    rad;                /* Radius of wedge arc */
```

**Note**: The start and/or end point of the arc does not have to be specified within the region. These points, with the center point, merely define the edges of the wedge.

#### D_ELPS: Elliptical region

```c
rg_creat(path, D_ELPS, x, y, xrad, yrad)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    x, y,               /* Center of ellipse */
    xrad, yrad;         /* Radii of the ellipse */
```

#### D_EWDG: Circular wedge region

```c
rg_creat(path, D_EWDG, sx, sy, ex, ey, cx, cy, xrad, yrad)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    sx, sy, ex, ey,     /* Start, end points of wedge */
    cx, cy,             /* Center point of the ellipse arc */
    xrad, yrad;         /* x and y radii */
```

**Note**: The start and/or end point of the arc does not have to be specified within the region. These points, with the center point, merely define the edges of the wedge.

#### D_RGN: Pre-defined region

```c
rg_creat(path, D_RGN, data)
int path,               /* Path to video device */
    regtype;            /* Region type code */
RegionDesc *data        /* Region definition */
```

This call will create a pre-defined region. When a region is created a transition table is created by the driver. By using the transition data from a previous call, time is saved in creating a new region of the same type.

#### D_BFIL: Region defined by border fill area

```c
rg_creat(path, D_CIRC, dmid, x, y, col)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    dmid,               /* Drawmap id */
    x, y,               /* Point within fill region */
    col;                /* Color of region border */
```

This call creates a region from all pixels adjacent to the specified `x`, `y` coordinates that do not have the specified border color (`col`).

#### D_FFIL: Region defined by flood fill area

```c
rg_creat(path, D_CIRC, dmid, x, y)
int path,               /* Path to video device */
    regtype,            /* Region type code */
    dmid,               /* Drawmap id */
    x, y;               /* Fill point */
```

This call creates a region from all pixels adjacent to the specified `x`, `y` coordinates that have the same color as the specified pixel.

-----

### `rg_del()` Delete Region

*UCM File Manager C Binding*

```c
rg_del(path, regid)
int path,               /* Path to video device */
    regid;              /* Region id */
```

`rg_del()` deallocates the memory associated with the region specified by `regid`.

`-1` is returned on error and `errno` is set.

-----

### `rg_diff()` Region Difference

*UCM File Manager C Binding*

```c
int rg_diff(path, reg1, reg2)
int path,               /* Path to video device */
    reg1,               /* ID of region l */
    reg2;               /* ID of region 2 */
```

`rg_diff()` creates a new region defined by the difference of the two regions identified by `reg1` and `reg2`.

The new region's identifier will be returned on successful creation of the region.

`-1` is returned on error and `errno` is set.

-----

### `rg_isect()` Region Intersection

*UCM File Manager C Binding*

```c
int rg_isect(path, reg1, reg2)
int path,               /* Path to video device */
    reg1,               /* ID of region l */
    reg2;               /* ID of region 2 */
```

`rg_isect()` creates a new region defined by the intersection of the two regions identified by `reg1` and `reg2`.
new region

The new region's identifier will be returned on successful creation of the region.

`-1` is returned on error and `errno` is set.

-----

### `rg_move()` Move Region

*UCM File Manager C Binding*

```c
int rg_move(path, regid, x, y)
int path,               /* Path to video device */
    regid,              /* Region id */
    x, y;               /* New upper left coordinates for region */
```

`rg_move()` changes the position of the upper-left corner of the region's boundary rectangle to the specified coordinates `x`, `y`. The `x`, `y` coordinates are relative to the top left corner of the display. The boundary rectangle is defined as the rectangle that completely encloses the region.

`-1` is returned on error and `errno` is set.

-----

### `rg_union()` Region Union

*UCM File Manager C Binding*

```c
int rg_union(path, reg1, reg2)
int path,               /* Path to video device */
    reg1,               /* ID of region l */
    reg2;               /* ID of region 2 */
```

`rg_union()` creates a new region defined by the union of the two regions identified by `reg1` and `reg2`.

The new region's identifier will be returned on successful creation of the region.

`-1` is returned on error and `errno` is set.

-----

### `rg_xor()` Region Exclusive Or

*UCM File Manager C Binding*

```c
int rg_xor(path, reg1, reg2)
int path,               /* Path to video device */
    reg1,               /* ID of region l */
    reg2;               /* ID of region 2 */
```

`rg_xor()` creates a new region which is the exclusive-or of the two regions identified by `reg1` and `reg2`.

The new region's identifier will be returned on successful creation of the region.

`-1` is returned on error and `errno` is set.

-----

### `sc_atten()` Set volume, balance and position of sound

*CDFM File Manager C Binding*

```c
sc_atten(path, attenvals)
int path,             /* Path to the audio processor */
    attenvals;        /* Attenuation values */
```

`sc_atten()` sets the overall volume of the sound, the balance of stereo sound and the position of sound in the stereo space to the audio processor specified by `path`. `path` must previously have been opened.

The attenuation values `attenvals` are specified as follows:

- Bits 0-7: Right input and Left output attenuation
- Bits 8-15: Right input and Right output attenuation
- Bits 16-23: Left input and Right output attenuation
- Bits 24-31: Left input and Left output attenuation

-----

### `sd_loop()` Set Soundmap Loopback Points

*CDFM File Manager C Binding*

```c
sd_loop(path, smid, stgrp, endgrp, lpcnt)
int path,             /* Path to the audio processor */
    smid,             /* Soundmap id */
    stgrp,            /* Starting sound group number */
    endgrp,           /* Ending sound group number */
    lpcnt;            /* Number of loops to execute */
```

`sd_loop()` changes or sets the loopback points for a soundmap. The top of the loop is the sound group number given by `stgrp`. The bottom of the loop is given by `endgrp`.

If the loopback points are set, when output to the audio processor reaches the end of the soundmap, data between the start and end points will be repeatedly output to the audio processor the number of times specified by `lpcnt`.

The soundgroups specified by `stgrp` and `endgrp` must be on sector boundaries.

`-1` is returned on error and `errno` is set.

-----

### `sd_mmix()` Mix Monaural to Stereo

*CDFM File Manager C Binding*

```c
sd_mmix(path, smid1, smid2, smid3, stsct)
int path,             /* Path to the audio processor */
    smid1,            /* ID of first mono soundmap to be mixed */
    smid2,            /* ID of second mono soundmap to be mixed */
    smid3,            /* ID of destination stereo soundmap */
    stsct;            /* Sector to begin mixing in smid1 */
```

`sd_mmix()` takes data from two monaural soundmaps of the same type specified by `smid1` and `smid2` and mixes them into the soundmap specified by `smid3`.

`stsct` specifies the sector number from which to begin mixing in the
first soundmap (`smid1`).

If the soundmaps are different sizes, the number of sectors mixed is determined by the smallest of the three soundmaps.

If successful, this call returns 0. `-1` is returned on error and `errno` is set.

-----

### `sd_smix()` Mix Stereo to Stereo

*CDFM File Manager C Binding*

```c
sd_smix(path, smid1, smid2, smid3, stsct, chansel)
int path,               /* Path to audio processor */
    smid1,              /* ID of stereo soundmap to mix */
    smid2,              /* ID of stereo soundmap to mix */
    smid3,              /* ID of destination stereo soundmap */
    stsct,              /* Sector to begin mixing in smid1 */
    chansel;            /* Stereo channels to mix */
```

`sd_smix()` mixes the data of one channel from the soundmap specified by `smid1` with a channel of the soundmap specified by `smid2`. The resulting sound data is placed in the soundmap specified by `smid3`.

`stsct` specifies the sector number from which to begin mixing in the first soundmap (`smid1`).

If the soundmaps are different sizes, the number of sectors mixed is determined by the smallest of the three soundmaps.

The `chansel` parameter specifies the channel mixing as defined in `cdfm.h` and given below:

- `0` `MIX_L1L2` Left channels of both soundmaps
- `1` `MIX_L1R2` Left channel of soundmap 1 and Right channel of soundmap 2
- `2` `MIX_R1L2` Right channel of soundmap 1 and Left channel of soundmap 2
- `3` `MIX_R1R2` Right channels of both soundmaps

If succesful, this call returns 0. `-1` is returned on error and `errno` is set.

-----

### `sm_close()` Close Soundmap

*CDFM File Manager C Binding*

```c
sm_close(path, smid)
int path,               /* Path to audio processor */
    smid;               /* Soundmap id */
```

`sm_close()` deallocates the memory associated with the specified soundmap.

`-1` is returned on error and `errno` is set.

-----

### `sm_cncl()` Conceal Error in Soundmap

*CDFM File Manager C Binding*

```c
sm_cncl(path, smid, errdata)
int path,               /* Path to audio processor */
    smid;               /* Soundmap id */
PL_ERR *errdata;        /* Error data structure */
```

`sm_cncl()` performs the error concealment function of the audio driver. `errdata` is a pointer to a `PL_Err` structure pointed to be `PCL_Err` in the Play Control List. If the bit is set, the corresponding group is in error and needs to be corrected.

`-1` is returned on error and `errno` is set.

-----

### `sm_creat()` Create Soundmap

*CDFM File Manager C Binding*

```c
#include <cdfm.h>

sm_creat(path, datatype, numgroups, mapaddr)
int   path,             /* Path to audio processor */
      datatype,         /* Soundmap type */
      numgroups;        /* Number of soundgroups in soundmap */
char **mapaddr;         /* address of soundmap data area */
```

`sm_creat()` creates a soundmap. Its size is determined by `numgroups`. `numgroups` specifies the number of sound groups which the soundmap will contain. This number will be divided by 18 (i.e. 18 sound groups per sector) and then rounded up so that the soundmap contains an integral number of sectors.

The soundmap type is specified by `datatype`. The valid data types are defined in `cdfm.h` and are given below:

- `0` `D_AMONO` Level A Mono
- `1` `D_ASTRO` Level A Stereo
- `2` `D_BMONO` Level B Mono
- `3` `D_BSTRO` Level B Stereo
- `4` `D_CMONO` Level C Mono
- `5` `D_CSTRO` Level C Stereo

The actual address of the soundmap data area is placed in the location specified by `mapaddr`. This function returns the soundmap identifier.

`-1` is returned on error and `errno` is set.

-----

### `sm_info()` Get Soundmap Information

*CDFM File Manager C Binding*

```c
#include <cdfm.h>
SoundmapDesc *sm_info(path, smid)
int path,               /* Path to audio processor */
    smid;               /* Soundmap id */
```

`sm_info()` returns a pointer to the soundmap descriptor for the soundmap specified by `smid`. `SoundmapDesc` is defined in `cdfm.h` and below:

```c
typedef struct _SoundmapDesc {
  short smd_id,      /* ID of soundmap */
        smd_stloop,  /* Start soundgroup for looping */
        smd_enloop,  /* End soundgroup for looping */
        smd_lpcnt,   /* Actual loop count */
        smd_lcntr;   /* Counter for looping */
    int smd_smaddr,  /* Address to soundmap */
        smd_smsize,  /* Size of soundmap */
        smd_curaddr, /* Current address of playing soundmap */
        smd_asyblk;  /* Pointer to ASY block*/
  short smd_nogrps,  /* Number of sound groups in soundmap */
        smd_grpnum;  /* Group which started sector's play */
   char smd_coding,  /* Coding information for soundmap */
        smd_resl:    /* Reserved */
   long smd_lpaddr,  /* Start address of loop */
        smd_res2,    /* Reserved */
        smd_res3,
        smd_res4;
} SoundmapDesc:
```

If an error occurs, a NULL pointer is returned and `errno` is set.

-----

### `sm_off()` Turn Off Audio Processor

*CDFM File Manager C Binding*

```c
sm_off(path)
int path;               /* Path to audio processor */
```

`sm_off()` causes output to the audio processor to be stopped. If a soundmap is playing, no signal is sent to the process.

`-1` is returned on error and `errno` is set.

-----

### `sm_out()` Output Soundmap

*CDFM File Manager C Binding*

```c
#include <cdfm.h>

sm_out(path, smid, statusblk)
int path,               /* Path to audio processor */
    smid;               /* Soundmap id */
STAT_BLK *statusblk;    /* Signal code sent when done */

```

`sm_out()` causes the data in the specified soundmap to be output. The data is output to the audio processor specified by `path`. `path` must previously have been opened.

This function is asynchronous, that is, it returns before the data is actually "played". When the last sector of the soundmap is sent to the audio processor, the signal code specified in `statusblk` is sent to the process which did the `sm_out()`. `STAT_BLK` is defined in `cdfm.h` and below:

```c
typedef struct stat_blk {
    short asy_stat: /* Current status of the operation */
    short asy_sig;  /* Signal to be sent on termination */
} STAT_BLK;
```

`asy_stat` contains the current status of this operation. The system updates this field. It should not be altered by the application. The bit values are defined as follows:

- `Bit     0` If set, operation is finished
- `Bits 1-14` Reserved (must be zero)
- `Bit    15` If set, an error has occurred.

`asy_sig` contains the signal number to be sent when the operation is finished or an error occurs. It is initialized by the application before the operation is begun and may be changed by the application during the operation. If this field is set to zero, no signal is sent when the operation is finished or an error occurs. The error code is placed in this field when an error occurs during the operation.

**NOTE**: If a `sm_out()` call is made before the last sector is played from a previous `sm_out()` call, the first soundmap will be discontinued and the signal will not be sent.

`-1` is returned on error and `errno` is set.

-----

### `sm_stat()` Return Current Play Status

*CDFM File Manager C Binding*

```c
#include <cdfm.h>

sm_stat(path, audiostat)
int path,               /* Path to audio processor */
AudioStatus *audiostat; /* Audio status block */
```

`sm_stat()` returns the status of the current audio play in the structure
pointed to by `audiostat`. `AudioStatus` is defined in `cdfm.h` and below:

```c
typedef struct AudioStatus {
  short sms_sctnum,  /* Current sector playing */
        sms_totsec,  /* Total number of sectors */
        sms_lpcnt,   /* Loop count left */
        sms_res;     /* Reserved */
} AudioStatus;
```

`-1` is returned on error and `errno` is set.

-----

### `srqcmem()` Allocate colored memory

*OS-9 System Call C Binding*

```c
#include <memory.h>

char *srqcmem(bytecount, memtype)
int bytecount,          /* Size of memory to allocate */
    memtype:            /* Type of memory to allocate */
```

`srqcmem()` is a direct hook into the OS-9 `F$SRqCMem` system call. The specified `bytecount` will be rounded to a system-defined block size. A size of `0xffffffff` will obtain the largest contiguous block of free memory in the system. `memtype` indicates the specific type of memory to allocate.

There are three types of memory that may be specified:

- `SYSRAM` System RAM memory
- `VIDEO1` Video memory for plane A
- `VIDEO2` Video memory for plane B

If successful, a pointer to the memory granted is returned. The pointer
returned will always begin on an even byte boundary. If the request was not granted, the function returns the value `-1` and the appropriate error code is left in the global variable `errno`.

-----

### `ss_abort()` Abort Operation in Progress

*CDFM File Manager C Binding*

```c
ss_abort(path)
int path;
```

`ss_abort()` is used to abort the asynchronous operation currently in progress specified by the path number `path`.

Only a process with a super-user ID (0,0) or with the user ID of the process which started the operation to be aborted may execute this call.

If successful, this function causes an `E$Abort` error to be returned to any asynchronous operations. If not, the function returns the value `-1` and the appropriate error code is left in the global variable `errno`.

-----

### `ss_cchan()` Change data and audio channels

*CDFM File Manager C Binding*

```c
ss_cchan(path)
int path;
```

`ss_cchan()` is used to notify the CD-I driver to attempt to change the data and audio channels that are being used with the current play in progress. Before making this call the application must change the values in the play control block for the desired channel changes (i.e., the `PCB_Chan` and `PCB_AChan` fields). This allows the program to change the channel being played at any time; not just when an end-of-record or a trigger bit in a sector is reached.

If successful, this function returns `0`; if not, the function returns the value `-1` and the appropriate error code is left in the global variable `errno`.

-----

### `ss_cdda()` Play CD Digital Audio

*CDFM File Manager C Binding*

```c
ss_cdda(path, nsecs, statb)
int path,          /* Path number of open file to play */
    nsecs;         /* Number of sectors */
STAT_BLK *statb;   /* Status block */
```

`ss_cdda()` is used to play CD-DA sectors on the disc. The play starts from the current file position and continues until one of the following occurs: an error occurs, `nsecs` sectors are played or the end of file is reached.

**NOTE**: When playing CD-DA sectors, the resolution of the current file position can only be guaranteed to 75 sectors (i.e. one second).

This function is an asynchronous operation and therefore uses a status block to keep the application informed of the play status. The status block is pointed to by `statb`. `STAT_BLK` is defined in `cdfm.h` and below:

```c
typedef struct stat_blk {
  short asy_stat; /* Current status of the operation */
  short asy_sig;  /* Signal to be sent on termination */
} STAT_BLK;
```

`asy_stat` contains the current status of the play operation. The system updates this field. It should not be altered by the application. The bit values are defined as follows:

- `Bit     0` If set, play is finished
- `Bits 1-14` Reserved (must be zero)
- `Bit    15` If set, an error has occurred.

`asy_sig` contains the signal number to be sent when the operation is finished or an error occurs. It is initialized by the application before the operation is begun and may be changed by the application during the operation. If this field is set to zero, no signal is sent when the play is finished or an error occurs. The error code is placed in this field when an error occurs during the operation.

If successful, `ss_cdda()` returns 0; otherwise `-1` is returned and the appropriate error code is left in the global variable `errno`.

-----

### `ss_cont()` Continue Play on Disc Drive

*CDFM File Manager C Binding*

```c
ss_cont(path)
int path;
```

This call causes the disc drive to continue playing after it has been given a pause command.

If successful, `ss_cont()` returns 0; otherwise `-1` is returned and the appropriate error code is left in the global variable `errno`.

-----

### `ss_disable()` Disable Hardware Control Buttons on Player

*CDFM File Manager C Binding*

```c
ss_disable(path)
int path;
```

`ss_disable()` disables any eject or disc drive related user controls on the
CD-I player.

If successful, `ss_disable()` returns 0; otherwise `-1` is returned and the
appropriate error code is left in the global variable `errno`.

-----

### `ss_eject()` Issue Door Open Command to Disc Drive

*CDFM File Manager C Binding*

```c
ss_eject(path)
int path;
```

`ss_eject()` is used to open the CD disc drive door. If a play is in progress, the equivalent of an `ss_abort()` is executed before the disc is ejected.

Only a process with a super-user ID (0,0) may execute this call.

If successful, `ss_eject()` returns 0; otherwise `-1` is returned and the appropriate error code is left in the global variable `errno`.

-----

### `ss_enable()` Enable Hardware Control Buttons on Player

*CDFM File Manager C Binding*

```c
ss_enable(path)
int path;
```

`ss_enable()` enables any eject or disc drive related user controls on the CD-I player.

If successful, `ss_enable()` returns 0; otherwise `-1` is returned and the appropriate error code is left in the global variable `errno`.

-----

### `ss_mount()` Mount Disc by Disc Number

*CDFM File Manager C Binding*

```c
ss_mount(path, diskn)
int path,
    diskn;              /* Disc number */
```

`ss_mount()` is used to change the default disc in use in multi-disc players. This call also causes the drive door to close if possible and the disk to be spun up.

The disc is specified by the disc number `diskn`. `diskn` may range between 0 and 65535. On a single disc player, the number used is 0.

If successful, `ss_mount()` returns 0; otherwise `-1` is returned and the appropriate error code is left in the global variable `errno`.

-----

### `ss_opt()` Set Path Descriptor Options

*CDFM File Manager C Binding*

```c
ss_opt(path, buffer)
int path;               /* Path number of open file */
char *buffer;           /* Pointer to buffer containing options data */
```

`ss_opt()` writes a 128 byte buffer of information to the options section of the path descriptor of the open file indicated by the path number `path`. The buffer to be written is pointed to by `buffer`.

The options section of the path descriptor contains information which may be of use to the application. Certain fields in the path descriptor may be changed by the application. To do so, first read the option section (i.e. using `gs_opt()`), change the individual fields in the buffer and write the new options section using this call.

Currently, only `PD_CNUM` may he changed by the application. `PD_CNUM` selects a different channel mask for the next synchronous CD disc I/O operation.

If the path is invalid, this call returns `-1`with the appropriate error code
in the variable `errno`.

-----

### `ss_pause()` Pause the Disc Drive

*CDFM File Manager C Binding*

```c
ss_pause(path)
int path;
```

`ss_pause()` is used to pause the CD disc drive.

Only a process with a super-user ID (0,0) or with the user ID of the process which started the operation to be paused may execute this call.

If successful, `ss_pause()` returns 0; otherwise `-1` is returned and the appropriate error code is left in the global variable `errno`.

-----

### `ss_play()` Play Real Time Records

*CDFM File Manager C Binding*

```c
#include <cdfm.h>

ss_play(path, pcb)
int path;
PCB *pcb:               /* Play control block */
```

`ss_play()` is used to play real time records off the disc. It can be used to play all types of Mode 2 sectors. The play operation is controlled by the
Play Control Block (PCB) pointed to by `pcb`.

A PCB is built by the application. It contains information such as which data types and channel masks to select, upon which events to send a signal and which signal to send, which records to play, the total number of records to play, and pointers to lists of Play Control Lists (PCL's) for each data type. PCB's and PCL's are defined in `cdfm.h` and below:

```c
typedef struct {
  short  PCB_Stat,       /* Current status of the play */
         PCB_Sig;        /* Signal to be sent on termination */
  int    PCB_Rec,        /* Maximum num of records to play */
         PCB_Chan;       /* Channel selection mask */
  short  PCB_AChan;      /* Audio to memory selection mask */
  PCL    **PCB_Video,    /* Pointer to array of PCL pointers for video sectors */
         **PCB_Audio,    /* Pointer to array of PCL pointers for audio sectors */
         **PCB_Data;     /* Pointer to array of PCL pointers for data sectors */
} PCB;

typedef struct __pcl {
  char   PCL_Ctrl,       /* Control byte */
         PCL_Rsvl,       /* Reserved */
         PCL_Smode,      /* Submode byte */
         PCL_Type;       /* Coding information byte */
  short  PCL_Sig;        /* Signal to be sent on buffer full */
  struct __pcl *PCL_Nxt; /* Pointer to next PCL */
  char   *PCL_Buf;       /* Pointer to buffer */
  int    PCL_BufSz;      /* Size of buffer */
  PL_ERR *PCL_Err;       /* Pointer to error buffer */
  short  PCL_Rsv2;       /* Reserved */
  int    PCL_Cnt;        /* Current offset in buffer */
} PCL;
```

The values with in the PCB, with the exception of `PCB_Stat`, can be changed at any time by the application. Each change takes effect when the next sector is transferred to memory or following receipt of an End Of Record or Trigger bit.

The play control structures control the functionality of play. The play is started using data pointed to by the file position pointer. The file position pointer is updated to the new file position after the play terminates.

Play is an asynchronous operation. This means that when the play is executing, the application continues to run.

-----

### `ss_raw()` Read Raw Sectors

*CDFM File Manager C Binding*

```c
ss_raw(path, buffer, count)
int  path,              /* Path number of open file*/
     count;             /* Number of bytes to read (2340 increments) */
char *buffer;           /* Pointer to buffer to hold data read */
```

`ss_raw()` reads raw sector data beginning with byte 12 through byte 2351 of the current sector. This read will include the header, subheader, data and ECC/EDC (if any). `ss_raw()` can read Mode 1, Mode 2, Form 1 and Form 2 sectors.

If more than one sector is to be read, the number of bytes to be read are specified by `count`. `count` must be a multiple of 2340.

The returned data is placed in the buffer pointed to be `buffer`. For each sector read 2340 bytes is returned.

When this call is executed the current file position must be on a sector boundary. The file position pointer is incremented logically by 2048 for each sector read. This will keep the file position on a sector boundary.

-----

### `ss_readtoc()` Read Table of Contents

*CDFM File Manager C Binding*

```c
#include <cdfm.h>

ss_readtoc(path, toc, numbytes)
int path,               /* Path number of open file to be read */
    *toc,               /* Pointer to buffer to hold data read */
    numbytes;           /* Number of bytes of table of contents to read */
```

`ss_readtoc()` reads the specified number of bytes (`numbytes`) of the table of contents of the file indicated by the path number `path`. The data read is placed in the buffer pointed to by toc.

In order to read the entire table of contents, the toc buffer must be 510 bytes. The table of contents structure is defined in `cdi.h` and below:

```c
typedef struct {
   char toc_ctrl0, /* Control word 0 */
        toc_a0,    /* $a0 byte */
        toc_stno,  /* Last data track number + 1 (CD-I) */
                   /* First READ track number (CD-I w/RED tracks) */
                   /* Start track number (CD-DA & CD-ROM) */
        toc_type,  /* $10 for CD-I or CD-I w/RED tracks */
                   /* $00 for CD-DA or CD-ROM */
        toc_pblk0, /* $00 */
        toc_ctrl1, /* Control word 1 */
        toc_a1,    /* $a1 byte */
        toc_ltno,  /* Last track num, for CD-I last track no + 1 */
        toc_psec1, /* $00 */
        toc_pblk1, /* $00 */
        toc_ctrl2, /* Control word 2 */
        toc_a2,    /* $a2 byte */
        toc_lmin,  /* Start minute of LEAD-OUT area */
        toc_lsec,  /* Start second of LEAD-OUT area */
        toc_lblk;  /* Start block of LEAD-OUT area*/
 struct _toc_packet toc_packets[99]: /* Track packs for 99 tracks */
} _toc;
```

If an error occurs `-1` is returned and `errno` is set.

-----

### `ss_seek()` Move File Position Pointer

*CDFM File Manager C Binding*

```c
#include <cdfm.h>

ss_seek(path, pos, statb)
int path,               /* Path number of open file */
    pos;                /* New file position */
STAT_BLK *statb;        /* Status block */
```

`ss_seek()` causes the file position pointer to be moved to the specified byte position `pos`. This call also causes the disc drive to physically seek to the beginning of the sector containing the specified byte. This call uses a logical sector size of 2048 bytes in order to compute the new file position.

This function is an asynchronous operation and therefore uses a status block to keep the application informed of the status of the seek. The status block is pointed to by `statb`. `STAT_BLK` is defined in `cdfm.h` and below:

```c
typedef struct stat_blk {
  short asy_stat; /* Current status of the operation */
  short asy_sig;  /* Signal to be sent on termination */
} STAT_BLK;
```

`asy_stat` contains the current status of the play operation. The system updates this field. It should not be altered by the application. The bit values are defined as follows:

- `Bit     0` If set, play is finished
- `Bits 1-14` Reserved (must be zero)
- `Bit    15` If set, an error has occurred.

`asy_sig` contains the signal number to be sent when the operation is finished or an error occurs. It is initialized by the application before the operation is begun and may be changed by the application during the operation. If this field is set to zero, no signal is sent when the play is finished or an error occurs. The error code is placed in this field when an error occurs during the operation.

-----

### `ss_slink()` Link in Subroutine Module

*UCM File Manager C Binding*

```c
ss_slink(path, modname)
int  path;              /* Path to video device */
char *modname;          /* Name of UCM extension module */
```

 `ss_slink()` links the module specified by modname as the subroutine extension module for UCM.

This subroutine module will be passed all unknown Getstat and Setstat system calls received by UCM. If this module does not recognize the system calls, they will be passed on to the driver.

80 bytes of UCM device static storage are reserved for this subroutine module to use. Because this memory is global to all paths open to UCM, it is impossible for two subroutine modules to be linked at the same time. However, more than one process may be linked to the same subroutine module.

For further information about the use of subroutine modules and the `SS_SLink` system call, refer to the Green Book.

If an error occurs `-1` is returned and `errno` is set.

-----

### `ss_stop()` Stop the Disc Drive

*CDFM File Manager C Binding*

```c
ss_stop(path)
int path;
```

`ss_stop()` causes the disc drive to stop. If a play operation is in progress, the equivalent of an `ss_abort()` is executed first. This call does not open the drive door.

If an error occurs `-1` is returned and `errno` is set

-----

### `viq_cpos()` Return Relative Character Positions

*UCM File Manager C Binding*

```c
viq_cpos(path, dmid, str, buff, maxchrs)
int   path,             /* Path to video device */
      dmid;             /* Drawmap id */
char  *str;             /* Pointer to text*/
int   *buff;            /* Pointer to buffer for returning coordinates*/
short maxchrs;          /* Maximum number of characters to use */
```

`viq_cpos()` calculates the relative horizontal placement of characters if the string pointed to by `str` were to be printed using the `dr_text()` function. Up to `maxchrs` characters are used or until a NULL byte is reached in the string. The UCM coordinate positions are returned in a buffer pointed to by `buff`. The coordinates are returned in an array of shorts. Consequently, the buffer must have at least twice as many bytes as characters specified by `maxchars`.

`-1` is returned on error and `errno` is set.

-----

### `viq_dminfo()` Return Drawmap Descriptor Information

*UCM File Manager C Binding*

```c
#include <ucm.h>

DrawmapDesc *viq_dminfo(path, dmid)
int path,               /* Path to video device */
    dmid;               /* Drawmap id */
```

`viq_dminfo()` returns a pointer to the drawmap descriptor for the drawmap specified by `dmid`.

If an error occurs, this function returns a NULL pointer and `errno` is set.

-----

### `viq_fdta()` Return Font Data

*UCM File Manager C Binding*

```c
#include <ucm.h>

viq_fdta(path, fntid, buff)
int path,               /* Path to video device */
    fntid;              /* Font id */
FONTDATA *buff;         /* Pointer to buffer to hold font data */
```

`viq_fdta()` places the font data section of the font specified by `fntid` into the `FONTDATA` structure pointed to by `buff`. `FONTDATA` is defined in `ucm.h` and below:

```c
typedef struct FontData {
  unsigned short fnt_type,     /* Font type and flags */
                 fnt_width.    /* Maximum Glyph cell width */
                 fnt_height,   /* Glyph cell height (ascent+descent) */
                 fnt_ascent,   /* Ascent of char cell above baseline */
                 fnt_descent,  /* Descent of cell below baseline */
                 fnt_pxlsz,    /* Pixel size in bits */
                 fnt_frstch,   /* First character value of font */
                 fnt_lastch;   /* Last character value of font */
    unsigned int fnt_lnlen,    /* Line length of font bitmap in bytes */
                 fnt_offstbl,  /* Offset to glyph offset table */
                 fnt_datatbl,  /* Offset to glyph data table */
                 fnt_map1off,  /* Offset to first bitmap */
                 fnt_map2off;  /* Offset to second bitmap */
} FONTDATA;
```

`-1` is returned on error and `errno` is set.

-----

### `viq_gdta()` Return Glyph Data

*UCM File Manager C Binding*

```c
viq_gdta(path, fntid, glyphno, buff)
int   path,             /* Path to video device */
      fntid,            /* Font id */
      glyphno;          /* Glyph number */
char *buff;             /* Pointer to buffer to hold glyph data */
```

`viq_gdta()` returns the glyph data table entry in the 4-byte buffer pointed to by `buff` for the glyph specified by `glyphno` for the font specified by `fntid`.

`-1` is returned on error and `errno` is set.

-----

### `viq_jcps()` Return Character Positions for Justified Text

*UCM File Manager C Binding*

```c
viq_jcps(path, dmid, justlen, str, buff, maxchrs)
int   path,             /* Path to video device */
      dmid,             /* Drawmap id */
      justlen;          /* Justification length */
char  *str;             /* Pointer to text*/
int   *buff;            /* Pointer to buffer for returning coordinates*/
short maxchrs;          /* Maximum number of characters to use */
```

`viq_jcps()` calculates the relative horizontal placement of characters for justified text that is to be printed with `dr_jtxt()`. The text string is pointed to by `str` and the justification length is given by `justlen`. Only up to `maxchrs` characters are used in the string or until a NULL byte is reached. The relative positions in UCM coordinates will be returned in a buffer pointed to by `buff`. The coordinates are returned in an array of shorts. Consequently, the buffer must have at least twice as many bytes as characters specified by `maxchars`.

`-1` is returned on error and `errno` is set.

-----

### `viq_pntr()` Test if Point is in Region

*UCM File Manager C Binding*

```c
viq_pntr(path. regid, x, y)
int path,               /* Path to video device */
    regid,              /* Region id */
    x, y;               /* coordinates to test */
```

`viq_pntr()` tests whether the point specified by `x`, `y` is found in the region identified by `regid`.

This function returns non-zero if true and zero if false.

-----

### `viq_rinfo()` Return Region Descriptor Information

*UCM File Manager C Binding*

```c
#include <ucm.h>

RegionDesc *viq_rinfo(path, regid)
int path,               /* Path to video device */
    regid;              /* Region id */
```

`viq_rinfo()` returns a pointer to the region descriptor for the region specified by regid.

On error it returns a NULL pointer and errno is set.

-----

### `viq_rloc()` Inquire Region Location

*UCM File Manager C Binding*

```c
viq_rloc(path, regid, sx, sy, ex, ey)
int path,               /* Path to video device */
    regid;              /* Region id */
   *sx, *sy,            /* Pointers to region's upper left coordinates */
   *ex, *ey;            /* ptrs to region's lower right coordinates */
```

`viq_rloc()` finds the boundary rectangle of a region and returns the coordinates of its opposite comers in the values pointed at by `sx`, `sy` and `ex`, `ey`.

`-1` is returned on error and `errno` is set.

-----

### `viq_txtl()` Calculate Length of Text

*UCM File Manager C Binding*

```c
viq_txtl(path, dmid, length, str, nochars, maxchrs)
int   path,             /* Path to video device */
      dmid,             /* Drawmap id */
      length;           /* Length of text */
char  *str;             /* Pointer to text */
int   *nochars,         /* Pointer to number of chars that fit in length */
      maxchrs:          /* Maximum number of characters to calculate */
```

`viq_txtl()` calculates the following:

1) If the number of characters in the string pointed to by `str` is less than `maxchrs`, the pixel length of the entire string is returned as UCM coordinates. If the number of characters is greater than `maxchrs`, the pixel length of `maxchrs` characters of the string is returned.
2) The number of characters (in `str`) that will fit in the specified length is returned in the integer pointed to by `nochars`.

This is useful for text placement using `dr_text()` and `dr_jtxt()`.

`-1` is returned on error and `errno` is set.
