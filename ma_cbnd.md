# CD-i MPEG Audio Libraries Technical Reference Manual

> This document is a manual conversion from the scanned `ma_cbnd.pdf`. Notes by the creator of this document are listed in quote sections like this one.
>
> Parts of this document have been appended from the Green Book descriptions.
>
> Please refer to the Green Book description of the corresponding `MA_` function for complete information on function parameters and behaviour.
>
> The original document is &copy; Copyright 1993, Philips Consumer Electronics B.V.

## Table of contents

- [`ma_abort()`](#ma_abort-abort-the-currently-executed-play) Abort the currently executed play
- [`ma_close()`](#ma_close-free-the-used-audio-descriptor) Free the used audio descriptor
- [`ma_cntrl()`](#ma_cntrl-select-stream-number-and-attenuator-settings) Select stream number and attenuator settings
- [`ma_continue()`](#ma_continue-continue-a-paused-play) Continue a paused play
- [`ma_create()`](#ma_create-reserve-and-initialize-an-audio-map-descriptor) Reserve and initialize an audio map descriptor
- [`ma_info()`](#ma_info-return-a-pointer-to-the-audio-map-descriptor) Return a pointer to the audio map descriptor
- [`ma_jump()`](#ma_jump-compensate-audio-streams-timing-parameters-on-behalf-of-seamless-jump) Compensate audio streams timing parameters on behalf of seamless jump
- [`ma_loop()`](#ma_loop-repeat-a-part-of-the-audio-map) Repeat a part of the audio map
- [`ma_pause()`](#ma_pause-pause-the-current-asynchronous-play) Pause the current asynchronous play
- [`ma_cdplay()`](#ma_cdplay-play-audio-from-cd) Play audio from CD
- [`ma_hostplay()`](#ma_hostplay-start-play-of-an-host-audio-map) Start play of an host audio map
- [`ma_release()`](#ma_release-release-the-mpeg-audio-decoder-for-other-processes) Release the MPEG Audio Decoder for other processes
- [`ma_slink()`](#ma_slink-link-to-external-subroutine-module) Link to external subroutine module
- [`ma_status()`](#ma_status-return-the-status-of-the-current-mpeg-audio-play) Return the status of the current MPEG Audio Play
- [`ma_trigger()`](#ma_trigger-define-events-to-signal) Define events to signal

## Introduction

The functions described in this chapter are additions to the standard OS-9 library.

The library has the following name:

- `macdi.l` c-bindings with parameter checking

A C-program accessing the in this chapter mentioned functions must
link with the following library files. For example:

```plaintext
cc prog.r -l=/dd/lib/macdi.l
          -l=/dd/lib/fmv.l -f=prog
```

All constants and structure definitions are made in DEFS file `ma.h`.

## MPEG Audio Library Functions

### `ma_abort()` Abort the currently executed play

```c
ma_abort(path)
int  path;   /* Path Number */
```

This function aborts the play that is currently being executed.
The play will not be active any longer and
fields in the map descriptor are reset. Audio
output is disabled (muted). This is a privileged call
and may used only by a process with a userID of the
super user or with the userID of the process which
started the operation. If no play is active, an abort
error will be returned.

If this play was running in synchronized mode with
a video play, then this call will end the synchronized
mode. The video playback will continue in non-synchronized
mode.

`ma_abort()` returns `0` if successful; otherwise `-1` is
and `errno` is set.

-----

### `ma_close()` Free the used audio descriptor

```c
ma_close(path, aumid)
int     path,     /* Path Number */
        aumid;    /* Audio mapID */
```

This function aborts any ongoing actions on the
specified audio mapID and frees the used descriptor.
This is a privileged call and may be used only by a
process with a userID of the super user or with the
userID of the process which created the audio mapID.

`ma_close()` returns `0` if successful; otherwise `-1`
returned and `errno` is set.

-----

### `ma_cntrl()` Select stream number and attenuator settings

```c
ma_cntrl(path, aumid, atten, stream)
int     path,    /* Path Number */
        aumid,   /* Audio mapID */
        atten,   /* attenuator settings */
        stream;  /* MPEG stream number */
```

This function selects for the given mapID the MPEG
audio stream number to decode. The attenuation at the
time of playback is also set with this call. When the
given audio roapID is not active at the time of
call the new parameters will be stored in the descriptor.
rf the audio map is active at the time of
call, then the parameters are used immediately. The
new parameters are only programmed to the hardware if
differ from the current active values.

This a privileged call and may be used only by a
process with a userID of the super user or with the
userID of the process which created the audio mapID.
, on an play, the audio stream
changed, it can take some time before this new stream
is actually decoded. The moment that new stream
becomes active can be monitored in two ways:

- the `ma_status()` function shows which stream is active
- the `ma_trigger()` function can be used to assign a
signal-to the event where the new stream becomes
active.

attenuation value format:

```plaintext
|   Left-to-Left  |  Left-to-Right  |  Right-to-Right |  Right-to-Left  |
| M | Attenuation | M | Attenuation | M | Attenuation | M | Attenuation |
 31  30   ...   24 23  22   ...   16 15  14   ...    8  7  6    ...    0
```

where M is:

- `1`: Mute (no sound)
- `0`: Use attenuation value

`ma_cntrl()` returns `0` if successful; otherwise `-1`
is returned and `errno` is set.

-----

### `ma_continue()` Continue a paused play

```c
ma_continue(path)
int     path; /* Path Number */
```

This function continues a paused, asynchronous play.
This a privileged call and may be used only by a
process with a userID of the super user or with the
userID of the process which issued the pause command.
If the play is not paused, or no asynchronous play
is active, a bad mode error will returned.

The availability of audio data is the responsibility
of the application. When playing from CD, data
retrieval must be started after this call.

When playing from host, data retrieval is resumed by
the audio system. Decoding will resume when the first
"entrypoint" enters decoder

When playing from CD, decoding resume as soon as
the data coming from CD is ready to be decoded.

If the audio play is active in synchronized mode with
video, if this synchronized play paused,
function will cause both audio and the video to
continue. If a synchronous play is frozen, this function will
return an error.

`ma_continue()` returns `0` if successful; otherwise `-1` is
returned and `errno` is set.

-----

### `ma_create()` Reserve and initialize an audio map descriptor

```c
ma_create(path, type)
int    path; /* Path Number */
       type; /* Descriptor Type */
```

This function reserves and initializes a descriptor.
The format of the descriptor can be found in the
`ma.h` file. The initial values of the descriptor fields
are given in the following table:

| Name | Initial value |
| - | - |
| MD_Id | free index# |
| MD_Type | user setting |
| MD_Stream | 0 |
| MD_StLoop | 0 |
| MD_EnLoop | 0 |
| MD_LpCnt | 0 |
| MD_LCntr | 0 |
| MD_At_LL | 0x80, muted |
| MD_At_LR | 0xFF |
| MD_At_RR | 0x80, muted |
| MD_At_RL | 0xFF |

Format of the audio type parameter:

```plaintext
     +----------------------------------+-----+
     |                                  | SRC |
     +----------------------------------+-----+
bit 15                                     0
```

Bit 0: When the SRC (source) bit is set (`1`), the system will assume that data is coming from CD. If this bit is not set (`0`), data must be provided from host memory.

Bit 1-15: Reserved for future use, must be zero.

Note: If the audio type indicates the source is
host memory, it is the task of the application
to allocate (and return) an audio map. The
address and size of this map is passed to the
system with the `ma_play()` function.

-----

### `ma_info()` Return a pointer to the audio map descriptor

```c
MAmapDesc *ma_info(path, aumid)
int        path,    /* Path Number */
           aumid;   /* Audio mapID */
```

This function returns a pointer to the audio map
descriptor corresponding to the given audio mapID.
The fields in the descriptor are for information
purposes only. Fields should only be changed by
calling the appropriate functions.
The audio map descriptor has the following format:

```c
typedef struct mamapdesc {
   unsigned short  MD_Id;
   unsigned short  MD_Type;
   unsigned short  MD_Stream;
   unsigned int    MD_StLoop;
   unsigned int    MD_EnLoop;
   unsigned short  MD_LpCnt;
   unsigned short  MD_LCntr;
   unsigned char   MD_At_LL;
   unsigned char   MD_At_LR;
   unsigned char   MD_At_RR;
   unsigned char   MD_At_RL;
   unsigned char[10] Reserved;
} MAmapDESC;
```

`ma_info()` returns the pointer to the mapdescriptor
if-the call is successful, otherwise `-1` is returned
and `errno` is set.

-----

### `ma_jump()` Compensate audio streams timing parameters on behalf of seamless jump

```c
ma_jump(path, delta)
int     path,     /* Path Number */
        delta;    /* Delta Value (signed) */
```

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
resolution of 22.5 kHz. SCR(j) is the value of the
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

`ma_jump()` returns `0` if successful, otherwise `-1` is
returned and `errno` is set.

-----

### `ma_loop()` Repeat a part of the audio map

```c
ma_loop(path, aumid, strtaddr, endaddr, count)
int     path,     /* Path Number */
        aumid,    /* Audio mapID */
        strtaddr, /* start address of loop */
        endaddr,  /* end address of loop */
        count;    /* loopcounter */
```

This function sets the loopback points. This call is
only allowed for audio mapIDs of the "host" type.
For other types, a bad mode error will be returned.
If the audio map is not active, setting the loopback
variables will have no immediate effect. As soon as
the map is activated, the loopback function will be
executed. The loopback start must point to the start
of an audio access unit (entrypoint), the loopback
end must point to the byte following the last audio
access unit to be included in the loopback area. The
start and end are offsets relative to the start of
the map. If the audio map (with it's loopback points
set) is activated (through `ma_play()`), it will play
the audio data from the start-of the looparea until
the end of the looparea is reached. The loopcount
will be decremented and when it did not reach zero,
the data between the looppoints will be decoded
again, until the loopcount is zero. When the
loopcount reaches zero, the playback will be
aborted.. If the audio map is already being played
when this call is made, the behavior of this
function depends on the current position in the
audio map. If the current position is before the
ending loopback point, then playing continues until
the ending loopback point is reached, at which time
the loop count is decremented and the looping area
is played again, if necessary. If the current
position is already past the ending loopback point,
an immediate jump will take place to the starting
loopback point.

If the loopback function is already active when this
call is made, the current loop will be played until
the old endpoint. At that moment the new loopback
points and count will become effective (no decrement
of count) and the procedure described in the
previous paragraph will be followed.

If the designated mapID is active and in synchronized
mode with video, then this `ma_loop()` will
result in a bad mode error.

Variable `startaddr` must be smaller then variable
endaddr. `count` may vary between 0 and 65536.

`ma_loop()` returns `0` if successful; otherwise `-1`
returned and `errno` is set.

### `ma_pause()` Pause the current asynchronous play

-----

```c
ma_pause(path)
int     path;     /* Path Number */
```

This function pauses the asynchronous play that is
currently being executed. This is a privileged
function call and may be used only be a process with
a userID of the super user or with the userID of the
process that started, the play operation. If no
asynchronous play is active at the time this
function is called, or if the play is already
paused, a bad mode error will be returned.

If the audio play is in synchronized mode with
video, this function will pause both the video and
the audio playback.

If a synchronous play is active and the video driver
is "frozen", this function will return a "bad mode"
error.

`ma_pause()` returns `0` if successful; otherwise `-1` is
returned and `errno` is set.

-----

### `ma_cdplay()` Play audio from CD

```c
ma_cdplay(path, aumid, offset, pcl, statusblk, syncmode, syncoffset)
int      path,       /* Path Number */
         aumid,      /* Audio mapID */
         offset,     /* Offset in PCL buffer */
         syncmode,   /* Sync mode/path to sync */
         syncoffset; /* Sync offset to video */
PCL      *pcl;       /* Address of PCL structure */
STAT_BLK *statusblk; /* Pointer to statusblock */
```

This function starts to play, asynchronously, the
data belonging to the given audio mapID. This is a
privileged call and may be used only by a process
with the userID of the superuser of with the userID
of the process which created the audio descriptor.
The data is coming from the CD. PCL is a pointer to
the PCL structure which the application has to
initialize before calling the `ss_play()` function
to retrieve data from the disc. If-the process has
some other mapID currently playing through the
same path, that one will be aborted instantly and
the new play will be started. When, for the same
process, a play is active on another path, a device
busy error will be returned. If a process wants to
start a play, while some other process is executing
an ma_play(), it will be put in the sleeping queue
until the other process releases the audio device
(`ma_release()`).

If the sync mode parameter is set to `-1`, then the
system assumes that no synchronization to video is
required.

If the sync mode parameter is set to `-2`, then this
play will enter a wait state. It will remain in this
state until a video play has been started, that must
synchronize to this audio path.
In synchronized mode, this parameter contains the
Path Number of the active video play to which the
audio play should synchronize itself. If the video
play is paused or in a not-normal speed, then the
audio will be started anyway, but no audible output
will be generated. As soon as the accompanying
video starts to run in normal mode, the audio will
be enabled. The synchronization offset parameter
indicates the constant difference between the timing
parameters in the audio and video sequence. This
parameter is defined, in untis of 22.5 kHz, as the
most significant 32 bits of the difference between
the decoder system clocks in the MPEG Video decoder
and the MPEG audio decoder.

In synchronized mode, no queuing will be done. Any
play request while the decoder is in the
synchronized mode will result in an "device busy"
error. The play that is active on the (sync) path to
which this function wants to synchronize, should
have beeri started by the same process and user,
otherwise a "permission" error will result.

`ma_cdplay()` returns `0` if successful; otherwise `-1`
is returned and `errno` is set.

-----

### `ma_hostplay()` Start play of an host audio map

```c
ma_hostplay(path, aumid, size, audmap, offset, statusblk, syncmode, syncoffset)
int      path,         /* Path Number */
         aumid,        /* Audio mapID */
         size,         /* Size of the audio map */
         offset,       /* Offset within audio map */
         syncmode,     /* Sync mode/path to sync with */
         syncoffset;   /* Sync offset */
int      *audmap;      /* Pointer to audio map */
STAT_BLK *statusblk;   /* Pointer to statusblock */
```

This function start to play, asynchronously, the
data belonging to the given audio mapID. It is a
privileged call and may be used only by a process
with the user ID of the superuser or with the
userID of the process which created the audio
descriptor.

The data in the audio map is supplied by the host.
If the process has some other mapID currently
playing through the same path, this play will be
aborted immediately and the new play will be
started. When, for the same process, a play is
active on another path, a device busy error will
be returned. If a process wants to start a play,
while some other process is executing an `ma_play()`
it will be put in the sleeping queue until the
other process releases the audio device
(`ma_release()`).

If loopback variables are set, these will be
tested before the play is started. If these
variables are not correct (not within the offered
map), an `E$IllPrm` error is returned. otherwise the
playback starts at the starting loopback point.
If the sync mode parameter is set to `-1`, then the
system assumes that no synchronization to video is
required.

If the sync mooe parameter is set to `-2`, then this
play will enter a wait state. It will remain in
this state until a video play has been started,
that must synchronize to this audio path.

In synchronized mode, this parameter contains the
Path Number of the active video play to which this
audio play should synchronize itself.

If the video play is paused or in a not-normal
speed, then the audio will be started anyway, but
no audible output will be generated. As soon as
the accompanying video starts to nm in normal
mode, the audio will be enabled. The
synchronization offset parameter, indicates the
constant difference between the timing parameters
in the audio and the video sequence. This
parameter is defined in units of 22.5 kHz, as the
most significant 32 bits of the difference between
the decoder system clocks in the MPEG Video
decoder and the MPEG audio decoder.

In synchronized mode, no queuing will be done. Any
play request while the decoder is in the
synchronized mode will result in an "device busy"
error. The play that is active on the (sync) path
to which this function wants to synchronize,
should have been started by the same process and
user, otherwise a "permission" error will resulte

`ma_hostplay()` returns `0` if successful, otherwise
`-1` is returned and `errno` is set.

-----

### `ma_release()` Release the MPEG Audio Decoder for other processes

```c
ma_release(path)
int  path, /* Path Number */
```

When an MPEG Audio play has finished, the process should release the MPEG audio
decoder for other processes. As long as the decoder is not released, other
processes that issued an `ma_play()` command will remain in the sleeping queue.

If an MPEG audio play is still active on the given path when the MA_Release
function is called, the play will be aborted. Implicitly, the synchronized mode,
if enabled, will end because of this call. The MPEG video playback, that is part
of this synchronized play, will continue in non-synchronized mode, without MPEG
audio.

If the userID of the caller is not the super user and is not equal to the userID
of the process that started that play, then aborting will fail and result in a
"permission error".

If the audio output was not yet muted, the `ma_release()` call will mute the
output of the audio. If the device is not allocated to the path, then a "bad mode"
error will be returned.

-----

### `ma_slink()` Link to external subroutine module

```c
ma_slink(path, subptr)
int  path,         /* Path Number */
     subptr;       /* Pointer to the name of subroutine module */
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

`ma_slink()` function returns `0` if successful,
otherwise `-1` is returned and `errno` is set.

-----

### `ma_status()` Return the status of the current MPEG Audio Play

```c
ma_status(path, mastatus)
int       path;         /* Path Number */
MA_status *mastatus;   /* Pointer to MA Status Block */
```

This function returns the currently active audio mapID and its status. If no map
is active, an `E$NoPlay` error will result. If the active play was not started
with a userID equal to that of the caller and the caller is not the super user,
then a permission error will be returned.

A 32-byte buffer has to be passed which is filled in by the decoder. If
`mastatus` is a NULL pointer, no status block will be filled in and just the
currently active MapID is returned.

By using the returned audio mapID value, more information can be retrieved by
issuing the `ma_info()` call and reading the descriptor fields.

`ma_status()` returns the status of the current audio play in the structure
pointed to by `mastatus`. `MA_status` is defined in `ma.h` and below:

```c
typedef struct _audiostatus {
    unsigned short MAS_Stream;    /* Actually decoded stream */
    unsigned int   MAS_Att;       /* Actual attenuator status */
    unsigned int   MAS_Head;      /* Audio frame header */
    unsigned int   MAS_CurAdr;    /* Current pointer in audio map */
             int   MAS_DSC;       /* Decoder system clock value */
    unsigned char  MAS_res[14];   /* Reserved */
} MA_status;
```

`ma_status()` returns the audio mapID if successful; otherwise `-1` is,
returned and `errno` is set.

-----

### `ma_trigger()` Define events to signal

```c
ma_trigger(path, mask)
int path,    /* Path Number */
    mask;    /* signal and event info */
```

This function defines the audio events to send a
signal on. The application can define, by means of
setting or clearing bits in the event mask
parameter, on the occurence of which events it wants
to receive a signal from the decoding system.

In case where one (or more) of the indicated events
happens, a signal will be received. The value of
this signal consists of two parts. The upper S bits
of the 16-bit signal value are determined by the
application at the moment it issued the `ma_trigger()`
call. The remaining bits reflect the status of the
decoding system at the time the event occured. The
system will only pass those status bits to the
application that were enabled in the eventmask at the
time the `ma_trigger()` call was made. The setting of the
signal eventmask remains valid for this path until
either-the path is closed or a new `ma_trigger()` call is
issued for this path.

Format of `mask` parameter:

```plaintext
   +------------+---------------+---+---+---+---+---+
   | signalbase |               |DEC|UNF|UPD|CSU|EOI|
   +------------+---------------+---+---+---+---+---+
bit 15  ...   11 10    ...     5  4   3   2   1   0
```

Where

- bit 0: `EOI` End Of Iso stream detected
- bit 1: `CSU` Decoder changed to new audio stream
- bit 2: `UPD` The decoder updated the frameheader
- bit 3: `UNF` The decoder has no data to decode
- bit 4: `DEC` The decoder started decoding
- bit 5...10 Reserved, should be zero
- bit 11..15 signalbase: upper 5 bits of 16-bit signal to send (value must be between `00001` and `11111` binary)

`ma_trigger()` returns `0` if successful; otherwise `-1` is,
returned and `errno` is set.
