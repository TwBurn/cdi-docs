# CD-i MPEG Video Libraries Technical Reference Manual

> This document is a reconstruction as any original documentation seems to be lost. Notes by the creator of this document are listed in quote sections like this one.
>
> Please refer to the Green Book description of the corresponding `MV_` function for complete information on function parameters and behaviour..
>
> Any original document was probably &copy; Copyright 1993, Philips Consumer Electronics B.V.

## Table of contents

- [`mv_abort()`](#mv_abort-abort-the-current-mpeg-video-play) Abort the current MPEG Video play
- [`mv_borcol()`](#mv_borcol-set-the-border-colour) Set the border colour
- [`mv_chspeed()`](#mv_chspeed-change-the-mpeg-video-playback-speed) Change the MPEG Video playback speed
- [`mv_close()`](#mv_close-free-the-used-video-descriptor) Free the used video descriptor
- [`mv_cncl()`](#mv_cncl-replace-errors-in-mpeg-video-datablock) Replace errors in MPEG Video datablock
- [`mv_continue()`](#mv_continue-continue-a-paused-play) Continue a paused play
- [`mv_create()`](#mv_create-reserve-and-initialize-a-video-map-descriptor) Reserve and initialize a video map descriptor
- [`mv_freeze()`](#mv_freeze-freeze-the-current-picture-cd-only) Freeze the current picture (CD only)
- [`mv_hide()`](#mv_hide-disable-the-window-output) Disable the window output
- [`mv_imgsize()`](#mv_imgsize-preset-picture-sizes) Preset picture sizes
- [`mv_info()`](#mv_info-return-a-pointer-to-the-video-map-descriptor) Return a pointer to the video map descriptor
- [`mv_jump()`](#mv_jump-compensate-video-streams-timing-parameters-on-behalf-of-seamless-jump) Compensate video streams timing parameters on behalf of seamless jump
- [`mv_loop()`](#mv_loop-repeat-a-number-of-pictures) Repeat a number of pictures
- [`mv_cdnext()`](#mv_cdnext-display-next-cd-picture) Display next CD picture
- [`mv_hostnext()`](#mv_hostnext-display-next-host-picture) Display next Host picture
- [`mv_off()`](#mv_off-turn-off-the-display-output) Turn off the display output
- [`mv_org()`](#mv_org-set-origin-offset) Set origin offset
- [`mv_pause()`](#mv_pause-pause-the-current-asynchronous-play) Pause the current asynchronous play
- [`mv_cdplay()`](#mv_cdplay-play-video-from-cd) Play video from CD
- [`mv_hostplay()`](#mv_hostplay-start-play-of-a-host-video-map) Start play of a host video map
- [`mv_pos()`](#mv_pos-set-position-of-window) Set position of window
- [`mv_release()`](#mv_release-release-the-mpeg-video-decoder-for-other-processes) Release the MPEG Video decoder for other processes
- [`mv_selstrm()`](#mv_selstrm-select-an-mpeg-video-stream) Select an MPEG Video stream
- [`mv_show()`](#mv_show-enable-the-display-of-the-window) Enable the display of the window
- [`mv_slink()`](#mv_slink-link-to-external-subroutine-module)-Link to external subroutine module
- [`mv_status()`](#mv_status-return-the-status-of-an-active-mpeg-video-play) Return the status of an active MPEG Video play
- [`mv_trigger()`](#mv_trigger-define-mpeg-video-events-to-signal) Define MPEG Video events to signal
- [`mv_window()`](#mv_window-define-display-window-within-decoded-picture) Define display window within decoded picture

## Introduction

The functions described in this chapter are additions to the standard OS-9 library.

The library has the following name:

- `mvcdi.l` c-bindings with parameter checking
- `mvcdin.l` c-bindings without parameter checking

A C-program accessing the in this chapter mentioned functions must
link with the following library files. For example:

```plaintext
cc prog.r -l=/dd/lib/mvcdi.l
          -l=/dd/lib/fmv.l -f=prog
```

All constants and structure definitions are made in DEFS file `mv.h`.

## MPEG Video Library Functions

### `mv_abort()` Abort the current MPEG Video play

```c
mv_abort(path)
int     path; /* Path Number */
```

This function aborts the play that is currently being executed.
The play will not be active any longer and
fields in the map descriptor are reset. The last
video picture displayed will remain visible.

This is a privileged call
and may used only by a process with a userID of the
super user or with the userID of the process which
started the operation. If no play is active, an abort
error will be returned.

If this play was running in synchronized mode with
an audio play, then this call will end the synchronized
mode. The audio playback will continue in non-synchronized
mode.

`mv_abort()` function returns `0` if successful, otherwise `-1` is returned and `errno` is set.

### `mv_borcol()` Set the border colour

```c
mv_borcol(path, vimid, color)
int     path,     /* Path Number */
        vimid,    /* Video mapID */
        color;    /* Color Value */
```

This function sets the border colour. If the specified MVM is currently active, the
parameters are copied into the MVM descriptor and the parameters are activated
immediately. If the MVM is not active, the parameters are copied into the MVM
descriptor only.

Format of the `color` parameter:

```plaintext
bits   0-7  Blue
      8-15  Green
     16-23  Red
     24-31  Unused
```

`mv_borcol()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_chspeed()` Change the MPEG Video playback speed

```c
mv_chspeed(path, offset, speed, pcl)
int      path,       /* Path Number */
         offset,     /* Offset to start */
         speed,      /* Speed parameter */
         syncoffset; /* Sync offset to video */
PCL      *pcl;       /* Address of PCL structure */
```

This function changes the speed of an active play. When changing to scan mode or
single picture forward mode, the system will not restart decoding anymore. Any
next picture must be requested with an `mv_next()` call. If no MapID is active or when
this call is made for a map of type Still or when the currently active play is frozen,
a bad mode error is returned.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which started the play.

If, during synchronized mode, the application changes from normal speed to nonnormal speed, the decoder will stay in synchronized mode and will mute the audio.

If a change to normal speed takes place, audio will be demuted again and
synchronization will be restored.

The format of the speed parameter is defined in Chapter IX.8.2.3.3.

The contents of the PCL parameter and the Offset to start parameter are ignored
when:

- the new speed mode is scan mode (the call `mv_next()` will provide the required
values for `offset` and `pcl`).
- the old speed is NOT scan mode. The system will continue reading from the
current position.

#### Playing from CD

If a play is already active, the PCL pointer may be passed to make the system fluently
change to another chain of PCL buffers.

If the application requests for such a change, the system will link the new chain of
PCL's to the first PCL structure of which the buffer is emptied. If the system is
already changing to another chain of PCL buffers, this parameter is ignored which
may result in using a chain of PCL's unfit for the new speed mode. When the system
does no longer use the 'old' PCL structure, it will restore the link and the application
will be informed (by the CNP bit in the status block or by the CNP signal if set with
`mv_trigger`).

The Offset to start parameter must contain a byte offset (relative to the first incoming
byte) to the first byte of a pack start code, or the first byte of a sequence header
code, group start code or picture start code.

#### Consequences on changing the speed when playing from CD

- A change to normal speed can be done in two ways using the call:
    1) `mv_chspeed` Speed change is fluent, but synchronization between audio and video is only restored after a while, depending on the amount of data in the chain of PCLbuffers and the MPEG decoder.
    2) `mv_play()` The speed change is not fluent, but audio and video are immediately synchronized.
- When changing to slow motion, the application must be sure to have enough
data in the PCL-buffers to decode while pausing and continuing the CD-player.
- When changing from slow motion forward mode or single picture forward mode
to normal forward mode, the system will not change to normal speed until a new
sector from disc is detected or when a recommended time out period of 4
seconds has elapsed. After this speed change, the MPEG decoder must get rid
of superfluous data by skipping pictures or frames.
- If the system is in scan mode and an `mv_next()` call is issued, the `mv_chspeed()`
call will return the `E$DevBsy` error from the moment that data from disc is copied
until the picture is displayed unless the new speed is scan speed.

#### Playing from host

The PCL pointer parameter is not used. The `offset` to start parameter must contain
a byte offset (relative to the start of the map) to the first byte of a pack start code, or
the first byte of a sequence header code, group start code or picture start code.

`mv_chspeed()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_close()` Free the used video descriptor

```c
mv_close(path,vimid)
int     path,     /* Path Number */
        vimid;    /* Video mapID */
```

This function aborts any ongoing actions on the
specified video mapID and frees the used descriptor.
This is a privileged call and may be used only by a
process with a userID of the super user or with the
userID of the process which created the video mapID.

`mv_close()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_cncl()` Replace errors in MPEG Video datablock

```c
mv_cncl(path, size, errdata, data)
int     path,            /* Path Number */
        size;            /* Size of the datablock to correct */
PL_ERR  *errdata;        /* Error data structure */
char    *data            /* Pointer to data */
```

This function, for host only, replaces suspected data in an MPEG datablock. If the
map to which the specified data belongs is currently in use (= data retrieved from),
data integrity is not guaranteed for the complete map.

Error concealment is automatically done during a Full Motion video play that makes
use of a PCL provided that the PCL also includes the error structure.

`mv_cncl()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

### `mv_continue()` Continue a paused play

```c
mv_continue(path, filter)
int     path,            /* Path Number */
        filter;          /* Filter parameter */
```

This function continues a paused or frozen MPEG Video play. If the play is not in
paused or frozen state or if no MPEG Video play is active, a bad mode error will be
returned.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which paused the play.

If the MPEG video play is active in synchronized mode with MPEG audio, and if this
synchronized play is paused or frozen, this function will cause both the video and
the audio to continue.

If the play was frozen, decoding and displaying starts according the filter parameter.
During the time the play is frozen, data retrieval continues, so this has not to be
restarted.

If the play was paused, the data retrieval from CD must be restarted after this call.

In case of normal speed play, data available in the internal buffer(s) will be decoded
when new data from disc is received or when a recommended time out period of
4 seconds has elapsed.

Data retrieval from 'host' is restarted automatically by the decoder. The application
does not have to do any additional (seek) actions.

The filter parameter is ignored when the player is continued from a paused state.

The values of the filter parameter:

- 0: Resume decoding as soon as possible, provided that the sequence parameters
are known by the MPEG decoder
- 1: Resume decoding at the next group of pictures, provided that the sequence
parameters are known by the MPEG decoder
- 2: Resume decoding at the next sequence header
- Other values are reserved.

`mv_continue()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_create()` Reserve and initialize a video map descriptor

```c
mv_create(path, type)
int     path,   /* Path Number */
        type;   /* Descriptor Type */
```

This call reserves and initializes an MVM descriptor. The format of the descriptor can be found in the
`mv.h` file. The initial values of the descriptor fields
are given in the following table:

||

If the selected type is "still:, the `mv_play()` call returns an `E$IllPrm` error when the page
number is out of bounds. Page zero is the first selectable page.

Format of the `type` parameter:

```plaintext
     +------+-------+-------------------+-----+
     | Page | Still |                   | SRC |
     +------+-------+-------------------+-----+
bit   15..12   11    10      ...       1   0
```

Bit 0: When this bit is set, the decoder will assume that data is coming from CD
(in PCL structure). If this bit is not set, data must be provided in host
memory.

Bit 1-10: Reserved for future use, must be zero
Bit 11: When set, the decoder will play the MPEG Still Picture sequence in single
step into the specified page.

Bit 12-15: When bit 11 is set, these bits indicate in what page the data must be
stored. The application should check the CSD to know the number of Still
pages supported by the hardware.

`mv_create()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_freeze()` Freeze the current picture (CD only)

```c
mv_freeze(path)
int     path;  /* Path Number */
```

This function causes the output of a normal speed MPEG Video play to be frozen,
while the data retrieval continues. Output will be resumed after issuing an
`mv_continue()` call. If no normal speed play is active, or if the play is already frozen
or paused, a bad mode error will be returned.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which started the play.

`mv_freeze()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_hide()` Disable the window output

```c
mv_hide(path)
int     path;  /* Path Number */
```

This function disables the output of the window on the next vertical retrace. The
window will become black but decoding continues.

This is a privileged function. Permission is granted if the function is called by the
super user, if no play is currently active or if the currently active play was started by
the same user that calls this function.

`mv_hide()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_cdnext()` Display next CD picture

```c
mv_cdnext(path, page_number, offset, pcl)
int     path,        /* Path Number */
        page_number, /* Page number (stills only) */
        offset;      /* Offset to start */
PCL     *pcl;        /* Address of PCL structure */
```

This function must be used to step to the first or the next picture when in single
picture forward mode (or playing still pictures) or to tell the MPEG Video decoder
that it must expect new data to display the next entry point when in scanning mode. In this mode, the next picture will be made visible when the current scantime
(parameter in `mv_play()` and `mv_chspeed()` calls) has elapsed.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which started the play.

The parameter `page_number` is only used when the active mapID is of the type
"Still". It must be given to select the page to decode the next picture in. If the value
does not fit the number of pages available in hardware, an `E$IllPrm` is returned. The
first page is identified as page zero.

When playing from CD in scan mode, this call indicates that the chain of PCLstructure was emptied by the application and that new data is to be expected. The
decoder does not accept a new `mv_next()` call (device busy) until the picture is
displayed (a PIC-signal (PIC/GOP/SOS/LPD) is sent).

When playing in single step mode the next picture will be made visible as soon as
possible (data available and decoded). The PCL pointer and offset parameter are
ignored in this mode.

If no play is active, an `E$BMode` error is returned.

`mv_cdnext()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_hostnext()` Display next Host picture

```c
mv_hostnext(path, page_number, offset)
int     path,        /* Path Number */
        page_number, /* Page number (stills only) */
        offset;      /* Offset to start */
```

This function must be used to step to the first or the next picture when in single
picture forward mode (or playing still pictures) or to tell the MPEG Video decoder
that it must expect new data to display the next entry point when in scanning mode. In this mode, the next picture will be made visible when the current scantime
(parameter in `mv_play()` and `mv_chspeed()` calls) has elapsed.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which started the play.

The parameter `page_number` is only used when the active mapID is of the type
"Still". It must be given to select the page to decode the next picture in. If the value
does not fit the number of pages available in hardware, an `E$IllPrm` is returned. The
first page is identified as page zero.

When playing from host in scan mode, the Offset to start parameter (relative to the
start of the map) must be passed.

When in single step mode, the Offset to start parameter is ignored.

If no play is active, an `E$BMode` error is returned.

`mv_hostnext()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_imgsize()` Preset picture sizes

```c
mv_imgsize(path, vimid, width, height, rate)
int     path,     /* Path Number */
        vimid;    /* Video mapID */
        width,    /* Width (UCM coords) */
        height,   /* Height (UCM coords) */
        rate;     /* Picture rate (23..25 or 29..30) */
```

This function sets in the MVM descriptor the picture size and picture rate of the
sequence to be played.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which created the descriptor.

The parameters are specific to the sequence and cannot be changed by an
application during a play (the call will return an `E$BMode` error when active).

Changing the picture size (by a call or by the data stream) may clip the specified
window.

The width must be less than or equal to 768, the height must be less than or equal
to 576. Except for stills, the picture rate can be 23, 24, 25, 29 or 30 and:

- ((W/2 + 15)/16) \* ((H/2 + 15)/16) &leq; 396
- ((W/2 + 15)/16) \* ((H/2 + 15)/16) \* rate &leq; (396 \* 25)

`mv_imgsize()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_info()` Return a pointer to the video map descriptor

```c
MVmapDesc *mv_info(path, vimid)
int     path,    /* Path Number */
        vimid;   /* Video mapID */
```

This function returns a pointer to the video map
descriptor corresponding to the given video mapID.
The fields in the descriptor are for information
purposes only. Fields should only be changed by
calling the appropriate functions.
The video map descriptor has the following format:

```c
typedef struct _mvmapdesc {
    unsigned short  MD_Id;        /* Id of the MVM */
    unsigned short  MD_Type;      /* type of this map */
    unsigned short  MD_Stream;    /* Current stream number */
    unsigned int    MD_StLoop;    /* Start address of loop */
    unsigned int    MD_EnLoop;    /* End address of loop */
    unsigned short  MD_LpCnt;     /* Initial loopback count */
    unsigned short  MD_LCntr;     /* Remaining loops */
    unsigned int    MD_ImgSz;     /* Image size (W:H) */
    unsigned int    MD_DecWin;    /* Window in the image (W:H) */
    unsigned int    MD_DecOff;    /* Offset window to decode (H:V) */
    unsigned int    MD_ScrOrg;    /* Origin offset window (H:V) */
    unsigned int    MD_ScrOff;    /* Offset within screen (H:V) */
    unsigned int    MD_BCol;      /* Border color */
    unsigned int    MD_Speed;     /* Display speed */
    unsigned int    MD_TimeCd;    /* timecode of current picture */
    unsigned short  MD_TmpRef;    /* temporal reference */
    unsigned char   MD_PicRt,     /* picture rate */
                    MD_Res1[11];  /* reserved */
} MVmapDesc;
```

`mv_info()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_jump()` Compensate video streams timing parameters on behalf of seamless jump

```c
mv_jump(path, delta)
int     path,     /* Path Number */
        delta;    /* Delta value */
```

> In earlier DVC ROMs this function was notoriously unreliable, so much so that
> no released CD-i titles actually use this function.

The application is allowed, when playing from CD, to
discontinue the current data stream and to start
supplying data that comes from another position. To
overcome a discontinuity in timing parameters, this
function provides the application a means to adapt the
new timing parameters in such a way that they fit
seamless into the old stream. As soon as either a play
is started or the decoder detects a discontinuity in
the supplied stream, it will start correcting the
timing parameters with a given delta value. This delta
value is defined as SCR(j) - SCR(i) and is given in a
resolution of 22.5 kHz. SCR( j) is the value of the
first sector of the new data and SCR(i) is the value
of the SCR of the sector that would have been supplied
next if no jump was done. Initially the delta value is
zero. After a play is aborted, the delta value will
be reset to zero. When playing from host, the delta
value will have no influence at all.

Since a discontinuity in SCR is detected only if the
difference in between SCRs is negative or greater than
0.70 seconds, a positive delta value less than or
equal to 15,750 is considered as an illegal parameter.

`mv_jump()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_loop()` Repeat a number of pictures

```c
mv_loop(path, vimid, strtaddr, endaddr, count)
int     path,     /* Path Number */
        vimid,    /* Video mapID */
        strtaddr, /* start address of loop */
        endaddr,  /* end address of loop */
        count;    /* loopcounter */
```

This function sets the loopback points for a play from memory.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which created the descriptor.

If this MVM is currently not active, parameter checking is not possible since a link to
a map will exist only after an `mv_play()` is issued (until then no mapsize is available).

When the map is activated by `mv_play()`, the loop variables will be checked.
This call is only allowed for MVM's of type Host. For other types, a bad mode error
will be returned. If the MVM is in scan mode, setting the loopback variables will
have no immediate effect. As soon as the speed is changed to normal, slow motion
or single step mode, the loopback function will be executed.

The Starting loopback point offset contains an MPEG Video Pointer, provided that the sequence parameters are either in the datastream or set
by the application (`mv_selstrm()` or `mv_imgsize`), the Ending loopback point offset
must be on the byte following the last byte of the last picture to be included in the
loop area. It is the application's responsibility to provide the correct addresses. The
addresses must be relative to the start of the map.

When the MVM is afterwards played (through `mv_play()`), it will play the pictures from
the start of the looparea until the end of the looparea is reached. The first picture
to be displayed is the first Access picture encountered in the loop area. The
Loopcount will be decremented and when it did not reach zero, the images between
the looppoints will be displayed again until the Loopcount is zero.

If the MVM is already being played when this call is made, the behavior of this
function depends on the current position in the MVM. If the current position is
before the Ending loopback point offset, then playing continues until the Ending
loopback point offset is reached, at which time the Loopcount is decremented and
the looping area is played again, if necessary. If the current position is already
behind the Ending loopback point offset, a jump will take place to the Starting
loopback point offset at the next picture change.

The application is responsible for setting the correct sequence parameters before the
play is started if these are not in the data itself. Every time a jump is made to the
Starting loopback point offset, the decoder will set the sequence parameters as they
are available in the descriptor.

If the `mv_loop()` call is applied to a play which is already in loopback mode, then the
currently active loop area will be completed first (i.e. played until the end position
of the active loop area). At that moment, the loop parameters of the most recent
MV_Loop call will become active. How the play continues depends on the current
position (which is the end position of the 'old' loop area). See the sub-section
before the previous one.

If the designated mapID is active and in synchronized mode with audio, then this
`mv_loop()` call will result in a `E$BMode` error.

`mv_loop()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_off()` Turn off the display output

```c
mv_off(path)
int     path;  /* Path Number */
```

This function disables the output of the display. The screen will become black but
decoding continues.

This is a privileged function. Permission is granted if the function is called by the
super user, if no play is currently active or if the currently active play was started by
the same user that calls this function.

`mv_off()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_org()` Set origin offset

```c
mv_org(path, vimid, x, y, scroll)
int     path,     /* Path Number */
        vimid,    /* Video mapID */
        x,        /* Horizontal Origin (UCM coordinates) */
        y,        /* Vertical Origin (UCM coordinates) */
        scroll;   /* Scroll Flag (not 1: scroll, 0: no scroll) */
```

MV_Org sets the origin of the window relative to the top left corner of the full screen
image. This facilitates the overlay of the graphics cursor. Default origin is the upper
left corner of the full screen image, coordinates 0:0.

If the specified map is currently active, the new parameters will become effective on
the next picture change. If the `scroll` parameter is not zero and the window
move is over less than 16 lines, the window will be repositioned on the next vertical
retrace.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which created the descriptor.

`mv_org()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

### `mv_pause()` Pause the current asynchronous play

```c
mv_pause(path)
int     path;  /* Path Number */
```

This function pauses the asynchronous play that is
currently being executed. This is a privileged
function call and may be used only be a process with
a userID of the super user or with the userID of the
process that started, the play operation. If no
asynchronous play is active at the time this
function is called, or if the play is already
paused, a bad mode error will be returned.

If the video play is in synchronized mode with
audio, this function will pause both the audio and
the video playback.

`mv_pause()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

### `mv_cdplay()` Play video from CD

```c
mv_cdplay(path, vimid, offset, pcl, statusblk, syncmode, syncoffset)
int      path,       /* Path Number */
         vimid,      /* Video mapID */
         offset,     /* Offset in PCL buffer */
         syncmode,   /* Sync mode/path to sync */
         syncoffset; /* Sync offset to video */
PCL      *pcl;       /* Address of PCL structure */
STAT_BLK *statusblk; /* Pointer to statusblock */
```

This function starts to play the data belonging to the given MPEG Video mapID. Play is an asynchronous operation, i.e. the application continues execution at the same time as the play is executed. To have the first picture displayed when the playback speed is scan, single picture forward or stills, an `mv_next()` call must be issued.

This is a privileged call and may be used only by a process with a userID of the super user or with the userID of the process which created the descriptor.

When playing from disc, the PCL parameter points to that PCL structure from the circularly linked chain of PCL's (Transfer Buffer), in which the first sector from the disc is to be expected. In practice, this corresponds to the address in the video CIL for the selected CD-I channel. If a buffer of which the PCL_Sig field contains a signal value is emptied, this signal is sent to the application.

If a play on the given path is already queued, then this call will be queued by the kernel. If the process is currently playing through the same path, the active play will be aborted and the new play will be started. If the process has an active play through some other path, or if the given path is active because another process started a play on that path, then a `E$DevBsy` will be returned. If some other process has a play active on a different path, then this `mv_play()` request will be queued. The next play in the queue will be activated as soon as the active play is released through a `mv_release()` call.

The first `mv_play()` call after the MPEG Video decoder was released (or after a cold start) will allocate the MPEG Video buffer. If that memory is in use (modules are
resident), a E$NoRAM error is returned. If, for a non-still map, the speed parameter is not equal to zero (scan, slow motion or single step), the application must take care of providing the correct data at the correct moment (i.e. doing a play from the CD and pause/continue that dataflow).

If loopback variables are set (relative to the beginning of the map) , these will be tested before it starts the play. If these variables are not correct (not within the
offered map), an `E$IllPrm` error is returned. Otherwise playback starts at the starting loopback point. Loopback variables will have no effect when playing in scan mode.

Each `mv_play()` call will enable the video. If `mv_show() was issued before, a previously defined window will become visible.

If the type of the descriptor is still-image, the data is decoded into the still page as indicated by the MD_Type field of the descriptor. The speed parameter is ignored. The decoder handles this play as if it were playing a sequence in single step speed. The first picture is decoded into the specified page (an illegal page number will result in an E$IllPrm error). An `mv_next()` call requests the decoder to reconstruct the following picture (if any) and to store into the page specified by that `mv_next() call.

Playing non-still type maps destroys the contents of the still pages in the decoder.

A play is finished:

1) When playing from CD, an `mv_abort()` must be issued. When running out of data the application must decide whether it is the end of the play or that it is temporarily out of data.
2) When playing from Host, an `mv_abort()` may be used. When the decoder runs out of data the play will also be finished when the last available picture is displayed (LPD signal).
3) Still pictures are treated as a normal sequence in single step mode.

If the sync mode parameter is set to `-1`, then the decoder assumes that no synchronization to MPEG Audio is required. This mode is called the Nonsynchronized mode. If the sync mode parameter is set to `-2`, then this play will enter a wait state. It will remain in this state until an audio play, that must synchronize to this video path, has been started. This mode is called the Wait mode.

In synchronized mode, this parameter contains the path number of the active MPEG Audio play to which this MPEG Video play should synchronize itself. In the synchronized mode, no queuing will be done. Any play request while the decoder is in the synchronized mode will result in `E$DevBsy`. The play that is active on the (sync) path to which this function wants to synchronize, should have been started by the same process and user. If not, a permission error will result.

The synchronization offset parameter indicates the constant difference between the timing parameters in the audio and video sequence. This parameter is defined, in units of 22.5 kHz, as the most significant 32 bits of the difference between the decoder system clocks in the MPEG Video decoder and the MPEG Audio decoder. In a formula: `dsc(video) - dsc(audio)`.

`mv_cdplay()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_hostplay()` Start play of a host video map

```c
mv_hostplay(path, vimid, size, vidmap, offset, statusblk, syncmode, syncoffset)
int      path,         /* Path Number */
         vimid,        /* Video mapID */
         size,         /* Size of the video map */
         offset,       /* Offset within video map */
         syncmode,     /* Sync mode/path to sync with */
         syncoffset;   /* Sync offset */
int      *vidmap;      /* Pointer to video map */
STAT_BLK *statusblk;   /* Pointer to statusblock */
```

This function starts to play the data belonging to the given MPEG Video mapID. The data may be coming from disc or from host memory. Play is an asynchronous operation, i.e. the application continues execution at the same time as the play is executed. To have the first picture displayed when the playback speed is scan, single picture forward or stills, an `mv_next()` call must be issued.

This is a privileged call and may be used only by a process with a userID of the super user or with the userID of the process which created the descriptor.

If a play on the given path is already queued, then this call will be queued by the kernel. If the process is currently playing through the same path, the active play will be aborted and the new play will be started. If the process has an active play through some other path, or if the given path is active because another process started a play on that path, then a `E$DevBsy` will be returned. If some other process has a play active on a different path, then this `mv_play()` request will be queued. The next play in the queue will be activated as soon as the active play is released through a `mv_release()` call.

The first `mv_play()` call after the MPEG Video decoder was released (or after a cold start) will allocate the MPEG Video buffer. If that memory is in use (modules are
resident), a E$NoRAM error is returned. If, for a non-still map, the speed parameter is not equal to zero (scan, slow motion or single step), the application must take care of providing the correct data at the correct moment (i.e. doing a play from the CD and pause/continue that dataflow).

If loopback variables are set (relative to the beginning of the map) , these will be tested before it starts the play. If these variables are not correct (not within the
offered map), an `E$IllPrm` error is returned. Otherwise playback starts at the starting loopback point. Loopback variables will have no effect when playing in scan mode.

Each `mv_play()` call will enable the video. If `mv_show() was issued before, a previously defined window will become visible.

If the type of the descriptor is still-image, the data is decoded into the still page as indicated by the MD_Type field of the descriptor. The speed parameter is ignored. The decoder handles this play as if it were playing a sequence in single step speed. The first picture is decoded into the specified page (an illegal page number will result in an E$IllPrm error). An `mv_next()` call requests the decoder to reconstruct the following picture (if any) and to store into the page specified by that `mv_next() call.

Playing non-still type maps destroys the contents of the still pages in the decoder.

A play is finished:

1) When playing from CD, an `mv_abort()` must be issued. When running out of data the application must decide whether it is the end of the play or that it is temporarily out of data.
2) When playing from Host, an `mv_abort()` may be used. When the decoder runs out of data the play will also be finished when the last available picture is displayed (LPD signal).
3) Still pictures are treated as a normal sequence in single step mode.

If the sync mode parameter is set to `-1`, then the decoder assumes that no synchronization to MPEG Audio is required. This mode is called the Nonsynchronized mode. If the sync mode parameter is set to `-2`, then this play will enter a wait state. It will remain in this state until an audio play, that must synchronize to this video path, has been started. This mode is called the Wait mode.

In synchronized mode, this parameter contains the path number of the active MPEG Audio play to which this MPEG Video play should synchronize itself. In the synchronized mode, no queuing will be done. Any play request while the decoder is in the synchronized mode will result in `E$DevBsy`. The play that is active on the (sync) path to which this function wants to synchronize, should have been started by the same process and user. If not, a permission error will result.

The synchronization offset parameter indicates the constant difference between the timing parameters in the audio and video sequence. This parameter is defined, in units of 22.5 kHz, as the most significant 32 bits of the difference between the decoder system clocks in the MPEG Video decoder and the MPEG Audio decoder. In a formula: `dsc(video) - dsc(audio)`.

`mv_hostplay()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_pos()` Set position of window

```c
mv_pos(path, vimid, x, y, scroll)
int     path,     /* Path Number */
        vimid,    /* Video mapID */
        x,        /* Horizontal Origin (UCM coordinates) */
        y,        /* Vertical Origin (UCM coordinates) */
        scroll;   /* Scroll Flag (not 1: scroll, 0: no scroll) */
```

This function sets the position of the window on the full screen image, relative to the
origin as set with `mv_org()`.

If the specified map is currently active, the new parameters will become effective on
the next picture change. If the `scroll` parameter is not zero and the window
move is over less than 16 pixels and less than 16 lines, the window will be
repositioned on the next vertical retrace.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which created the descriptor.

```plaintext
       o-----------------+
SCREEN |         ^       |
       |         Y       |
       |         v       |
       |<   X   >+-------+===+  WINDOW
       |         |       |   :
       +---------+-------+   :
                 :           :
                 +===========+
                    THIS PART WILL NOT BE VISIBLE
```

`mv_pos()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_release()` Release the MPEG Video decoder for other processes

```c
mv_release(path)
int     path;  /* Path Number */
```

When an MPEG Video play has finished, the process should release the decoder for
other processes. As long as the decoder is not released, other processes that issued
an `mv_play()` command will remain in the sleeping queue.

If a map is active when the `mv_release()` function is called, the play will be aborted.
Implicitly, the synchronized mode, if enabled, will end because of this call. The
MPEG Audio playback, that is part of this synchronized play, will continue in nonsynchronized mode, without MPEG video.

If the display output was not yet turned off, the MV_Release call will disable the
output of the display. If the decoder was already released, a bad mode error will be returned.

This is a privileged call and may be used only by a process with a userID of the super
user or, if the given path has an active play, by a process with the same userID as that of the process which started the play.

This call will also return the MPEG Video buffer to the operating system when no
more plays are queued, otherwise the play that is queued first will be started.

`mv_release()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_selstrm()` Select an MPEG Video stream

```c
mv_selstrm(path, vimid, stream, width, height, rate)
int     path,     /* Path Number */
        vimid;    /* Video mapID */
        stream,   /* MPEG Video stream number (0..15) */
        width,    /* Width (UCM coords) */
        height,   /* Height (UCM coords) */
        rate;     /* Picture rate (0, 23..25, 29..30) */
```

When another stream must be selected, this call passes the required parameters to
the decoder. The sequence parameters must be passed together with this call to
prevent a mismatch of those parameters with the selected stream.

This function will become effective on a picture change. At the same moment any
data available in the input buffer of the MPEG Video decoder is skipped. The
incoming data not belonging to the selected stream is skipped also.

The constraints for the image size parameter can be found in the `mv_imgsize()`
function description.

If the parameter Picture rate is zero, the decoder will skip all data until a sequence
header was found.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which created the descriptor.

`mv_selstrm()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_show()` Enable the display of the window

```c
mv_show(path, page)
int     path,   /* Path Number */
        page;   /* Page Number */
```

This function enables the window to be displayed in the full motion video plane. If
an MPEG Video play is currently active then the window of the active map will be
enabled on the next picture change, otherwise it is enabled on the next vertical
retrace.

The page number parameter indicates the page to be displayed when not active or
when the active play is of type still.

This is a privileged function. Permission is granted if the function is called by the
super user, if no play is currently active or if the currently active play was started by
the same user that calls this function.

`mv_show()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_slink()` Link to external subroutine module

```c
mv_slink(path, subptr)
int  path,         /* Path Number */
     subptr;       /*ptr. name of subr. module*/
```

This function is used as a generalized method of
expanding the command set of MoviMan. It utilizes
a special subroutine module which is linked into
MoviMan. When MoviMan receives an unknown GetStat
or Setstat service request, it will pass the call
to the subroutine module. If it is unknown to the
subroutine module, the subroutine module should
then pass the call to the driver.
There are four significant entry points into this
external subroutine module: `Init`, `SetStat`, `GetStat`
and `Close`.

`Init` will be called when the module is first
linked to by MoviMan. It is provided to allow the
module to initialize data structures and
variables. Likewise, the `Close entry` point will be
called when the last path to MoviMan is closed. At
this time the subroutine module should free any
memory which was allocated during its execution.

The `SetStat` and `GetStat` entry points are provided
to actually do the main work of the subroutine
module.

It is impossible for two different subroutine
modules to be linked at the same time. This
applies even when they are linked by different
processes. However, more than one process may be
linked to the same subroutine module.

`mv_slink()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_status()` Return the status of an active MPEG Video play

```c
mv_status(path, mvstatus)
int          path;        /* Path Number */
MotionStatus *mvstatus;   /* Pointer to MV Status Block */
```

This function returns the currently active MapID and its status. It passes a buffer
which is filled in by the decoder. If no map is active, an `E$NoPlay` error will result.
If `mvstatus` is a null-pointer, no status block will be filled and only the currently active
MapID is returned.

By using the returned MapID value, more information can be retrieved by issuing the
`mv_info()` call and reading the descriptor fields.

This is a privileged call and may be used only by a process with a userID of the super
user or, if a play is active, with the userID of the process which started that play.

`mv_status()` returns the status of the current audio play in the structure
pointed to by `mvstatus`. `MotionStatus` is defined in `mv.h` and below:

```c
typedef struct _motionstatus {
    unsigned short  MVS_LCntr;      /* loops remaining */
             char   *MVS_CurAdr;    /* address to retrieve data */
    unsigned int    MVS_Speed;      /* display speed */
    unsigned int    MVS_ImgSz;      /* image size of current stream */
    unsigned int    MVS_TimeCd;     /* timecode of current picture */
    unsigned short  MVS_TmpRef;     /* temporal reference */
    unsigned short  MVS_Stream;     /* current stream number */
    unsigned char   MVS_PicRt,      /* picture rate */
                    MVS_Res1;       /* reserved */
    unsigned int    MVS_DSC,        /* Video decoder system clock */
                    MVS_Res2;       /* reserved */
} MotionStatus;
```

`mv_status()` function returns the video mapID if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_trigger()` Define MPEG Video events to signal

```c
mv_trigger(path, mask)
int     path,   /* Path Number */
        mask;   /* Signal/Event Mask */
```

This function activates signalling of MPEG Video events. The application can define,
by means of setting or clearing bits in the event mask parameter, on the occurrence
of which MPEG Video events it wants to receive a signal from the decoder. In case
where one or more of the indicated events happens, a signal will be received. The
value of this signal consists of two parts. The upper 5 bits of the 16-bit signal value
are determined by the application at the moment it issued the `mv_trigger()` call. The
remaining bits reflect the status of the decoder at the time the event occurred. The
decoder will only pass those status bits to the application that were enabled in the
eventmask at the time the `mv_trigger()` call was made.

The setting of the signal/eventmask remains valid for this path until either the path
is closed or a new `mv_trigger()` call is issued for this path.

```plaintext
   +------------+---+---+---+---+---+---+---+---+---+---+---+
   | signalbase |   |NIS|BUF|EOS|EOI|CNP|LPD|SOS|GOP|PIC|DER|
   +------------+---+---+---+---+---+---+---+---+---+---+---+
bit 15  ...   11  10  9   8   7   6   5   4   3   2   1   0
```

Where:

- bit 0: `DER` Signal when Data ERror detected
- bit 1: `PIC` Signal on PICture displayed
- bit 2: `GOP` Signal when display of the first intra-coded picture of a Group Of Pictures start
- bit 3: `SOS` Signal when display of the first intra-coded picture of a Sequence starts
- bit 4: `LPD` Signal when Last Picture Displayed
- bit 5: `CNP` Signal when old PCL-structure not in use anymore
- bit 6: `EOI` Signal when ISO 11172 End Code detected
- bit 7: `EOS` Signal when Sequence End Code detected
- bit 8: `BUF` Signal when Buffer UnderFlow detected
- bit 9: `NIS` Signal when new sequence parameters are found
- bit 10: Reserved and must be zero
- bit 11..15 signalbase: upper 5 bits of 16-bit signal to send (value must be between `00001` and `11111` binary)

The PIC, GOP, SOS, LPD and NIS events are synchronized to the video display
process. They are generated in the vertical blanking interval in between active video periods.

An EOI or an EOS event is generated when an iso_11172_end_code or a
sequence_end_code respectively is detected at the input of the MPEG Video
Decoder.

The NIS event indicates that one or more of the values stored in the `MD_ImgSz` or
`MD_PicRt` fields of the MPEG Video Descriptor have been changed by the Full Motion
system because new values have been found in the MPEG Video stream by the
MPEG Video Decoder. Current values (if any) are still available in the `mv_status()`
block fields `MVS_ImgSz` and `MVS_PicRt`

During normal speed play, slow motion and single step mode, the NIS event is
generated at the display picture change prior to the change to the picture with the
new parameter values. At the beginning of a play and in scan mode, the NIS event
will be generated at least two video display fields before the change to the picture
with the new parameter values.

The LPD event indicates that the last picture of an MPEG Video sequence is about
to appear on the output of the MPEG Video Decoder. Generally this is the last
picture - in display order - before a sequence_end_code of an MPEG Video stream.
There is one exception though. When playing from host, the LPD event is also
generated just before the last picture of the map or the looparea is displayed.

`mv_trigger()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `mv_window()` Define display window within decoded picture

```c
mv_window(path, vimid, x, y, width, height, scroll)
int     path,     /* Path Number */
        vimid,    /* Video mapID */
        x,        /* Horizontal Offset (UCM coordinates) */
        y,        /* Vertical Offset (UCM coordinates) */
        width,    /* Width (UCM coordinates) */
        height,   /* Height (UCM coordinates) */
        scroll;   /* Scroll Flag (not 1: scroll, 0: no scroll) */
```

This function defines the display window within the decoded picture. It does not
enable the display of the window. If the size and offset make the window come
partly outside the decoded picture, the size will be adapted. If the window is
positioned completely outside the decoded picture, an illegal parameter error will
be returned. This means automatically that the decoded picture size must be known
before the `mv_window()` call.

```plaintext
DECODED PICTURE
+---------------------------+
|         ^                 |
| OFFSET  Y                 |
|         v                 |
|<   X   >+========+        |
|         | WINDOW | HEIGHT |
|         +========+        |
|            WIDTH          |
+---------------------------+
```

Two exceptional cases, where the window comes partly outside the decoded picture,
are illustrated below. In both cases, the decoder will adapt the size of the window.

1) In this case the sum of the offset and the size is greater than the size of the
decoded picture. Only the area enclosed by dotted lines will be displayed:

```plaintext
+-----------------+
|                 |
|             +===+---+
|             :   :   |
+-------------+===+   |
              |       |
              +-------+
```

2) In this case, the offset is negative. Only the area of the window enclosed by  dotted lines will be displayed.

```plaintext
+-------+
|       |
|   +===+---------------+
|   :   :               |
+---+===+               |
    |                   |
    +-------------------+
```

If the specified map is currently active, the new parameters will become effective on
the next picture change. If the `scroll` parameter is not zero and the window
move is less than 16 pixels and less than 16 lines, the window will be repositioned
on the next vertical retrace.

If the MVM is not active, the parameters are copied into the MVM descriptor and will
be executed as soon as the MVM becomes active.

When the size of the decoded picture changes due to new sequence parameters, the
window size will be adapted on the next picture change if the window is larger than
the decoded picture. If the window is smaller than the decoded picture, no
adaptation is done.

This is a privileged call and may be used only by a process with a userID of the super
user or with the userID of the process which created the descriptor.
Applications must be aware that the return value (the actual size) is only valid if a
play is active and actual sequence parameters are set.

`mv_window()` function returns the actual window size (W:H) in UCM coordinates if successful, otherwise `-1` is returned and `errno` is set.
